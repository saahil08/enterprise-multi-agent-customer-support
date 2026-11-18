# Unit and static validation results

## Local unit tests

Command:

```powershell
node --test .\tests\unit.test.js
```

Result: **9 tests passed, 0 failed**.

Coverage:
1. Ordinary text passes input guardrails.
2. Empty text is blocked.
3. Credential-like text and prompt injection are blocked.
4. Likely PII is marked for review.
5. Valid Supervisor contract is accepted.
6. Inconsistent approval routing is rejected.
7. Chunking creates bounded overlapping chunks and metadata.
8. Pinecone search response normalization supports the expected response variants.
9. Customer ID extraction and observability event construction.

## Static workflow checks

The release validation script checked:
- JSON parsing.
- Unique node names and IDs within each workflow.
- Connection source and target names.
- Execute Workflow IDs resolve to workflows in this package.
- JavaScript Code node syntax.
- Structured output parser JSON schemas.

All checks passed at package build time.

## What these tests do not prove

These are local unit/static tests. They do not execute n8n node implementations or external integrations. The following still require live n8n Cloud tests:
- Credential authentication.
- DynamoDB table/key schema and permissions.
- Telegram trigger/send behavior.
- Gemini model response and parser execution.
- Pinecone documents upsert/search response shape.
- Slack request signature, reviewer authorization and interactive callback.
- Wait-node resume and timeout behavior.
- Observability event persistence.
