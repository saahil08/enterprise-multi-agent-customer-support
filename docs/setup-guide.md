# Setup Guide

## Prerequisites
- n8n Cloud or compatible self-hosted n8n.
- AWS account with DynamoDB access.
- Google Gemini access if using Gemini model nodes.
- Slack app with Interactivity enabled for approval.
- Telegram bot if Telegram notifications are enabled.
- Git and GitHub.

## Configure n8n credentials
Create credentials inside n8n for the configured LLM provider, AWS IAM, Slack bot, Slack signing secret and Telegram (if used). Do not put actual credentials in this repository. Re-map credential references after importing workflow JSON.

## Configure DynamoDB
- Current development region: `ap-southeast-2`.
- Approval table referenced by the test flow: `support-agent-approvals-dev`.
- Review `../database/approval-record.md`.
- Confirm actual partition/sort keys, indexes, attribute types and IAM permissions from AWS and the latest workflow nodes. This documentation is not a complete authoritative schema.

## Configure Slack
- Enable Interactivity & Shortcuts in the Slack app.
- Set Request URL to the callback workflow's Production URL.
- Activate the callback workflow.
- Ensure the app posting the card is the same app configured for interactivity.
- Verify with a real Slack button interaction and inspect n8n Executions.

Known issue: the test card posts successfully, but a real button click has not yet been observed triggering the callback. See `implementation-status.md`.

## Import workflows
1. Import current JSON exports.
2. Reconnect credentials.
3. Review environment-specific URLs, IDs, table names and channel settings.
4. Confirm sub-workflow references and permissions.
5. Keep workflows inactive until configuration and security review is complete.
6. Test components individually before end-to-end execution.

## Suggested validation order
Supervisor routing; specialist agents; memory; guardrails; knowledge ingestion/retrieval; approval record creation; Slack message posting; real signed callback; conditional DynamoDB update; production resume; observability and failure paths.

## Production readiness
Add least-privilege IAM, secrets management, reviewer authorization management, idempotency, retry/dead-letter handling, retention policies, monitoring and rollback procedures.
