# Enterprise Customer Support — workflow-by-workflow setup guide

This guide accompanies the updated workflow JSON exports. Import the files as **new inactive workflows** first. Do not overwrite your existing working versions until the new set is configured and reviewed.

## 1. AWS resources to prepare (region: ap-southeast-2)

| Resource | Required configuration | Used by |
|---|---|---|
| DynamoDB `support-agent-dev` | Existing customer table; verify its partition key and item schema match your current customer records | `WF_Customer_Support` |
| DynamoDB `support-agent-memory-dev` | Existing conversation-memory table; verify key attributes expected by the memory workflow | `WF_Conversation_Memory` |
| DynamoDB `support-agent-approvals-dev` | Existing approvals table; verify key attributes used by request and callback workflows | Slack approval request + callback |
| DynamoDB `support-agent-observability-dev` | **Create if missing**. Partition key `correlation_id` (String), sort key `event_id` (String), on-demand capacity | `WF_Observability_Log` |
| S3 bucket `support-agent-knowledge-dev-saahil` | Existing bucket; upload policy PDFs under category folders such as `policies/returns/` | `WF_Knowledge_Ingestion` |
| Pinecone index `support-knowledge-dev` | Confirm index endpoint, API key, namespace `support-policies`, and integrated embedding configuration compatible with `chunk_text` | Knowledge ingestion + retrieval |

For the observability table, create only the two key attributes above. DynamoDB accepts the other event attributes on `PutItem`. Grant the n8n AWS identity `dynamodb:PutItem` on this table. Add only the specific read/write permissions needed by the other existing tables and S3 bucket; do not use broad `dynamodb:*` or `s3:*` in production.

## 2. n8n credentials

Confirm or recreate credentials in the target n8n workspace. Credential IDs are workspace-specific; after import, reselect the credential on nodes if n8n shows a missing credential.

- `AWS (IAM) account` — AWS region `ap-southeast-2`; permissions for the named DynamoDB tables and S3 bucket operations used by ingestion.
- `Google Gemini(PaLM) Api account` — API key with access to `models/gemini-3.1-flash-lite`.
- `Pinecone API - Support Dev` — access to `support-knowledge-dev`.
- `Telegram_Support_Bot_Dev` — bot token for the customer-facing Telegram trigger and response nodes.
- `Slack_HITL_Dev` — Slack OAuth/bot credential able to post approval cards and confirmations in the configured channel.
- `Slack_Signing_Secret_Dev` — Slack app signing secret used to validate interactive callback requests.

Never paste API keys, bot tokens, signing secrets or AWS access keys into Code nodes or workflow JSON.

## 3. Import order and per-workflow configuration

Import in this order so called subworkflows exist before the Supervisor is imported:

1. **`WF_Support_Error_Notifier`** — select `Telegram_Support_Bot_Dev`; verify the admin Telegram chat ID in its configuration. This is the error-notification entry workflow. Keep inactive until ready.
2. **`WF_Observability_Log`** — select `AWS (IAM) account`; verify table name `support-agent-observability-dev`, region and `PutItem` permission. It stores correlation ID, execution ID, workflow/event name, status, intent/route and timing metadata; it is designed not to store raw customer message or profile content.
3. **`WF_AI_Guardrails`** — inspect the input checks and response contract. No additional database is introduced by this workflow. Verify its Execute Workflow caller receives the expected normalized text field.
4. **`WF_Conversation_Memory`** — select AWS credentials; verify `support-agent-memory-dev` key schema and that the workflow accepts the session/chat identifier and message payload passed by its callers.
5. **`WF_Customer_Support`** — select AWS and Gemini credentials; verify `support-agent-dev` customer-table schema and approved profile fields. Ensure the workflow's call to conversation memory points to the imported `WF_Conversation_Memory`; its logger call points to `WF_Observability_Log`.
6. **`WF_Order_Support`** — select the AWS and Gemini credentials required by its nodes; verify the order table/resource configured in the workflow and its key schema. Ensure its observability subworkflow reference points to `WF_Observability_Log`.
7. **`WF_Knowledge_Support`** — select Gemini and Pinecone credentials; verify index `support-knowledge-dev`, namespace `support-policies`, and the `chunk_text` embedding field. Ensure memory and observability Execute Workflow nodes reference the imported workflows.
8. **`WF_Slack_Approval_Request_Subworkflow_Inputs_v2`** — select Slack and AWS credentials; verify the approval table `support-agent-approvals-dev`, channel ID, and approval request payload fields. Ensure its logger reference points to `WF_Observability_Log`.
9. **`WF_Slack_Interactive_Approval_HTTP_DynamoDB_E2E`** — select Slack, signing-secret and AWS credentials. Inspect `Code - Parse Approval Decision` for the correct Slack channel ID and `Code - Authorize Reviewer` for the exact allowed reviewer user IDs. Do not enable the Slack app callback until the production webhook URL is configured.
10. **`WF_Knowledge_Ingestion`** — select AWS S3 and Pinecone credentials; verify bucket `support-agent-knowledge-dev-saahil`, source prefix/category folders and index/namespace. Run manually only after uploading test PDFs and confirming Pinecone configuration. It extracts PDFs, chunks text (target 1,200 characters with 200-character overlap), and upserts chunks.
11. **`Enterprise Support Agent - Supervisor`** — select Telegram, Gemini, AWS/Slack credentials on relevant nodes. Verify Execute Workflow references for `WF_AI_Guardrails`, `WF_Customer_Support`, `WF_Order_Support`, `WF_Knowledge_Support`, `WF_Slack_Approval_Request_Subworkflow_Inputs_v2` and `WF_Observability_Log`. Confirm Telegram chat/user metadata is passed through and the common response node uses the right Telegram credential.

