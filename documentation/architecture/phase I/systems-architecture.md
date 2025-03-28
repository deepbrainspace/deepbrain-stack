```mermaid
graph TD
    %% System Architecture (Revised - Attempt 2)

    %% Define Styles (ALL AT THE TOP)
    classDef cloudflare fill:#F6821F,color:white;
    classDef hetzner fill:#D50C2D,color:white;
    classDef firewall fill:#D50C2D,color:white,stroke-width:4px,stroke-dasharray: 5 5;
    classDef gcp fill:#4285F4,color:white;
    classDef vercel fill:#000000,color:white;
    classDef external fill:#666666,color:white;
    classDef qdrant fill:#656666,color:white; // Style for Qdrant node
    classDef groq fill:#666366,color:white;   // Style for Groq node
    classDef swarm fill:#2496ED,color:white;
    classDef core fill:#ADD8E6,color:black;   // Style for Core Hetzner Services
    classDef apps fill:#E6E6FA,color:black;   // Style for Apps on Hetzner
    classDef ai fill:#9C27B0,color:white;     // Style for AI Services subgraph

    %% External Inputs / Edge Layer
    subgraph External_APIs [External APIs]
        direction LR
        Ext_Aircall["Aircall API"]:::external
        Ext_Guesty["Guesty API"]:::external
    end

    subgraph Edge_Layer [Cloudflare & Vercel]
        direction LR
        subgraph Cloudflare
           CF_DNS["DNS / Proxy"]:::cloudflare
           CF_Workers["Workers (Generic)"]:::cloudflare
           CF_R2["R2 (Backup Storage)"]:::cloudflare
           CF_KV["KV (Secrets Data)"]:::cloudflare
           CF_AircallWorker["Aircall Webhook Worker"]:::cloudflare
           CF_GuestyWorker["Guesty Webhook Worker"]:::cloudflare
           CF_LLMWorker["LLM Query Worker"]:::cloudflare
        end
        subgraph Vercel
            Vercel_SecretsUI["Secrets UI"]:::vercel
        end
    end

    %% Core Infrastructure (Hetzner)
    subgraph Hetzner [Hetzner Cloud]
        subgraph HZ_Firewall [Firewall Zone]
            direction TB
            Core_Traefik["Traefik (Ingress)"]:::core
            subgraph Swarm [Docker Swarm Cluster]
                direction TB
                Node1["Node 1"]:::swarm
                Node2["Node 2"]:::swarm
                Node3["Node 3"]:::swarm
                %% Note: Internal Swarm traffic implied between nodes

                subgraph Core_Services [Core Services]
                   Core_SeaweedFS["SeaweedFS (Storage)"]:::core
                   Core_Netdata["Netdata (Monitoring)"]:::core
                end

                subgraph Applications [Applications]
                   App_RocketChat["RocketChat"]:::apps
                   App_MongoDB["MongoDB (for RocketChat)"]:::apps
                   App_Keycloak["Keycloak (Auth)"]:::apps
                   %% Note: Apps use SeaweedFS for persistent storage
                end
            end
        end
    end
    class HZ_Firewall firewall; % Apply firewall style to the zone subgraph

    %% Backend Services (GCP)
    subgraph GCP [GCP Services]
        direction TB
        GCP_Firestore["Firestore (App Data / Status)"]:::gcp
        GCP_Scheduler["Cloud Scheduler (Triggers)"]:::gcp
        GCP_BackupFunc["Backup Function"]:::gcp
        GCP_VectorizeFunc["Vectorize Function"]:::gcp
        GCP_PubSub["Pub/Sub (Events)"]:::gcp
    end

    %% AI Services (Managed)
    subgraph AI_Services [Managed AI Services]
        direction LR
        Qdrant["Qdrant Cloud (Vector DB)"]:::qdrant
        Groq["Groq Cloud (LLM Inference)"]:::groq
    end
    class AI_Services ai; % Style the AI Services subgraph


    %% --- Define Primary Flows / Connections ---

    %% Ingress Flow
    CF_DNS --> Core_Traefik;

    %% Webhook Data Ingestion Flow
    Ext_Aircall -- "Webhook" --> CF_AircallWorker;
    Ext_Guesty -- "Webhook" --> CF_GuestyWorker;
    CF_AircallWorker -- "Store Transformed Data" --> GCP_Firestore;
    CF_GuestyWorker -- "Store Transformed Data" --> GCP_Firestore;
    GCP_Firestore -- "Trigger (on write)" --> GCP_VectorizeFunc;
    %% Alternate RocketChat Data Ingestion (via Listener triggering PubSub)
    App_MongoDB -- "Change Stream (Implied Listener)" --> GCP_PubSub;
    GCP_PubSub -- "Trigger" --> GCP_VectorizeFunc;
    GCP_VectorizeFunc -- "Vectorize & Upsert" --> Qdrant;

    %% LLM Query Flow
    App_RocketChat -- "User Query (via Webhook)" --> CF_LLMWorker;
    CF_LLMWorker -- "1. Retrieve Context" --> Qdrant;
    CF_LLMWorker -- "2. Generate Response" --> Groq;
    CF_LLMWorker -- "3. Send Response" --> App_RocketChat;
    CF_LLMWorker -- "4. Store Q&A (Optional)" --> GCP_Firestore; %% Feedback loop for potential re-vectorization

    %% Backup Flow
    GCP_Scheduler -- "Trigger" --> GCP_BackupFunc;
    GCP_BackupFunc -- "Orchestrates Hetzner Server Creation/Snapshot" --> Hetzner; %% Simplified interaction point
    Hetzner -- "Stores Backup via Restic (Implied Process)" --> CF_R2; %% Simplified interaction point
    GCP_BackupFunc -- "Logs Status" --> GCP_Firestore;

    %% Secrets Management Flow
    Vercel_SecretsUI -- "Manages Secrets Data In" --> CF_KV;
    %% Note: CF Workers / Other services access KV as needed (connection not explicitly drawn to reduce clutter)

    %% General Service Connections
    Core_Traefik -- "Routes Traffic To" --> Applications;
    Applications -- "Authenticate Via" --> App_Keycloak;
    Applications -- "Read/Write Data" --> GCP_Firestore; %% Direct interaction besides webhook triggers
```