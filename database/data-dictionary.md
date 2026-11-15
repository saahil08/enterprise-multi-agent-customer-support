# Data Dictionary — Approval Flow

Verify exact types and required status against DynamoDB before treating this as the final schema.

| Attribute | Purpose | Notes |
|---|---|---|
| `request_id` | Unique request ID and callback lookup key | Lookup key observed |
| `status` | Approval lifecycle | Test record begins `PENDING` |
| `test_record` | Test/production discriminator | Test flow uses string `true`; verify representation |
| `customer_id` | Customer reference | Use synthetic IDs publicly |
| `request_type` | Request category | Example: `refund_exception` |
| `reason` | Approval explanation | Avoid personal data |
| `resume_url` | Waiting n8n execution endpoint | Sensitive; never publish actual values |
| `reviewer_id` | Slack reviewer identifier | Environment-specific |
| `reviewer_name` | Reviewer display name | Store only if needed |
| `decided_at` | Decision timestamp | Prefer UTC |
| `approval_decision` / `decision` | Reviewer outcome | Confirm exact field names in callback output |

Never commit actual resume URLs, customer records, signing secrets, access tokens or production payloads.
