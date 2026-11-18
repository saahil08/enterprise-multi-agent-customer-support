# Workflow inventory

| Workflow | ID | Entry trigger | Calls |
|---|---|---|---|
| Enterprise Support Agent - Supervisor | `h0EEPMyPA0n4iHec` | Telegram Trigger - Customer Message | Call 'WF_Order_Support' → WF_Order_Support; Call 'WF_Knowledge_Support' → WF_Knowledge_Support; Call WF_AI_Guardrails → WF_AI_Guardrails; Call Slack Approval Request → WF_Slack_Approval_Request_Subworkflow_Inputs_v2; Call WF_Customer_Support → WF_Customer_Support; Call WF_Observability_Log - Supervisor Route → WF_Observability_Log |
| WF_Customer_Support | `8727f99d-6cf8-4dcd-9f5f-12c217d8ba96` | When Executed by Another Workflow | Call WF_Conversation_Memory - Customer User → WF_Conversation_Memory; Customer - Call Output Guardrails → WF_AI_Guardrails; Call WF_Conversation_Memory - Customer Assistant → WF_Conversation_Memory; Call WF_Observability_Log - Customer Completion → WF_Observability_Log |
| WF_AI_Guardrails | `NLsnMNctUX0kTEaY` | When Executed by Another Workflow | — |
| WF_Conversation_Memory | `MXerfypR7ei54yL1` | When Executed by Another Workflow | — |
| WF_Knowledge_Ingestion | `tjORmnww5uHcCPfI` | When clicking ‘Execute workflow’ | — |
| WF_Knowledge_Support | `QzUhAFlW8kj96oHe` | When Executed by Another Workflow | Call 'WF_Conversation_Memory' → WF_Conversation_Memory; Call WF_Conversation_Memory - Assistant → WF_Conversation_Memory; Knowledge - Call Output Guardrails → WF_AI_Guardrails; Call WF_Observability_Log - Knowledge Completion → WF_Observability_Log |
| WF_Order_Support | `SYsIPQJOhQoIyGPw` | When Executed by Another Workflow | Call WF_Conversation_Memory - Order User → WF_Conversation_Memory; Call WF_Conversation_Memory - Order Assistant → WF_Conversation_Memory; Order - Call Output Guardrails → WF_AI_Guardrails; Call WF_Observability_Log - Order Completion → WF_Observability_Log |
| WF_Slack_Approval_Request_Subworkflow_Inputs_v2 | `MppejC5udv1Egras` | Execute Workflow Trigger - Supervisor | Call WF_Observability_Log - Approval Requested → WF_Observability_Log |
| WF_Slack_Interactive_Approval_HTTP_DynamoDB_E2E | `xnrnaTYQzty9aS38` | Manual Trigger - Send Test Card, Webhook - Slack Interaction Test | Call WF_Observability_Log - Approval Decision → WF_Observability_Log |
| WF_Support_Error_Notifier | `lpZclZLRH6RPj1Gx` | Error Trigger - Capture Failure | — |
| WF_Observability_Log | `30f7fa68-a6ca-4500-b3a0-c80d8c9e2484` | When Executed by Another Workflow | — |

## Independent entry points

- `Enterprise Support Agent - Supervisor`: Telegram Trigger.
- `WF_Knowledge_Ingestion`: Manual Trigger.
- `WF_Slack_Interactive_Approval_HTTP_DynamoDB_E2E`: Slack Webhook plus manual test trigger.
- `WF_Support_Error_Notifier`: Error Trigger.

The remaining workflows are reusable subworkflows invoked by Execute Workflow nodes.
