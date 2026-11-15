# Test Matrix

| ID | Area | Scenario | Expected result | Current evidence/status |
|---|---|---|---|---|
| T-001 | Supervisor | Customer profile lookup | Route to customer support | Previously tested at project level; attach execution |
| T-002 | Supervisor | Order status request | Route to order support | Previously tested at project level; attach execution |
| T-003 | Knowledge | Answerable question | Retrieve content and answer | Previously tested; attach execution |
| T-004 | Memory | Follow-up in same conversation | Relevant context retained | Previously tested |
| T-005 | Guardrails | Missing/invalid input | Block or request information | Previously tested; add exact case |
| T-006 | Approval | Create test record | DynamoDB record starts `PENDING` | Reported successful |
| T-007 | Approval | Post Slack card | Slack message contains buttons | Confirmed successful |
| T-008 | Webhook | Direct POST with dummy signature | Reject invalid signature | Confirmed; proves webhook reachability only |
| T-009 | Approval | Real Slack button click | Callback starts and verifies signature | Pending / not working at last test |
| T-010 | Approval | Authorized reviewer approves | Conditional status update | Pending |
| T-011 | Approval | Duplicate decision | Second decision rejected | Pending |
| T-012 | Approval | Unauthorized reviewer | Decision rejected | Pending |
| T-013 | Resume | Production approval with resume URL | Waiting Supervisor resumes | Pending |

## Evidence template
- Test ID:
- Date/time/timezone:
- Workflow and version:
- Synthetic input:
- Expected result:
- Actual result:
- n8n execution ID:
- DynamoDB status:
- Pass / Fail / Blocked:
- Notes:
