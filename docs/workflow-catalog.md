# Workflow Catalogue

This is an index, not proof every export is present. Reconcile filenames and statuses with the latest n8n workspace.

| Area | Expected workflow/asset | Purpose | Status note |
|---|---|---|---|
| Orchestration | Enterprise Support Agent - Supervisor | Classify and route requests | Verify latest export is in repository |
| Knowledge | WF_Knowledge_Ingestion | Ingest knowledge documents | Previously used |
| Knowledge | WF_Knowledge_Support | Answer knowledge questions | Previously used |
| Order | WF_Order_Support | Handle order requests | Previously used |
| Customer | Customer Support Agent | Customer support | Previously tested; verify exact filename |
| Memory | Conversation Memory | Maintain context | Previously tested; verify export |
| Governance | Guardrails | Policy checks | Previously tested; verify export |
| Approval | Approval Request | Create pending record and post Slack message | Verify latest export |
| Approval | WF_Slack_Interactive_Approval_HITL | Process Slack callbacks | Real Slack callback test pending |
| Testing | WF_HITL_Test_Card_Creator | Create test record and Slack card | Record creation and message posting succeeded |
| Observability | Observability workflow/nodes | Execution monitoring | Previously reported tested; verify scope |

## Export checklist
- Export latest workflow version.
- Use stable filenames.
- Remove secrets and sensitive test/customer data.
- Document required sub-workflows and external services.
- Record last successful test and known limitations.
