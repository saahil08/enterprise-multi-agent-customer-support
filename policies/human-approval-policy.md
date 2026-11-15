# Human Approval Policy — Baseline Specification

## Intended lifecycle
1. Create a unique approval request ID.
2. Persist the request with status `PENDING`.
3. Send the reviewer a Slack message with decision buttons.
4. Validate Slack request authenticity using the signing secret.
5. Validate reviewer authorization.
6. Accept only supported decisions such as `APPROVED` or `REJECTED`.
7. Conditionally update only a still-pending record.
8. Record reviewer and decision time.
9. Resume downstream execution only for an eligible production request with a valid resume URL.

## Safeguards
- Reject stale or invalid signatures.
- Reject unauthorized reviewers and unknown action IDs.
- Reject duplicate decisions and non-pending records.
- Never let test records trigger production actions.
- Do not expose signing secrets.

Verify each safeguard in the current callback workflow before marking it implemented.