### Important n8n import note

If an Execute Workflow node shows a missing workflow or references the old workflow ID, open it and reselect the imported workflow by name. Do this for every caller, not only the Supervisor. Keep all imported workflows inactive during configuration.

## 4. Slack app setup (HITL callback)

1. Open your Slack app settings → **Interactivity & Shortcuts** and enable interactivity.
2. Copy the **Production URL** from the imported `WF_Slack_Interactive_Approval_HTTP_DynamoDB_E2E` webhook node.
3. Paste that URL as the Slack Request URL and save.
4. Confirm the signing secret credential is the signing secret for this exact Slack app.
5. Confirm the approval channel ID and reviewer allow-list values in the callback Code nodes.
6. Confirm the Slack bot is a member of the approval channel and has permission to post messages.
7. Only activate the callback workflow after these checks.

The callback records the approval decision and resumes the waiting Supervisor execution. Approval is authorization only; this workflow does not itself execute a refund or payment transaction.

## 5. Recommended activation sequence

Keep everything inactive while configuring. Then activate only the required entry points, in this order:

1. `WF_Support_Error_Notifier` (if you want error notifications active).
2. `WF_Slack_Interactive_Approval_HTTP_DynamoDB_E2E` (after Slack Request URL is saved).
3. `Enterprise Support Agent - Supervisor` (Telegram trigger) last.

Subworkflows are called by Execute Workflow nodes; follow your n8n version's requirement for whether called workflows need to be active. `WF_Knowledge_Ingestion` is a manual operational workflow, not a customer-facing trigger.

## 6. What is new / what you need to create

- **New DynamoDB table:** `support-agent-observability-dev` with `correlation_id` (String) partition key and `event_id` (String) sort key.
- **New workflow:** `WF_Customer_Support`, extracted as a separate customer specialist subworkflow.
- **New workflow:** `WF_Observability_Log`, centralized structured event logging.
- **Supervisor cleanup:** layout and sticky-note organization; removed the empty `Edit Fields` pass-through node and wired `Normalize Customer Message` directly to `AI Agent Supervisor`. No other functional node is removed in this layout pass.
- **Existing infrastructure to verify:** customer, memory and approval DynamoDB tables; S3 bucket; Pinecone index; Telegram and Slack app configuration.

## 7. Pre-test checklist

- [ ] All 11 workflows import without JSON errors.
- [ ] Each credential field is mapped to the correct workspace credential.
- [ ] No Execute Workflow node points to an old/missing workflow ID.
- [ ] Observability DynamoDB table exists and n8n IAM can PutItem.
- [ ] Customer, memory and approval tables have the expected key schemas.
- [ ] Pinecone index, namespace and embedding field match ingestion and retrieval.
- [ ] Slack callback URL, signing secret, channel ID and reviewer allow-list are correct.
- [ ] Admin Telegram chat ID is correct.
- [ ] All workflows remain inactive until configuration review is complete.

## 8. Testing comes next — do not infer runtime success from import

After setup is confirmed, test one route at a time: guardrails → customer lookup → order lookup → knowledge/RAG → complaint escalation → clarification → approval approve/reject/timeout → observability events. Use development customer/order records only. Check the DynamoDB event table by `correlation_id` to confirm trace continuity. Do not use real refund/payment requests for initial tests.

The JSON package was statically validated, but it has not been executed against your live n8n Cloud, AWS, Pinecone, Slack or Telegram credentials. Treat it as a configured candidate until these integration tests pass.
