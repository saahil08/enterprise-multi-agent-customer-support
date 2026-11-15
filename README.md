# Enterprise Multi-Agent Customer Support Automation

## Project Overview

An enterprise-oriented multi-agent customer support automation platform built using n8n, LLM-powered agents, AWS services and human-in-the-loop approval.

A Supervisor Agent interprets incoming requests and delegates work to specialized agents.

## Problem Statement

Traditional customer support operations require manual ticket classification, customer lookups, order verification, policy checks and human intervention for exceptions.

This project demonstrates how AI agents and workflow orchestration can automate these activities while retaining human control over sensitive decisions.

## Solution Architecture

1. **Supervisor Agent** – Understands customer intent and routes requests.
2. **Customer Support Agent** – Handles customer-related queries.
3. **Order Support Agent** – Handles order-related requests.
4. **Knowledge Support Agent** – Retrieves relevant knowledge.
5. **Conversation Memory** – Maintains conversation context.
6. **Guardrails** – Enforces security and business policies.
7. **Human-in-the-Loop** – Uses Slack for approval of sensitive decisions.
8. **Observability** – Tracks workflow execution and outcomes.

## Technology Stack

| Component | Technology |
|---|---|
| Orchestration | n8n |
| AI models | Google Gemini |
| Database | AWS DynamoDB |
| Human approval | Slack |
| Notifications | Telegram |
| Development | VS Code |
| Version control | Git and GitHub |
| Containers | Docker |

## Repository Structure

- `workflows/` – n8n workflow exports
- `docs/` – Technical documentation
- `prompts/` – Agent prompts
- `policies/` – Guardrails and governance
- `database/` – Database schemas and sample data
- `scripts/` – Utility scripts
- `tests/` – Test cases
- `config/` – Sanitized configuration examples
- `architecture/` – Architecture diagrams
- `screenshots/` – Project demonstrations

## Setup

1. Clone the repository.
2. Import the required n8n workflow JSON files.
3. Configure AI provider credentials.
4. Configure AWS and DynamoDB.
5. Configure Slack and Telegram integrations.
6. Configure environment-specific settings.
7. Run the test scenarios.

## Security

Never commit API keys, AWS credentials, Slack signing secrets, Telegram tokens, real customer information or production execution payloads.

Use sanitized sample data and configure credentials independently in your environment.

## Project Status

Development and end-to-end testing in completed.
