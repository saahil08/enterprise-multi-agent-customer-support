# Governance and security controls

## Implemented in the candidate workflows

- Telegram text normalization and duplicate update filtering.
- Input guardrails for empty/oversized text, control characters, common credential patterns, likely PII and a limited set of prompt-injection phrases.
- Output guardrails before customer responses are saved and returned.
- Structured Supervisor output schema and route validation.
- Structured specialist response schema.
- Customer profile field filtering for common sensitive field names and internal DynamoDB keys before the LLM sees the record.
- AWS, Gemini, Pinecone, Slack and Telegram credentials referenced through n8n Credentials, not embedded API secrets.
- Slack signature verification with timestamp freshness check.
- Slack reviewer allow-list and channel validation.
- Conditional DynamoDB approval update from PENDING only.
- Correlation-based audit events without raw customer message content.
- Error notification text redacts common secret patterns.

## Important limitations

- Regex-based PII and prompt-injection checks are heuristic controls, not a full DLP or prompt-injection defence.
- Customer ID is not identity verification. Do not disclose sensitive profile fields until an identity-verification process is designed and tested.
- Slack channel ID and reviewer allow-list are environment-specific values in Code nodes. Verify them before activation.
- Least-privilege IAM permissions must be configured in AWS; the JSON only references an IAM credential.
- Credential rotation and access reviews remain operational responsibilities.
- n8n execution history retention and access policies must be configured for the workspace.
- Do not commit production credentials, exported execution data, customer records or secrets to Git.

## Minimum IAM permissions to review

Existing workflows need the actions actually used against their tables/bucket:
- DynamoDB: GetItem, PutItem, Query and UpdateItem as applicable.
- S3: ListBucket and GetObject for the configured knowledge bucket/prefix.
- Pinecone API key: only the target index/namespace operations required by ingestion and retrieval.

For the new `support-agent-observability-dev` table:
- `dynamodb:PutItem`.

Use separate development and production credentials and resources when moving to production.
