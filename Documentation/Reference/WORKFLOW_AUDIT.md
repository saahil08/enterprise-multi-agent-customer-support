# Workflow audit and refactoring summary

## Supplied workflows

The nine supplied files contained the following existing components:

| Workflow | Role | Action in this release |
|---|---|---|
| Enterprise Support Agent - Supervisor (Phase 3 Observability) | Telegram entry point, supervisor, routing | Refactored; customer logic extracted; structured output and trace logging added |
| WF_AI_Guardrails | Input/output checks | Preserved |
| WF_Conversation_Memory | DynamoDB conversation memory | Preserved |
| WF_Knowledge_Ingestion | S3 PDF ingestion and Pinecone upsert | Rewired ingestion loop and corrected S3 prefix; improved chunk boundaries |
| WF_Knowledge_Support | Pinecone retrieval and knowledge response | Structured output, result normalization, error fallback and trace propagation |
| WF_Order_Support | Order lookup and response | Structured output and trace propagation |
| WF_Slack_Approval_Request_Subworkflow_Inputs_v2 | Create approval record and Slack card | Added correlation ID and approval-request log event |
| WF_Slack_Interactive_Approval_HTTP_DynamoDB_E2E | Slack callback, reviewer validation, conditional update and resume | Added early webhook acknowledgements, removed unrelated Slack header auth from resume request, connected production resume path to confirmation and customer notification |
| WF_Support_Error_Notifier | Error-triggered administrator notification | Added basic secret redaction in alert text |

## New workflows

### WF_Customer_Support

The original customer profile lookup path was embedded in the Supervisor workflow. It is now a dedicated subworkflow called by the Supervisor.

It accepts a structured request contract, retrieves memory context, extracts a customer ID, reads the customer profile, filters sensitive/internal fields, generates a structured response, runs output guardrails, saves assistant memory and returns a standardized response.

### WF_Observability_Log

A reusable logging subworkflow persists low-sensitivity workflow events to `support-agent-observability-dev`. It records correlation ID, execution ID, workflow name, event name, status, intent, target agent, timestamp and duration. It deliberately does not log customer message bodies or full customer records.

## Issues addressed in the supplied JSON

1. Customer Support Specialist and customer lookup were inline in the Supervisor; extracted to `WF_Customer_Support`.
2. Supervisor output was parsed with repeated `JSON.parse()` expressions; replaced with a Structured Output Parser and a semantic contract validator.
3. Customer, order and knowledge agents now use structured response output (`response` field).
4. Customer ID missing/not-found/database-error/agent-error paths now have customer-safe response paths.
5. Customer profile records are filtered to remove common sensitive fields and DynamoDB internal keys before being passed to the model.
6. Supervisor's approval Wait node had a 5-second timeout; increased to 24 hours.
7. Approval timeout now attempts a conditional transition from PENDING to EXPIRED.
8. Slack callback previously used the Slack bot header-auth credential for the n8n Wait resume URL; that unrelated credential was removed. The signed Wait resume URL is used directly.
9. Slack callback now acknowledges the Slack interaction immediately after signature verification using Respond to Webhook nodes.
10. The production resume branch now continues to Slack confirmation and customer notification.
11. Correlation IDs are carried into approval records and log events.
12. Knowledge workflow's empty Execute Workflow input mappings were changed to passthrough where the called workflow requires the incoming data.
13. Knowledge ingestion used an S3 folder prefix that repeated a bucket-like name; changed to `policies/<category>/` because the bucket is already configured on the S3 node.
14. The Knowledge Ingestion SplitInBatches wiring was simplified: S3 listing metadata and extracted PDF text now enter the two Merge inputs by position.
15. Pinecone search request body is passed as a JSON object and its response is normalized to a `matches` array from supported response shapes.
16. Input normalization no longer throws for ordinary non-text Telegram updates; unsupported/empty text is sent through the guardrail response path where Telegram identifiers are available.
17. Added a reusable DynamoDB structured logging workflow and calls from Supervisor, Customer, Order, Knowledge and HITL workflows.
18. Error notification text now redacts common credential patterns.

## RAG assessment

The supplied files already contain a RAG design, not just a direct database lookup:

- S3 policy PDFs are listed and downloaded.
- PDF text is extracted.
- Text is split into overlapping chunks.
- Chunks are upserted to Pinecone's documents endpoint.
- Knowledge Support performs embedding-based search over `chunk_text` with `top_k: 4`.
- Retrieved metadata and chunks are supplied to the Knowledge Support agent.

The chunking implementation is a simple recursive character-boundary approach, not semantic chunking. Current starting settings are 1,200 characters and 200 characters overlap. The search has no score threshold or reranker in the supplied implementation. Retrieval quality must be tuned with a labelled evaluation set.

## Important limitations / live checks

These cannot be proven from exported JSON alone:

- Credentials are references, not credential secrets. They must exist and be valid in the destination n8n workspace.
- DynamoDB table key schemas and IAM permissions must match the guide.
- `support-agent-observability-dev` is a new table and must be created before enabling logging.
- The Pinecone index must support the documents API and have integrated embedding configuration compatible with `chunk_text`.
- The exact Pinecone search response shape must be confirmed from a live execution. The normalizer handles common `matches` and `result.hits` forms.
- Slack channel ID and reviewer allow-list values are environment-specific constants in the callback Code nodes and must be checked.
- Slack Interactivity Request URL must point to the callback workflow's production webhook URL.
- The Telegram administrator chat ID in `WF_Support_Error_Notifier` is environment-specific.
- Customer ID is not proof of identity. This release does not implement a complete identity-verification process. Sensitive profile disclosure must remain blocked until one exists.
- The approval flow records authorization and resumes the Supervisor. It does not execute a refund/payment transaction.
- The 24-hour wait, Slack callback, database state transition and Telegram result require end-to-end verification in n8n Cloud.

## Static validation results

- 11 workflow JSON files parse successfully.
- No duplicate node names or node IDs within the exported workflows.
- No dangling connection endpoints.
- All Execute Workflow references in the package resolve to a workflow ID in the package.
- JavaScript Code node syntax checks pass.
- 9 unit tests pass.

No live n8n integration test has been run from this environment.
