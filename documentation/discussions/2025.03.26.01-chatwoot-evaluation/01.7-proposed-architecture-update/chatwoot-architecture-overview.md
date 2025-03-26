# Chatwoot Architecture Overview

## System Architecture

The proposed architecture implements Chatwoot as the central communication platform, with PostgreSQL as the database, and integrates AI capabilities through Cloudflare Workers and external services.

```
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                        Client Devices                           │
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                                                                 │
│                      Cloudflare (DNS/CDN)                       │
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Cloudflare                               │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Aircall        │   │  Guesty         │   │  LLM Query      ││
│  │  Webhook Worker │   │  Webhook Worker │   │  Worker         ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Voice          │   │  R2 Storage     │   │  KV Store       ││
│  │  Processing     │   │  (Backups)      │   │  (Secrets)      ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Hetzner Cloud                            │
│                                                                 │
│  ┌─────────────────────────────────────────────────────────────┐│
│  │                     Docker Swarm Cluster                    ││
│  │                                                             ││
│  │  ┌─────────────────┐   ┌─────────────────┐   ┌────────────┐ ││
│  │  │                 │   │                 │   │            │ ││
│  │  │  Chatwoot       │   │  PostgreSQL     │   │  Redis     │ ││
│  │  │                 │   │                 │   │            │ ││
│  │  └─────────────────┘   └─────────────────┘   └────────────┘ ││
│  │                                                             ││
│  │  ┌─────────────────┐   ┌─────────────────┐   ┌────────────┐ ││
│  │  │                 │   │                 │   │            │ ││
│  │  │  Keycloak       │   │  Sync App       │   │  Traefik   │ ││
│  │  │                 │   │                 │   │            │ ││
│  │  └─────────────────┘   └─────────────────┘   └────────────┘ ││
│  │                                                             ││
│  │  ┌─────────────────┐   ┌─────────────────┐                  ││
│  │  │                 │   │                 │                  ││
│  │  │  SeaweedFS      │   │  Netdata        │                  ││
│  │  │                 │   │                 │                  ││
│  │  └─────────────────┘   └─────────────────┘                  ││
│  │                                                             ││
│  └─────────────────────────────────────────────────────────────┘│
│                                                                 │
└───────────────────────────────┬─────────────────────────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                        External Services                         │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Qdrant Cloud   │   │  Groq Cloud     │   │  Azure Speech   ││
│  │  (Vector DB)    │   │  (LLM)          │   │  Services       ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Core Components

### Chatwoot Platform
- **Purpose**: Central communication platform for both team and customer interactions
- **Components**: Web application, API servers, background workers
- **Database**: PostgreSQL (required by Chatwoot)
- **Cache**: Redis for session management and background jobs

### Docker Swarm Cluster
- **Purpose**: Container orchestration for all services
- **Components**: Manager nodes, worker nodes
- **Network**: Overlay networks for service communication
- **Storage**: Volumes for persistent data

### Cloudflare Workers
- **LLM Query Worker**: Processes conversation queries and generates AI responses
- **Aircall Webhook Worker**: Processes call events from Aircall
- **Guesty Webhook Worker**: Processes booking events from Guesty
- **Voice Processing Worker**: Handles voice input/output using Azure Speech Services

### External Services
- **Qdrant Cloud**: Vector database for semantic search
- **Groq Cloud**: LLM provider for generating responses
- **Azure Speech Services**: Speech-to-Text and Text-to-Speech capabilities

## Data Flow

### Customer Inquiry Flow
```
Customer → Chatwoot Inbox → LLM Query Worker → Qdrant → Groq → Chatwoot Inbox → Customer
```

### Voice Interaction Flow
```
User (Speaking) → Chatwoot Voice UI → Voice Processing Worker → Azure STT → 
LLM Query Worker → Qdrant → Groq → 
Voice Processing Worker → Azure TTS → Chatwoot Voice UI → User (Listening)
```

### Aircall Integration Flow
```
Aircall → Aircall Webhook Worker → PostgreSQL → Vectorize Function → Qdrant
```

### Guesty Integration Flow
```
Guesty → Guesty Webhook Worker → PostgreSQL → Vectorize Function → Qdrant
```

## Infrastructure as Code

The entire infrastructure will be managed using:
- **Terraform**: For provisioning cloud resources
- **Ansible**: For configuration management
- **GitHub Actions**: For CI/CD pipelines
- **Git-crypt**: For secrets management

## Security Considerations

- **Authentication**: Keycloak for identity and access management
- **Data Encryption**: TLS for in-transit, encryption at rest for databases
- **API Security**: Token-based authentication, rate limiting
- **Voice Data**: Secure storage with automatic expiration
- **Secrets Management**: Secure storage in Cloudflare KV and git-crypt

## Monitoring and Observability

- **System Monitoring**: Netdata for real-time metrics
- **Application Monitoring**: Custom health checks and metrics
- **Log Management**: Centralized logging
- **Alerting**: Configurable alerts for critical services

## Backup Strategy

- **Database Backups**: Regular PostgreSQL backups to R2 Storage
- **Configuration Backups**: Infrastructure as Code in version control
- **Disaster Recovery**: Documented recovery procedures

## Scaling Considerations

- **Horizontal Scaling**: Add more Docker Swarm nodes as needed
- **Database Scaling**: PostgreSQL read replicas for scaling reads
- **Worker Scaling**: Increase Cloudflare Worker limits as needed
- **Cost Optimization**: Resource allocation based on actual usage