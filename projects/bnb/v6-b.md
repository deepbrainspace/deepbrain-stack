---

# BnB Automation System README

## Overview
This project automates operations, marketing, and financial management for a BnB business, scaling from 0-20 to 100+ properties. Built on a **Hetzner CCX23 cluster** with **Docker Swarm**, it leverages **Rust-based AI agents**, **SurrealDB (TiKV)**, and **GroqCloud DeepSeek-R1-Distill-Qwen-32B** for intelligent automation. The system evolves in three phases:

- **Phase I**: Operations with **OLGA** (Ops Lightweight GenAI Agent).
- **Phase II**: Marketing with **EMMA** (Efficient Multichannel Marketing Agent).
- **Phase III**: Financial analysis with **RAIFA** (Responsive AI Financial Agent).

## Components
- **AI Agents (Rust)**:
  - **OLGA**: Real-time staff ops via Rocket.Chat (e.g., “Jane’s last request”).
  - **EMMA**: Marketing automation/content generation (e.g., promos via Postiz).
  - **RAIFA**: Financial insights, reports, board calls (e.g., “Q2 profits”).
  - Shared `bnb-core` crate for SurrealDB/GroqCloud logic.
- **Core Services**:
  - **Rocket.Chat**: Staff/stakeholder communication hub.
  - **SurrealDB (TiKV)**: Multi-model DB (graph, vector, relational) for ops, marketing, financial data.
  - **GroqCloud**: DeepSeek-32B LLM for AI reasoning (~1-1.5s responses).
  - **Traefik**: Reverse proxy for routing (80/443).
- **Supporting Tools**:
  - **Restic**: Backups to IDrive e2 (~$10/month).
  - **Netdata**: Monitoring (CPU, RAM, network).
  - **Heliocone**: Enhances LLM interactions.
- **Phase II Additions**:
  - **Matomo**: Analytics tracking.
  - **YOURLS**: URL shortening/tracking.
  - **Postiz**: Social media scheduling.
  - **Systeme.io**: External CRM/email automation.
- **Phase III Additions**:
  - APIs: Tracking (Matomo), Banking (e.g., Plaid), Guesty, Stripe.

## Architecture Diagram
```mermaid
graph TD
    subgraph "Phase I - Ops"
        A[Rocket.Chat<br>@Olga] -->|WebSocket| B[OLGA<br>Ops Lightweight GenAI Agent]
        B -->|Queries| C[SurrealDB<br>TiKV]
        B -->|Prompts| D[GroqCloud<br>DeepSeek-32B]
        E[Traefik<br>Reverse Proxy] -->|443| A
        F[Heliocone] -->|Enhances| B
    end

    subgraph "Phase II (Marketing)"
        G[Matomo<br>Analytics] -->|Data| H[EMMA<br>Efficient Multichannel Marketing Agent]
        I[YOURLS<br>URL Tracking] -->|Data| H
        J[Postiz<br>Scheduling] -->|Content| H
        K[Systeme.io<br>CRM/Email] -->|API| H
        H -->|Queries| C
        H -->|Prompts| D
        E -->|443| G
        E -->|443| I
        E -->|443| J
    end

    subgraph "Phase III (Financial)"
        L[RAIFA<br>Responsive AI Financial Agent] -->|Queries| C
        L -->|Prompts| D
        L -->|Board Calls| A
        M[Banking API] -->|Data| L
        N[Guesty API] -->|Data| L
        O[Stripe API] -->|Data| L
        P[Tracking<br>Matomo] -->|Data| L
    end

    subgraph "Infrastructure"
        Q[Hetzner CCX23<br>Cluster] -->|Swarm| R[Docker Swarm<br>Nodes]
        R -->|Hosts| E
        R -->|Hosts| C
        R -->|Hosts| B
        R -->|Hosts| H
        R -->|Hosts| L
        S[Restic<br>Backups] -->|IDrive e2| R
        T[Netdata<br>Monitoring] -->|Metrics| R
    end

    U[Cloudflare<br>DNS Proxy] -->|DDoS, SSL| E

    style "Phase I (Ops)" fill:#e6f3ff,stroke:#0066cc
    style "Phase II (Marketing)" fill:#e6ffe6,stroke:#009900
    style "Phase III (Financial)" fill:#fff2e6,stroke:#ff6600
    style "Infrastructure" fill:#f2f2f2,stroke:#666666
```

