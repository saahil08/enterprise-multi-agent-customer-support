# Integration test plan

## Test status

- Local unit tests: 9/9 passed.
- Workflow JSON/static checks: passed.
- Live n8n Cloud integration: NOT RUN from this environment.
- AWS, Pinecone, Slack and Telegram external calls: NOT LIVE-VERIFIED.

## Supervisor and routing

| ID | Input | Expected result | Evidence to capture |
|---|---|---|---|
| SUP-01 | Customer ID profile request | `customer_support_agent` | Supervisor output and Customer Support execution |
| SUP-02 | Order status with ORD1001 | `order_support_agent` | Order Support execution and response |
| SUP-03 | Return policy question | `knowledge_agent` | Pinecone search and response |
| SUP-04 | Complaint | `human_support` | Admin notification and customer response |
| SUP-05 | Refund exception | `approval_required` | Slack card and PENDING record |
| SUP-06 | Ambiguous request | `clarification_required` | Clarification response |
| SUP-07 | Prompt injection | Guardrail blocks before Supervisor | Guardrail decision |

## Customer Support workflow

- Customer ID missing: asks customer for ID.
- Customer record not found: returns not-found response.
- DynamoDB error: returns safe temporary-unavailable response.
- Customer record with sensitive key names: verify fields are filtered before LLM.
- Valid customer record: structured response parser returns `output.response`.
- Output guardrail blocks response: safe fallback returned.
- Memory write failure: customer response should still be restored from the prepared result.

## Order and Knowledge agents

- Order ID missing.
- Order not found.
- DynamoDB order lookup error.
- Valid order response parser.
- Knowledge question with matching policy.
- Knowledge question with no matching policy.
- Pinecone error path.
- Knowledge output guardrail fallback.
- Conversation memory user/assistant calls.

## HITL

- Valid Slack signature.
- Invalid Slack signature.
- Stale Slack timestamp.
- Authorized reviewer.
- Unauthorized reviewer.
- Approve.
- Reject.
- Duplicate callback.
- Missing approval record.
- Approval record not PENDING.
- Production resume URL present.
- Resume URL unavailable.
- 24-hour timeout and EXPIRED transition.
- Confirm the Wait execution resumes and Telegram response is sent.

## RAG

- Ingest at least one PDF from each configured S3 category.
- Verify PDF text extraction.
- Verify chunk sizes and overlap.
- Verify Pinecone upsert.
- Verify retrieval for known questions.
- Verify no-answer behavior.
- Verify policy version metadata and source.
- Measure retrieval hit rate and grounded-answer correctness.

## Reliability and error paths

- Gemini request fails.
- DynamoDB customer lookup fails.
- DynamoDB order lookup fails.
- Pinecone search fails.
- Slack post fails.
- Telegram send fails.
- Observability table unavailable (business flow should continue because log calls are non-blocking).
- Error notifier sends a redacted admin alert.

## Release evidence

For each test capture:
- Test ID.
- Input.
- n8n workflow execution ID.
- Expected output.
- Actual output.
- Pass/fail.
- Screenshot or execution URL.
- Defect and fix reference.
