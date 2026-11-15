# Building a Multi-Agent Customer Support Automation System with n8n, Gemini, Pinecone and AWS

*From Telegram-based customer queries to specialized AI agents, RAG, guardrails and human approval.*

## 1. The idea

What happens when a customer-support chatbot needs to do more than answer FAQs?

It may need to identify a customer, check an order, retrieve a company policy or escalate a sensitive request for human approval.

I built an enterprise-oriented multi-agent customer support project to explore how these capabilities can be orchestrated using n8n, Google Gemini, Pinecone and AWS.

Instead of putting every responsibility into one AI agent, the design uses a supervisor to classify requests and route them to specialized workflows.

**[IMAGE: Project architecture diagram]**

## 2. Technology stack

| Technology | Purpose |
|---|---|
| n8n Cloud | Workflow orchestration |
| Google Gemini | Intent classification and agent reasoning |
| Telegram | Customer interaction |
| AWS S3 | Source document storage |
| Pinecone | Vector-based knowledge retrieval |
| AWS DynamoDB | Customer, memory, approval and observability records |
| Slack | Human-in-the-loop approval |
| GitHub | Workflow exports and documentation |

## 3. How the multi-agent architecture works

The main workflow is `Enterprise Support Agent - Supervisor`.

The request follows this sequence:

1. Telegram receives the customer message.
2. n8n normalizes the message and removes duplicate events.
3. Input guardrails validate the request.
4. Gemini's supervisor classifies the intent.
5. A Switch node routes the request to the appropriate branch.

The supervisor supports customer support, order support, knowledge support, human support, clarification and approval-required routes.

Each specialist is maintained as a separate n8n workflow.

**[IMAGE: Supervisor workflow screenshot]**

### Specialist agents

- **Customer Support:** Validates customer information and looks up customer records in DynamoDB.
- **Order Support:** Extracts an order ID from the current message or recent conversation history and looks up the order record.
- **Knowledge Support:** Retrieves relevant policy content from Pinecone and uses it to prepare an answer.

This separation keeps responsibilities smaller and makes workflows easier to inspect and maintain.

## 4. RAG pipeline: S3 to Pinecone

The knowledge-ingestion workflow processes policy documents stored in Amazon S3.

**Ingestion flow:**

`S3 → PDF extraction → text normalization → chunking → metadata → Pinecone upsert`

The implementation uses a custom recursive character-boundary chunking approach.

| Setting | Implemented value |
|---|---:|
| Target chunk size | 1,200 characters |
| Chunk overlap | 200 characters |
| Boundary selection | Paragraph, line, sentence and whitespace |
| Search result limit | Top 4 chunks |
| Retrieval type | Embedding-based semantic search |

### How we handled chunk boundaries

A simple fixed-length splitter can cut text in the middle of a sentence or paragraph.

To reduce this problem, the ingestion code:

- Normalizes extracted text.
- Searches for a natural text boundary after at least 60% of the target chunk.
- Prefers paragraph and sentence boundaries.
- Carries 200 characters of overlap into the next chunk.
- Avoids beginning the next chunk in the middle of a word.

Each chunk receives a unique ID and metadata such as document ID, category, source, version, effective date and chunk index.

The Knowledge Support workflow then requests up to four semantically relevant chunks from Pinecone using embedding scoring against `chunk_text`.

**[IMAGE: Knowledge ingestion workflow screenshot]**

**[IMAGE: Pinecone index / namespace search screenshot]**

*Note: The current exported search request uses semantic embedding scoring. Lexical and hybrid search are possible future improvements, not claimed as implemented.*

## 5. Guardrails and human approval

An AI-generated response should not automatically authorize a business action.

The reusable `WF_AI_Guardrails` workflow checks:

- Missing or invalid text.
- Input and output length limits.
- Unsupported control characters.
- Common credential and secret patterns.
- Selected prompt-injection patterns on input.
- Email, phone and long-number patterns that may indicate PII.

Detected PII is routed for review. These are rule-based checks, not a complete security or content-safety system.

### Slack human-in-the-loop

The approval design creates a pending approval record in DynamoDB and sends a Slack interactive message with Approve and Reject actions.

The callback workflow includes:

- Slack signature verification using HMAC.
- Timestamp freshness validation.
- Reviewer authorization.
- Supported action and channel validation.
- Conditional DynamoDB state updates.
- Approval decision logging and notification.

**[IMAGE: Slack approval card]**

The approval message and test record creation have been demonstrated. However, the real Slack button callback and complete production resume journey still require final end-to-end verification.

## 6. Memory and observability

Conversation memory is stored in DynamoDB using a Telegram-based session identifier. The workflow reads conversation history and stores user and assistant messages.

A separate observability workflow creates structured events with:

- Correlation ID and execution ID.
- Workflow and event name.
- Status and timestamp.
- Intent and selected agent.
- Execution duration.

These events are persisted in a dedicated DynamoDB observability table.

An error-notification workflow also captures n8n failures and sends an administrator alert through Telegram.

**[IMAGE: DynamoDB table / observability workflow screenshot]**

## 7. Challenges and lessons learned

| Challenge | Approach / lesson |
|---|---|
| Incorrect supervisor routing | Align structured agent output with the exact Switch node field and route values. |
| Chunk boundaries breaking context | Use boundary-aware splitting and 200-character overlap. |
| Duplicate chunk records | Generate deterministic chunk IDs and deduplicate before upsert. |
| Missing order IDs in follow-up messages | Check recent user conversation history before requesting the ID again. |
| Approval state consistency | Use explicit DynamoDB status and conditional updates. |
| Slack callback security | Validate signature, timestamp, reviewer and supported actions. |
| Workflow debugging | Test individual branches and complete business journeys separately. |

The biggest lesson: connecting an LLM to tools is only the beginning. Reliable automation requires explicit state, validation, failure handling and observability.

## 8. Current status and next steps

The project has working workflow components for supervisor routing, specialist support, document ingestion, semantic retrieval, memory, guardrails and structured observability logging.

The remaining work includes completing end-to-end Slack callback and approval-resume validation, expanding automated tests, evaluating retrieval quality and strengthening production deployment controls.

This is an enterprise-oriented development project, not a claim of a fully production-certified support platform.

## 9. Explore the project

**GitHub repository:**  
https://github.com/saahil08/enterprise-multi-agent-customer-support

**Workflow exports:**  
https://github.com/saahil08/enterprise-multi-agent-customer-support/tree/main/workflows

The repository contains workflow exports and supporting project documentation.

I would love to hear from engineers working with n8n, RPA, agentic AI, RAG and enterprise automation.

---

*Project note: Screenshots and example records are from a development environment. Sensitive identifiers and credentials should be redacted before public sharing.*