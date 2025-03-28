```mermaid
graph TD
    %% Title
    %% System Architecture (Revised)

    %% Define Styles
    classDef cloudflare fill:#F6821F,color:white;
    classDef hetzner fill:#D50C2D,color:white;
    classDef firewall fill:#D50C2D,color:white,stroke-width:4px,stroke-dasharray: 5 5;
    classDef gcp fill:#4285F4,color:white;
    classDef vercel fill:#000000,color:white;
    classDef external fill:#666666,color:white;
    classDef qdrant fill:#656666,color:white;
    classDef groq fill:#666366,color:white;
    classDef swarm fill:#2496ED,color:white;
    classDef core fill:#2496ED,color:white,stroke-dasharray: 3 3;
    classDef apps fill:#2496ED,color:white,stroke-dasharray: 3 3;
    classDef ai fill:#9C27B0,color:white;

    %% External Inputs / Edge Layer
    subgraph External_APIs [External APIs]
        direction LR
        Ext_Aircall["Aircall API"]
        Ext_Guesty["Guesty API"]
    end
    class Ext_Aircall, Ext_Guesty external;

    subgraph Edge_Layer [Cloudflare & Vercel]
        direction LR
        subgraph Cloudflare
           CF_DNS["DNS / Proxy"]
           CF_Workers["Workers"]
           CF_R2["R2 (Backup Storage)"]
           CF_KV["KV (Secrets Data)"]
           CF_AircallWorker["Aircall Webhook Worker"]:::cloudflare
           CF_GuestyWorker["Guesty Webhook Worker"]:::cloudflare
           CF_LLMWorker["LLM Query Worker"]:::cloudflare
        end
        subgraph Vercel
            Vercel_SecretsUI["Secrets UI"]
        end
    end
    classDef cloudflare fill:#F6821F,color:white;
    class Cloudflare, CF_DNS, CF_Workers, CF_R2, CF_KV cloudflare;
    classDef vercel fill:#000000,color:white;
    class Vercel, Vercel_SecretsUI vercel;

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
    classDef hetzner fill:#D50C2D,color:white;
    class Hetzner, HZ_Firewall hetzner;
    classDef swarm fill:#2496ED,color:white;
    class Swarm, Node1, Node2, Node3 swarm;
    classDef core fill:#ADD8E6,color:black;
    class Core_Traefik, Core_Services, Core_SeaweedFS, Core_Netdata core;
    classDef apps fill:#E6E6FA,color:black;
    class Applications, App_RocketChat, App_MongoDB, App_Keycloak apps;


    %% Backend Services (GCP)
    subgraph GCP [GCP Services]
        direction TB
        GCP_Firestore["Firestore (App Data / Status)"]
        GCP_Scheduler["Cloud Scheduler (Triggers)"]
        GCP_BackupFunc["Backup Function"]
        GCP_VectorizeFunc["Vectorize Function"]
        GCP_PubSub["Pub/Sub (Events)"]
    end
    classDef gcp fill:#4285F4,color:white;
    class GCP, GCP_Firestore, GCP_Scheduler, GCP_BackupFunc, GCP_VectorizeFunc, GCP_PubSub gcp;


    %% AI Services (Managed)
    subgraph AI_Services [Managed AI Services]
        direction LR
        Qdrant["Qdrant Cloud (Vector DB)"]
        Groq["Groq Cloud (LLM Inference)"]
    end
    classDef ai fill:#9C27B0,color:white;
    class AI_Services, Qdrant, Groq ai;


    %% Define Primary Flows / Connections

    %% Ingress Flow
    CF_DNS --> Core_Traefik;

    %% Webhook Data Ingestion Flow
    Ext_Aircall -- "Webhook" --> CF_AircallWorker;
    Ext_Guesty -- "Webhook" --> CF_GuestyWorker;
    CF_AircallWorker -- "Store Transformed Data" --> GCP_Firestore;
    CF_GuestyWorker -- "Store Transformed Data" --> GCP_Firestore;
    GCP_Firestore -- "Trigger (on write)" --> GCP_VectorizeFunc;
    %% Alternate RocketChat Data Ingestion (via Listener not shown explicitly, triggers PubSub)
    App_MongoDB -- "Change Stream (Implied Listener)" --> GCP_PubSub
    GCP_PubSub -- "Trigger" --> GCP_VectorizeFunc
    GCP_VectorizeFunc -- "Vectorize & Upsert" --> Qdrant;

    %% LLM Query Flow
    App_RocketChat -- "User Query (via Webhook)" --> CF_LLMWorker;
    CF_LLMWorker -- "1. Retrieve Context" --> Qdrant;
    CF_LLMWorker -- "2. Generate Response" --> Groq;
    CF_LLMWorker -- "3. Send Response" --> App_RocketChat;
    CF_LLMWorker -- "4. Store Q&A (Optional)" --> GCP_Firestore; %% Feedback loop for potential re-vectorization

    %% Backup Flow
    GCP_Scheduler -- "Trigger" --> GCP_BackupFunc;
    GCP_BackupFunc -- "Orchestrates Hetzner Server Creation/Snapshot" --> Hetzner;
    Hetzner -- "Stores Backup via Restic (Implied)" --> CF_R2;
    GCP_BackupFunc -- "Logs Status" --> GCP_Firestore;

    %% Secrets Management Flow
    Vercel_SecretsUI -- "Manages Secrets Data In" --> CF_KV;
    %% Note: CF Workers / Other services access KV as needed (connection not explicitly drawn to reduce clutter)

    %% General Service Connections
    Core_Traefik -- "Routes Traffic To" --> Applications;
    Applications -- "Authenticate Via" --> App_Keycloak;
    Applications -- "Read/Write Data" --> GCP_Firestore %% Direct interaction besides webhook triggers


    %% Apply Styles to Subgraphs/Nodes not covered by defaults
    classDef firewall fill:#D50C2D,color:white,stroke-width:4px,stroke-dasharray: 5 5;
    class HZ_Firewall firewall;
```