# Target architecture

```mermaid
flowchart TD
    T[Telegram Customer] --> N[Normalize Telegram Update]
    N --> D[Deduplicate Update]
    D --> G[WF_AI_Guardrails - Input]
    G --> S[Supervisor Agent + Structured Output Parser]
    S --> V[Validate Supervisor Contract]
    V --> L[WF_Observability_Log]
    L --> R{Route}
    R -->|customer_support_agent| C[WF_Customer_Support]
    R -->|order_support_agent| O[WF_Order_Support]
    R -->|knowledge_agent| K[WF_Knowledge_Support]
    R -->|human_support| H[Human Support Escalation]
    R -->|clarification_required| Q[Clarification Response]
    R -->|approval_required| A[Slack Approval Request]
    C --> TG[Telegram Response]
    O --> TG
    K --> TG
    H --> TG
    Q --> TG
    A --> W[Wait for Approval - 24h]
    W --> TG
    C --> M[WF_Conversation_Memory]
    O --> M
    K --> M
    A --> DB[(DynamoDB Approval Records)]
    CB[Slack Callback Webhook] --> SIG[Verify Slack Signature]
    SIG --> REV[Authorize Reviewer]
    REV --> DB
    DB --> RES[Resume Supervisor Wait URL]
    RES --> W
    C --> LOG[(DynamoDB Observability Events)]
    O --> LOG
    K --> LOG
    S --> LOG
    A --> LOG
```

## Core workflow responsibilities

- **Supervisor:** classify and route only. It does not execute business actions.
- **Customer Support:** customer profile lookup and customer-safe response.
- **Order Support:** order ID extraction and DynamoDB order lookup.
- **Knowledge Support:** Pinecone retrieval and policy-grounded response.
- **AI Guardrails:** input/output validation, simple prompt-injection and sensitive-data pattern checks.
- **Conversation Memory:** DynamoDB-backed history by Telegram session.
- **Approval Request:** persist PENDING approval and post Slack interactive card.
- **Approval Callback:** verify Slack signature, validate reviewer, conditionally update approval and resume the waiting execution.
- **Observability Log:** persist low-sensitivity structured events.
- **Error Notifier:** notify administrator when a configured workflow fails.
- **Knowledge Ingestion:** manually ingest policy PDFs from S3 into Pinecone.

## Data contracts

Each agent workflow should return at least:
- `response`
- `status`
- `agent`
- `intent`
- `correlation_id`
- `execution_id`

The Supervisor supplies:
- `message`
- Telegram identifiers
- `correlation_id`
- `execution_id`
- `request_timestamp`
- `session_id`

The exact contracts are documented in `WORKFLOW_CONTRACTS.md`.
