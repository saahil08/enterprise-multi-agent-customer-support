# Security and Release Checklist

## Secrets
- Keep `.env` local and excluded from Git.
- Commit only `.env.example` with placeholders.
- Never commit AWS access keys, Gemini API keys, Slack bot tokens, Slack signing secrets or Telegram bot tokens.
- Store credentials in n8n Credentials or an approved secrets manager.
- Rotate any credential that may have been exposed.

## Workflow exports
Review all JSON for token-like strings, passwords, signing secrets, hardcoded IDs, environment URLs and customer data. Remove real names, emails, phone numbers, order details and production payloads. Credential references are not secret values, but review the full export.

## AWS
Use dedicated IAM roles/identities, least-privilege permissions and separate development/production resources.

## Public repository
The repository appeared Public in the provided screenshot. Keep it private during review if uncertain. Making a secret-containing repository private does not remove secrets from history; rotate exposed secrets and clean history where needed.

## Before every push
```powershell
git status
git diff --cached
git check-ignore -v .env
```