- **Phase I (Blue)**: OLGA drives ops via Rocket.Chat, SurrealDB, GroqCloud.
- **Phase II (Green)**: EMMA adds marketing with Matomo, YOURLS, Postiz, Systeme.io.
- **Phase III (Orange)**: RAIFA handles financials with APIs, integrates with Rocket.Chat.
- **Infrastructure (Gray)**: Hetzner CCX23 cluster, Swarm, backups, monitoring.

## Deployment
- **Hetzner CCX23 Cluster**:
  - Start: 2 nodes (~$64/month, 8 vCPUs, 32 GB, 320 GB) for Phase I (0-20 properties, ~5 QPS).
  - Scale: Add ~1 node (~$27.09) per phase or ~50-75 properties—e.g., 3 nodes (~$91/month) at 20-100, 4 (~$118/month) at 100+.
- **Docker Swarm**:
  - Services: `/opt/olga`, `/opt/emma`, `/opt/raifa` (separate Compose files), plus core/supporting tools.
  - Command: `docker swarm init`, `docker stack deploy -c <app>/docker-compose.yml bnb_<app>` (e.g., `bnb_olga`).
- **Ansible**: Roles in `/opt/<app>`—e.g., `ansible-playbook deploy.yml -t olga`.
- **Networking**:
  - **0-20**: Cloudflare DNS Proxy—DDoS protection, SSL (~10-50ms latency).
  - **20+**: Add Hetzner private network (~$1.03/month)—~0.1-1ms node latency, firewall for security (e.g., RAIFA’s financial data).

## Resource Estimates
- **0-20 Properties**: ~5.5-6 vCPUs, ~1.6-1.8 GB RAM, ~150-200 GB, 2 CCX23 (~$64/month), API ~$20-$50.
- **20-100 Properties**: ~7.5-9 vCPUs, ~2.2-2.5 GB RAM, ~180-220 GB, 3 CCX23 (~$91/month), API ~$50-$150.
- **100+ Properties**: ~11-15 vCPUs, ~3-4 GB RAM, ~200-260 GB, 4 CCX23 (~$118/month), API ~$150-$350.
- **Dev Cost**: ~$6K-$10K (Phase III integration, AI logic).

## Scaling
- **Horizontal**: Add CCX23 nodes in Swarm—e.g., 4 nodes (~16 vCPUs) at 100+ properties, up to ~5-6 (~$145-$172/month) at 500-1,000 properties.
- **Services**: Scale agents independently—e.g., `docker service scale bnb_raifa=3` for financial load.

## Getting Started
1. Provision 2 CCX23 nodes (~$54.18/month) on Hetzner.
2. Install Docker, Swarm: `docker swarm init --advertise-addr 10.0.0.1`.
3. Deploy core services: `docker stack deploy -c traefik/docker-compose.yml bnb_traefik`.
4. Add Phase I: `docker stack deploy -c olga/docker-compose.yml bnb_olga`.
5. Scale with Phases II/III: Add EMMA/RAIFA, more nodes as needed.
6. Configure Cloudflare DNS Proxy—add private network at 20+ properties.

## Future Considerations
- **HA**: Revisit high availability at 500+ properties (e.g., MongoDB/TiKV replication).
- **Custom LLM**: Explore RunPod serverless (~$1/month) for predictive financial modeling if needed.


### Notes on Mermaid Diagram
- **Colors**: Mermaid doesn’t natively support `#fill` in subgraphs, but I’ve added `style` directives with hex codes (blue: `#e6f3ff`, green: `#e6ffe6`, orange: `#fff2e6`, gray: `#f2f2f2`)—render this in a Mermaid-compatible tool (e.g., Mermaid Live Editor) for visual distinction.
- **Layout**: Top-down flow—external (Cloudflare) → Traefik → phased components → infrastructure.

This README and diagram give your team a clear roadmap—OLGA, EMMA, RAIFA shine as the system’s stars!