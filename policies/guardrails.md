# Guardrails — Baseline Policy Specification

## Input validation
- Validate required fields and data types.
- Reject malformed identifiers.
- Apply input-size limits.
- Treat user input and retrieved documents as untrusted.

## Privacy
- Retrieve minimum necessary customer data.
- Verify identity before disclosing account-specific information.
- Do not log secrets or unnecessary personal data.

## Action boundaries
- Separate read-only lookup from state-changing actions.
- Require authorization for refunds, account changes and restricted actions.
- Require human approval for configured exceptions.
- An LLM recommendation is not authorization.

## Output validation
- Validate structured output before routing.
- Handle missing or malformed fields safely.
- Do not claim success without downstream confirmation.

## Failure handling
- Fail closed for authorization checks.
- Use bounded retries for transient failures.
- Check prior outcome before repeating non-idempotent actions.
- Escalate uncertain cases.

## Auditability
- Record request IDs, outcomes, decisions and timestamps as appropriate.
- Avoid logging secrets or unnecessary customer content.
