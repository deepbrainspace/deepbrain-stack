```mermaid
flowchart TD
    classDef sgExternal fill:#FAFAFA,stroke:#AAAAAA;
    classDef sgCloudflare fill:#FFF8E1,stroke:#F38020;
    classDef sgVercel fill:#F5F5F5,stroke:#000000;
    classDef sgGCP fill:#E3F2FD,stroke:#4285F4;
    classDef sgHetzner fill:#F5F5F5,stroke:#D50C2D;
    classDef sgFirewall fill:#FDEDE8,stroke:#D50C2D,stroke-dasharray: 5 5;
    classDef sgAI fill:#F3E5F5,stroke:#9C27B0;
    classDef nodeExternal fill:#FFFFFF,stroke:#AAAAAA;
    classDef nodeCloudflare fill:#FFF3E0,stroke:#F38020;
    classDef nodeVercel fill:#FFFFFF,stroke:#000000;
    classDef nodeGCP fill:#E3F2FD,stroke:#4285F4;
    classDef nodeHetznerCore fill:#E8F5E9,stroke:#4CAF50;
    classDef nodeHetznerApp fill:#E1F5FE,stroke:#03A9F4;
    classDef nodeHetznerSwarm fill:#FFFFFF,stroke:#2496ED;
    classDef nodeDB fill:#FFFFE0,stroke:#BDBDBD;
    classDef nodeGroq fill:#FFEBEE,stroke:#FF4081;
    classDef nodeQdrant fill:#E0F7FA,stroke:#00BCD4;

    subgraph External_APIs [External APIs]
        direction LR
        Ext_Aircall("Aircall API"):::nodeExternal
        Ext_Guesty("Guesty API"):::nodeExternal
    end

    subgraph Cloudflare_Services [Cloudflare]
        CF_DNS["DNS / Proxy"]:::nodeCloudflare
        CF_R2[("R2 Storage")]:::nodeDB
        CF_KV[("KV Secrets")]:::nodeDB
        CF_AircallWorker("Aircall Worker"):::nodeCloudflare
        CF_GuestyWorker("Guesty Worker"):::nodeCloudflare
        CF_LLMWorker("LLM Query Worker"):::nodeCloudflare
    end

    subgraph Vercel_Service [Vercel]
        Vercel_SecretsUI["Secrets UI"]:::nodeVercel
    end

    subgraph GCP_Services [GCP Services]
        GCP_Scheduler["Cloud Scheduler"]:::nodeGCP
        GCP_BackupFunc("Backup Function"):::nodeGCP
        GCP_VectorizeFunc("Vectorize Function"):::nodeGCP
        GCP_PubSub["Pub/Sub Events"]:::nodeGCP
        GCP_Firestore[("Firestore DB")]:::nodeDB
    end

    subgraph Hetzner_Infra [Core Infrastructure - Hetzner]
        subgraph HZ_Firewall [Firewall Zone]
            Core_Traefik["Traefik Ingress"]:::nodeHetznerCore
            subgraph Swarm [Docker Swarm Cluster]
                Node1["Node 1"]:::nodeHetznerSwarm
                Node2["Node 2"]:::nodeHetznerSwarm
                Node3["Node 3"]:::nodeHetznerSwarm
                subgraph Core_Services [Core Services]
                   Core_SeaweedFS[("SeaweedFS")]:::nodeDB
                   Core_Netdata["Netdata Monitor"]:::nodeHetznerCore
                end
                subgraph Applications [Applications]
                   App_RocketChat["RocketChat"]:::nodeHetznerApp
                   App_MongoDB[("MongoDB")]:::nodeDB
                   App_Keycloak["Keycloak Auth"]:::nodeHetznerApp
                end
            end
        end
    end

    subgraph AI_Services [Managed AI Services]
        direction LR
        Qdrant[("Qdrant Vector DB")]:::nodeQdrant
        Groq("Groq LLM Inference"):::nodeGroq
    end

    External_APIs --> Cloudflare_Services;
    Vercel_Service --> Cloudflare_Services;
    Cloudflare_Services --> GCP_Services;
    Cloudflare_Services --> Hetzner_Infra;
    GCP_Services --> Hetzner_Infra;
    GCP_Services --> AI_Services;
    Hetzner_Infra --> GCP_Services;
    Hetzner_Infra --> Cloudflare_Services;
    Hetzner_Infra --> AI_Services;

    CF_DNS --> Core_Traefik;
    Vercel_SecretsUI -- "Manages Secrets --> KV" --> CF_KV;
    Ext_Aircall -- "Webhook" --> CF_AircallWorker;
    Ext_Guesty -- "Webhook" --> CF_GuestyWorker;
    CF_AircallWorker -- "Store Data" --> GCP_Firestore;
    CF_GuestyWorker -- "Store Data" --> GCP_Firestore;
    GCP_Firestore -- "Trigger (Write)" --> GCP_VectorizeFunc;
    GCP_VectorizeFunc -- "Vectorize/Upsert" --> Qdrant;
    App_MongoDB -- "Change Stream" --> GCP_PubSub;
    GCP_PubSub -- "Trigger" --> GCP_VectorizeFunc;
    GCP_Scheduler -- "Trigger" --> GCP_BackupFunc;
    GCP_BackupFunc -- "Orchestrate" --> Hetzner_Infra;
    GCP_BackupFunc -- "Log Status" --> GCP_Firestore;
    Hetzner_Infra -- "Store Backup --> R2" --> CF_R2;
    App_RocketChat -- "User Query --> Webhook" --> CF_LLMWorker;
    CF_LLMWorker -- "1. Get Context" --> Qdrant;
    CF_LLMWorker -- "2. Generate" --> Groq;
    Groq -- "LLM Response" --> CF_LLMWorker;
    CF_LLMWorker -- "3. Send Response" --> App_RocketChat;
    CF_LLMWorker -- "4. Store Q&A?" --> GCP_Firestore;
    Core_Traefik --> Applications;
    Applications -- "Auth Via" --> App_Keycloak;
    Applications -- "R/W App Data" --> GCP_Firestore;

    class External_APIs sgExternal;
    class Cloudflare_Services sgCloudflare;
    class Vercel_Service sgVercel;
    class GCP_Services sgGCP;
    class Hetzner_Infra sgHetzner;
    class HZ_Firewall sgFirewall;
    class AI_Services sgAI;