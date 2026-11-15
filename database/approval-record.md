# Approval Record — Example

## Known development references
- Table referenced by the approval test flow: `support-agent-approvals-dev`.
- AWS region used: `ap-southeast-2`.
- Callback lookup key: `request_id`.
- Initial test status: `PENDING`.
- Test/production discriminator observed: `test_record`.
- Production resume field observed: `resume_url`.

## Sanitized illustrative item
```json
{
  "request_id": "APR-EXAMPLE-0001",
  "status": "PENDING",
  "test_record": "true",
  "customer_id": "CUST-SAMPLE-0001",
  "request_type": "refund_exception",
  "reason": "Synthetic example for documentation",
  "created_at": "2026-11-15T00:00:00Z"
}
```
This is an illustrative example, not a verified complete DynamoDB item. Confirm types and required fields against the actual table and workflow.

## Callback behavior documented from workflow design
- Retrieve record by `request_id`.
- Update conditionally only if current status is `PENDING`.
- Test records must not resume production Supervisor executions.
- Production records may resume only when a valid `resume_url` exists.
