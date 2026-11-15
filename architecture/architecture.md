
flowchart TD
    A["Telegram Customer"] --> B["Supervisor Workflow"]
    B --> C["Normalize / Deduplicate"]
    C --> D["Input Guardrails"]
    D --> E["Gemini Supervisor"]
    E --> F{"Switch: target_agent"}

    F --> G["Customer Support"]
    F --> H["Order Support"]
    F --> I["Knowledge Support"]
    F --> J["Human Support"]
    F --> K["Clarification"]
    F --> L["Approval Required"]

    G --> M[("DynamoDB Customer Data")]
    H --> M

    I --> N["Pinecone Semantic Search"]
    N --> O["Top 4 Chunks"]
    O --> P["Knowledge Agent"]

    L --> Q["Slack Approval Request"]
    Q --> R["Wait for Decision"]
    R --> S["Approval Callback"]
    S --> T[("DynamoDB Approval State")]

    G --> U["Output Guardrails"]
    H --> U
    P --> U
    U --> V["Telegram Response"]

    W["Conversation Memory"] <--> X[("DynamoDB Memory")]
    B --> Y["Observability Logger"]
    Y --> Z[("DynamoDB Observability")]
