# Milestone 2: AI Integration and Data Migration - Architecture Details

## Database Schema Design

### PostgreSQL Schema for Chatwoot and Operational Data

```
┌─────────────────────────────────────────────────────────────────┐
│                     PostgreSQL Database                         │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Chatwoot       │   │  Operational    │   │  Migration      ││
│  │  Schema         │   │  Schema         │   │  Schema         ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Operational Schema Design

```sql
-- Property Information
CREATE TABLE properties (
    id SERIAL PRIMARY KEY,
    guesty_id VARCHAR(50) UNIQUE,
    name VARCHAR(255) NOT NULL,
    address TEXT,
    city VARCHAR(100),
    state VARCHAR(100),
    country VARCHAR(100),
    zip_code VARCHAR(20),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Reservation Information
CREATE TABLE reservations (
    id SERIAL PRIMARY KEY,
    guesty_id VARCHAR(50) UNIQUE,
    property_id INTEGER REFERENCES properties(id),
    guest_id INTEGER REFERENCES contacts(id),
    status VARCHAR(50) NOT NULL,
    check_in_date DATE NOT NULL,
    check_out_date DATE NOT NULL,
    total_price DECIMAL(10, 2),
    currency VARCHAR(3),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Call Records
CREATE TABLE call_records (
    id SERIAL PRIMARY KEY,
    aircall_id VARCHAR(50) UNIQUE,
    contact_id INTEGER REFERENCES contacts(id),
    direction VARCHAR(20) NOT NULL,
    status VARCHAR(50) NOT NULL,
    duration INTEGER,
    recording_url TEXT,
    notes TEXT,
    agent_id INTEGER REFERENCES users(id),
    started_at TIMESTAMP WITH TIME ZONE,
    ended_at TIMESTAMP WITH TIME ZONE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Operational Events
CREATE TABLE operational_events (
    id SERIAL PRIMARY KEY,
    event_type VARCHAR(50) NOT NULL,
    source VARCHAR(50) NOT NULL,
    source_id VARCHAR(50),
    payload JSONB NOT NULL,
    processed BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Vector Embeddings Reference
CREATE TABLE vector_references (
    id SERIAL PRIMARY KEY,
    reference_type VARCHAR(50) NOT NULL,
    reference_id INTEGER NOT NULL,
    vector_id VARCHAR(100) NOT NULL,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    UNIQUE(reference_type, reference_id)
);
```

## Data Migration Architecture

### Migration Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     Data Migration Flow                         │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Source Data    │──►│  Migration      │──►│  PostgreSQL     ││
│  │  (MongoDB/      │   │  Service        │   │  Database       ││
│  │   Firestore)    │   │                 │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                │                                 │
│                                ▼                                 │
│                      ┌─────────────────┐                         │
│                      │                 │                         │
│                      │  Validation     │                         │
│                      │  Service        │                         │
│                      │                 │                         │
│                      └─────────────────┘                         │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Migration Service Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Migration Service                           │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Extractor      │──►│  Transformer    │──►│  Loader         ││
│  │  Components     │   │  Components     │   │  Components     ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Error          │   │  Logging        │   │  Monitoring     ││
│  │  Handling       │   │  Component      │   │  Component      ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Cloudflare Workers Adaptation

### Updated LLM Query Worker

```
┌─────────────────────────────────────────────────────────────────┐
│                     LLM Query Worker                            │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Chatwoot       │──►│  Context        │──►│  Qdrant         ││
│  │  Webhook        │   │  Builder        │   │  Client         ││
│  │  Handler        │   │                 │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                │                     │          │
│                                │                     │          │
│                                ▼                     ▼          │
│                      ┌─────────────────┐   ┌─────────────────┐  │
│                      │                 │   │                 │  │
│                      │  Prompt         │◄──┤  Vector         │  │
│                      │  Builder        │   │  Results        │  │
│                      │                 │   │                 │  │
│                      └───────┬─────────┘   └─────────────────┘  │
│                              │                                  │
│                              ▼                                  │
│                      ┌─────────────────┐   ┌─────────────────┐  │
│                      │                 │   │                 │  │
│                      │  Groq           │──►│  Response       │  │
│                      │  Client         │   │  Formatter      │  │
│                      │                 │   │                 │  │
│                      └─────────────────┘   └───────┬─────────┘  │
│                                                    │            │
│                                                    ▼            │
│                                            ┌─────────────────┐  │
│                                            │                 │  │
│                                            │  Chatwoot       │  │
│                                            │  API Client     │  │
│                                            │                 │  │
│                                            └─────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Updated Aircall Webhook Worker

```
┌─────────────────────────────────────────────────────────────────┐
│                     Aircall Webhook Worker                      │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Aircall        │──►│  Event          │──►│  Data           ││
│  │  Webhook        │   │  Processor      │   │  Transformer    ││
│  │  Handler        │   │                 │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  PostgreSQL     ││
│                                              │  Client         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Chatwoot       ││
│                                              │  API Client     ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Updated Guesty Webhook Worker

```
┌─────────────────────────────────────────────────────────────────┐
│                     Guesty Webhook Worker                       │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Guesty         │──►│  Event          │──►│  Data           ││
│  │  Webhook        │   │  Processor      │   │  Transformer    ││
│  │  Handler        │   │                 │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  PostgreSQL     ││
│                                              │  Client         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Chatwoot       ││
│                                              │  API Client     ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Vectorize Function Update

### Updated Vectorize Function

```
┌─────────────────────────────────────────────────────────────────┐
│                     Vectorize Function                          │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  PostgreSQL     │──►│  Data           │──►│  Embedding      ││
│  │  Change         │   │  Processor      │   │  Generator      ││
│  │  Detector       │   │                 │   │                 ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Qdrant         ││
│                                              │  Client         ││
│                                              │                 ││
│                                              └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Vector         ││
│                                              │  Reference      ││
│                                              │  Updater        ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## PostgreSQL Change Detection

### Change Detection Mechanism

```
┌─────────────────────────────────────────────────────────────────┐
│                     Change Detection Flow                       │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  PostgreSQL     │──►│  Logical        │──►│  Change         ││
│  │  Database       │   │  Replication    │   │  Processor      ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Vectorize      ││
│                                              │  Function       ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Logical Replication Configuration

```sql
-- Create publication for relevant tables
CREATE PUBLICATION vectorize_publication FOR TABLE 
    conversations, 
    messages, 
    contacts, 
    properties, 
    reservations, 
    call_records, 
    operational_events;

-- Create a dedicated replication role
CREATE ROLE vectorize_replication WITH REPLICATION LOGIN PASSWORD 'secure_password';

-- Grant necessary permissions
GRANT SELECT ON ALL TABLES IN SCHEMA public TO vectorize_replication;
```

## Chatwoot API Integration

### API Integration Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Chatwoot API Integration                    │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Chatwoot       │◄──┤  API            │◄──┤  Cloudflare     ││
│  │  Application    │   │  Gateway        │   │  Workers        ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### API Authentication

```
┌─────────────────────────────────────────────────────────────────┐
│                     API Authentication                          │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  API Key        │──►│  JWT            │──►│  Request        ││
│  │  Storage        │   │  Generation     │   │  Authentication ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Data Validation Architecture

### Validation Service

```
┌─────────────────────────────────────────────────────────────────┐
│                     Validation Service                          │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Source Data    │──►│  Target Data    │──►│  Comparison     ││
│  │  Extractor      │   │  Extractor      │   │  Engine         ││
│  │                 │   │                 │   │                 ││
│  └─────────────────┘   └─────────────────┘   └───────┬─────────┘│
│                                                      │          │
│                                                      │          │
│                                                      ▼          │
│                                              ┌─────────────────┐│
│                                              │                 ││
│                                              │  Reporting      ││
│                                              │  Component      ││
│                                              │                 ││
│                                              └─────────────────┘│
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Validation Metrics

```
┌─────────────────────────────────────────────────────────────────┐
│                     Validation Metrics                          │
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐│
│  │                 │   │                 │   │                 ││
│  │  Record Count   │   │  Data           │   │  Relationship   ││
│  │  Validation     │   │  Integrity      │   │  Validation     ││
│  │                 │   │  Validation     │   │                 ││
│  └─────────────────┘   └─────────────────┘   └─────────────────┘│
│                                                                 │
│  ┌─────────────────┐   ┌─────────────────┐                      │
│  │                 │   │                 │                      │
│  │  Schema         │   │  Business Rule  │                      │
│  │  Validation     │   │  Validation     │                      │
│  │                 │   │                 │                      │
│  └─────────────────┘   └─────────────────┘                      │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```