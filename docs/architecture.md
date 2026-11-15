# Solution Architecture

## Purpose
Enterprise-oriented customer-support automation using n8n, LLM-powered agents, AWS DynamoDB, Slack human approval and supporting integrations.

## Logical components
| Component | Responsibility |
|---|---|
| Supervisor | Interprets request intent and selects a specialist route |
| Customer Support Agent | Handles customer-related requests |
| Order Support Agent | Handles order-related requests |
| Knowledge Support Agent | Answers using ingested knowledge |
| Conversation Memory | Maintains relevant conversation context |
| Guardrails | Applies policy checks and controlled routing |
| Human approval | Slack Block Kit, n8n callback and DynamoDB |
| Persistence | AWS DynamoDB |
| Notifications | Telegram integration, where configured |
| Observability | Execution tracking and operational signals |

## Request flow
1. Request enters the Supervisor workflow.
2. Supervisor classifies intent and selects a specialist.
3. Specialist handles its bounded task using configured tools and data.
4. Guardrails constrain actions and outputs.
5. Sensitive or exception requests can require human approval.
6. Approval decisions are validated and persisted before protected downstream actions.
7. Execution outcomes are recorded for troubleshooting.

## Human approval sequence
1. Create a pending approval record in DynamoDB.
2. Post a Slack message with Approve and Reject buttons.
3. Slack sends a signed POST to the n8n production callback webhook after a button interaction.
4. Callback validates signature and reviewer authorization.
5. Callback conditionally updates the pending record.
6. For eligible production records, callback may resume the waiting Supervisor execution using its stored resume URL.

## Current verification note
The test-card creator has successfully created a development DynamoDB record and posted a Slack interactive message. A direct PowerShell POST reached the callback webhook and was rejected at signature validation, as expected for a deliberately invalid signature. A real Slack button click has NOT yet been confirmed to trigger the callback workflow. Do not describe Slack HITL end-to-end as complete until a real signed interaction is observed and the record changes as expected.

## Environment
- n8n Cloud hosts workflow execution.
- Current AWS region used in development: `ap-southeast-2`.
- Approval test table reference: `support-agent-approvals-dev`.
- Slack channel/user IDs and webhook URLs are environment-specific. Do not publish them unless sanitized.
- Never commit credentials, signing secrets, tokens or real customer data.
