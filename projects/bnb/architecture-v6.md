
# BnB Internal Communications System

A real-time, AI-powered platform for managing guest and team interactions at a BnB, built with Rocket.Chat, SurrealDB (TiKV backend), and GroqCloud.

## Overview
This system integrates guest data from Guesty (bookings) and Aircall (calls) into a unified Rocket.Chat interface. Staff use natural language queries (e.g., "What did Jane Doe ask for last time?") via an AI Agent, which fetches and synthesizes data from SurrealDB—handling graph relationships, documents, and vectors—and delivers responses powered by GroqCloud’s LLM.

## Stack
- **Rocket.Chat**: Self-hosted chat platform for team comms and guest tracking.
- **SurrealDB**: Multi-model database (graph, document, vector) with TiKV backend for scalable, real-time storage of guest/property/team data.
- **GroqCloud**: LLM (DeepSeek-Qwen32B or GPT-4o Mini) for query parsing and natural response generation.
- **AI Agent**: Custom Node.js app bridging Rocket.Chat, SurrealDB, and GroqCloud.

## Features
- **Team Channels**: `#cleaning-team`, `#front-desk` for coordination.
- **Guest Threads**: `#guests` channel with per-guest threads (e.g., "Jane Doe - Booking #123").
- **Natural Queries**: Staff ask “Who’s cleaning Jane’s villa?”; AI Agent responds with context (e.g., “Team B, early check-in requested”).
- **Real-Time**: Guesty/Aircall updates sync instantly via WebSockets.
- **RAG**: Combines graph relationships (e.g., guest → property → team), document details (e.g., check-in dates), and vector search (e.g., call note embeddings) for full guest views.

## Architecture
- **Data Flow**: Guesty/Aircall → SurrealDB (TiKV) → AI Agent → GroqCloud → Rocket.Chat.
- **Scalability**: TiKV starts at 1 node (20-100 properties), scales to 3-5 nodes (1,000+ properties).

## Setup
1. **SurrealDB**:
   - Deploy with TiKV: `docker-compose up -d` (see `docker-compose.yml` for 1-node config).
   - Schema: Define `guest`, `property`, `team`, `booking`, `call` tables with relationships and vectors.
   - Webhooks: Configure Guesty/Aircall to POST to SurrealDB REST API.

2. **Rocket.Chat**:
   - Install self-hosted: `docker run rocketchat/rocket.chat`.
   - Create channels: `#guests`, `#cleaning-team`, etc.
   - Add AI Agent as a user with webhook access.

3. **AI Agent**:
   - Node.js app: `npm install surrealdb.js groq-sdk`.
   - Logic: Parse Rocket.Chat queries, LLM generates SurrealQL, fetch from SurrealDB, LLM synthesizes response.
   - Deploy: Run on same server or separate (e.g., `$10/month VPS).

4. **GroqCloud**:
   - API key: Sign up, use DeepSeek-Qwen32B or GPT-4o Mini.
   - Prompt: “Convert English to SurrealQL” + “Generate natural reply.”

## Example Query
- **Staff**: “What was Jane asking for in our last call?”
- **AI Agent**:
  - LLM: `SELECT content FROM call WHERE guest = 'guest:jane123' ORDER BY timestamp DESC LIMIT 1;`
  - SurrealDB: “early check-in.”
  - LLM Response: “In her last call, Jane asked for an early check-in.”

## Scaling
- **20 Properties**: 1 TiKV node (8 GB RAM, ~$40/month).
- **1,000 Properties**: 3-5 TiKV nodes (16 GB RAM each, ~$120-$200/month).
- No migration—TiKV scales from the start.

## Costs (Year 1)
- **License**: $0 (all open-source).
- **Dev Time**: 40-80 hours (~$2K-$4K at $50/hr).
- **Servers**: $40-$200/month (SurrealDB + Rocket.Chat + AI Agent).
- **GroqCloud**: ~$1-$10/month (1K-10K queries).
- **Total**: ~$2,500-$5,500 (20-100 properties).

## Next Steps
- Deploy SurrealDB with TiKV single-node.
- Prototype AI Agent with sample Guesty/Aircall data.
- Test queries in Rocket.Chat (e.g., “Jane’s habits”).


### Updated Architecture Diagram
Here’s the revised diagram with SurrealDB (TiKV backend) and AI Agent:

```mermaid
graph TD
    A[Guesty] -->|Webhooks| B[SurrealDB<br>TiKV Backend<br>Graph + Document + Vector]
    C[Aircall] -->|Webhooks| B
    B -->|SurrealQL Queries<br>Graph + Vector| D[AI Agent]
    D -->|English Question| E[GroqCloud<br>DeepSeek-Qwen32B or GPT-4o Mini]
    E -->|Generated SurrealQL| D
    D -->|Raw Data| E
    E -->|Natural Response| D
    D -->|Posts Results| F[Rocket.Chat]
    F -->|Staff Questions"] D
    subgraph Data Sources
        A
        C
    end
    subgraph Backend
        B
    end
    subgraph Processing
        D
        E
    end
    subgraph Interface
        F
    end
```

#### Diagram Notes
- **Data Sources**: Guesty/Aircall feed SurrealDB via webhooks.
- **Backend**: SurrealDB with TiKV handles all data (graph, documents, vectors), scalable from 1 node.
- **Processing**: AI Agent queries SurrealDB, uses GroqCloud to parse questions into SurrealQL and craft responses.
- **Interface**: Rocket.Chat displays results, accepts staff inputs.

### Why This Works
- **TiKV from Start**: Single node (e.g., 8 GB, ~$40/month) for 20-100 properties, scales to 3-5 nodes (~$120-$200/month) at 1,000+—no migration pain, just add servers.
- **SurrealDB**: Consolidates your needs (relationships, details, vectors), real-time via WebSockets, free forever.
- **AI Agent**: LLM-driven queries (e.g., “Jane’s habits”) make it staff-friendly, leveraging SurrealDB’s power.

This setup’s lean, scalable, and avoids rework—perfect for your greenfield build.