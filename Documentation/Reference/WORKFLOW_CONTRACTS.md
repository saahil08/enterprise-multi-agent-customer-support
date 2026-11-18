# Workflow input/output contracts

| Workflow | Required input | Output |
|---|---|---|
| WF_Customer_Support | message, telegram_chat_id, telegram_user_id, telegram_update_id, correlation_id, execution_id, request_timestamp, session_id | response, status, agent, intent, guardrail_decision, correlation_id, execution_id |
| WF_Order_Support | message, intent, target_agent, Telegram IDs, correlation_id, execution_id, session_id | response, status, agent, intent, order_id (when found), correlation_id, execution_id |
| WF_Knowledge_Support | passthrough customer request and Telegram/trace metadata | response, status, agent, intent, retrieval/memory metadata, correlation_id, execution_id |
| WF_AI_Guardrails | message, optional response/output, guardrail_mode (`input` or `output`) | guardrail_decision, guardrail_state, response/message, input_valid |
| WF_Conversation_Memory | message, role (`user` or `assistant`), Telegram chat/user/update IDs | session_id, conversation_history, history_text, current_message |
| WF_Slack_Approval_Request_Subworkflow_Inputs_v2 | approval_request_id, approval_required, approval_type, approval_reason, customer_id, Telegram chat ID, session_id, resume_url, test_mode, correlation_id, execution_id | request_id, Slack card result and approval record context |
| WF_Observability_Log | correlation_id, execution_id, workflow_name, event_name, status, intent, target_agent, logged_at, optional request_timestamp | original event plus observability_persisted |
| WF_Support_Error_Notifier | n8n Error Trigger payload | administrator Telegram alert |

## Supervisor classification schema

The Supervisor parser requires:
- `intent`: enum of supported intents.
- `target_agent`: enum of supported destinations.
- `requires_customer_lookup`: boolean.
- `requires_order_lookup`: boolean.
- `clarification_required`: boolean.
- `reason`: string.
- `approval_required`: boolean.
- `approval_type`: string.
- `approval_reason`: string.

The semantic validation Code node additionally enforces route/intent consistency and requires approval metadata for the approval route.

## Specialist output schema

Customer, Order and Knowledge agents return a structured object with one required field:

```json
{
  "response": "A concise customer-facing answer"
}
```

The surrounding workflow adds deterministic fields such as `status`, `agent`, `intent` and trace identifiers.
