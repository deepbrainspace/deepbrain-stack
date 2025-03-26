# Infrastructure Architecture and Process Flows

## System Architecture

```mermaid
graph TB
    subgraph "Cloudflare"
        CF_DNS["DNS"]
        CF_R2["R2 Storage"]
        CF_Workers["Workers"]
        CF_AircallWorker["Aircall Webhook Worker"]
        CF_GuestyWorker["Guesty Webhook Worker"]
        CF_LLMWorker["LLM Query Worker"]
    end

    subgraph "Hetzner Cloud"
        HZ_Firewall["Firewall"]
        
        subgraph "HZ_Network"
            subgraph "Docker Swarm Cluster"
                DS_Node1["Node 1\nManager + Worker"]
                DS_Node2["Node 2\nManager + Worker"]
                DS_Node3["Node 3\nManager + Worker"]
                
                subgraph "Core Services"
                    Core_SeaweedFS["SeaweedFS"]
                    Core_Traefik["Traefik"]
                    Core_Netdata["Netdata"]
                end
                
                subgraph "Applications"
                    subgraph "RocketChat"
                        App_RocketChat["RocketChat"]
                        App_MongoDB["MongoDB"]
                    end
                    App_Keycloak["Keycloak"]
                    App_Sync["Sync App"]
                end
            end
            
            HZ_BackupSnapshot["Backup Server Snapshot"]
        end
    end

    subgraph "GCP"
        GCP_Firestore["Firestore"]
        GCP_Functions["Cloud Functions"]
        GCP_CloudRun["Cloud Run"]
        GCP_Scheduler["Cloud Scheduler"]
        GCP_BackupFunction["Backup Function"]
        GCP_VectorizeFunction["Vectorize Function"]
        GCP_PubSub["Pub/Sub"]
    end

    subgraph "Vercel"
        Vercel_SecretsUI["Secrets UI"]
    end

    subgraph "External Services"
        Qdrant["Qdrant Cloud\nVector DB"]
        Groq["Groq Cloud\nInference"]
        Aircall["Aircall API"]
        Guesty["Guesty API"]
    end

    %% Connections
    CF_DNS --> HZ_Firewall
    CF_Workers --> HZ_Firewall
    
    DS_Node1 <--> DS_Node2
    DS_Node2 <--> DS_Node3
    DS_Node3 <--> DS_Node1
    
    DS_Node1 <--> HZ_Network
    DS_Node2 <--> HZ_Network
    DS_Node3 <--> HZ_Network
    
    Core_SeaweedFS <--> App_RocketChat
    Core_SeaweedFS <--> App_MongoDB
    Core_SeaweedFS <--> App_Keycloak
    Core_SeaweedFS <--> App_Sync
    
    Core_Traefik --> HZ_Firewall
    
    GCP_BackupFunction --> HZ_BackupSnapshot
    HZ_BackupSnapshot --> CF_R2
    
    GCP_Scheduler --> GCP_BackupFunction
    
    Aircall --> CF_AircallWorker
    Guesty --> CF_GuestyWorker
    
    CF_AircallWorker --> GCP_Firestore
    CF_GuestyWorker --> GCP_Firestore
    
    GCP_Firestore --> GCP_VectorizeFunction
    GCP_VectorizeFunction --> Qdrant
    
    App_RocketChat --> CF_LLMWorker
    CF_LLMWorker --> Qdrant
    CF_LLMWorker --> Groq
    CF_LLMWorker --> App_RocketChat
    
    App_RocketChat <--> GCP_Firestore
    App_Sync <--> GCP_Firestore
    
    App_Sync <--> Qdrant
    App_Sync <--> Groq
    
    Vercel_SecretsUI <--> CF_Workers
    
    classDef cloudflare fill:#F6821F,color:white;
    classDef hetzner fill:#D50C2D,color:white;
    classDef gcp fill:#4285F4,color:white;
    classDef vercel fill:#000000,color:white;
    classDef external fill:#666666,color:white;
    classDef swarm fill:#2496ED,color:white;
    classDef core fill:#2496ED,color:white,stroke-dasharray: 5 5;
    classDef apps fill:#2496ED,color:white,stroke-dasharray: 5 5;
    classDef network fill:#009688,color:white;
    
    class CF_DNS,CF_R2,CF_Workers,CF_AircallWorker,CF_GuestyWorker,CF_LLMWorker cloudflare;
    class DS_Node1,DS_Node2,DS_Node3,HZ_BackupSnapshot,HZ_Firewall hetzner;
    class Core_SeaweedFS,Core_Traefik,Core_Netdata core;
    class App_RocketChat,App_MongoDB,App_Keycloak,App_Sync apps;
    class GCP_Firestore,GCP_Functions,GCP_CloudRun,GCP_Scheduler,GCP_BackupFunction,GCP_VectorizeFunction,GCP_PubSub gcp;
    class Vercel_SecretsUI vercel;
    class Qdrant,Groq,Aircall,Guesty external;
    class HZ_Network network;
```

## Deployment Process Flow

```mermaid
flowchart TD
    A["Developer\nCommits Changes"] -->|Push to Branch| B["Create Pull Request"]
    B --> C{"GitHub Actions\nPR Validation"}
    C -->|Run Ansible Check Mode| D["Generate Diff"]
    C -->|Validate Functions| E["Test Functions Locally"]
    D --> F["Post Diff to PR"]
    E --> F
    F --> G{"PR Review"}
    G -->|Approved| H["Merge PR"]
    G -->|Changes Requested| I["Update PR"]
    I --> C
    H --> J{"GitHub Actions\nDeployment"}
    J -->|Run Terraform| K["Provision/Update\nInfrastructure"]
    K --> L["Run Ansible\nPlaybooks"]
    L --> M["Deploy Serverless Functions"]
    M --> N{"Deployment\nSuccessful?"}
    N -->|Yes| O["Verify Services"]
    N -->|No| P["Automatic Rollback"]
    P --> Q["Create Issue\nfor Failed Deployment"]
    O -->|All Healthy| R["Deployment Complete"]
    O -->|Issues Detected| S["Manual Intervention"]
    S --> T["Fix Issues"]
    T --> U["Update Repository"]
    U --> A
```

## Configuration Management Flow

```mermaid
flowchart TD
    A["Git Repository"] --> B{"git-crypt"}
    B -->|Encrypt| C["Encrypted Secrets"]
    B -->|Decrypt| D["Decrypted Secrets"]
    
    A --> E["Ansible Templates"]
    A --> F["Environment Variables"]
    
    subgraph "GitHub Actions"
        G["Checkout Code"]
        H["Decrypt Secrets"]
        I["Process Templates"]
        J["Deploy to Swarm"]
    end
    
    G --> H
    H --> I
    I --> J
    
    D --> H
    E --> I
    F --> I
    
    J --> K["Docker Swarm"]
    
    K --> L["Core Services"]
    K --> M["Applications"]
    
    classDef git fill:#F05033,color:white;
    classDef actions fill:#2088FF,color:white;
    classDef swarm fill:#2496ED,color:white;
    
    class A,B,C,D,E,F git;
    class G,H,I,J actions;
    class K,L,M swarm;
```

## Backup Process Flow

```mermaid
flowchart TD
    A["Google Cloud Scheduler"] -->|Daily Trigger| B["Backup Cloud Function"]
    B --> C["Create Server from Snapshot"]
    C --> D["Server Boots with Ansible Configuration"]
    
    D --> E["Restic Runs Backup Based on Configuration"]
    E --> F["Upload to Cloudflare R2"]
    
    F --> G["Apply Retention Policy"]
    G -->|Daily| H["Keep 7 Days"]
    G -->|Weekly| I["Keep 4 Weeks"]
    G -->|Monthly| J["Keep 12 Months"]
    
    E --> K["Update Status in Firestore"]
    K --> L["Self-Destruct Server"]
    
    M["Monitoring Checks Status"] --> N{"Backup Successful?"}
    N -->|Yes| O["Log Success"]
    N -->|No| P["Alert Team"]
    
    classDef gcp fill:#4285F4,color:white;
    classDef hetzner fill:#D50C2D,color:white;
    classDef cloudflare fill:#F6821F,color:white;
    classDef process fill:#4CAF50,color:white;
    classDef decision fill:#9C27B0,color:white;
    
    class A,B gcp;
    class C,D,E,L hetzner;
    class F,G,H,I,J,O cloudflare;
    class K,M process;
    class N decision;
    class P alert;
```

## Secrets Management Flow

```mermaid
flowchart TD
    A["Developer"] -->|GPG Key| B["git-crypt"]
    B -->|Encrypt| C["Encrypted Secrets\nin Repository"]
    
    D["GitHub Actions"] -->|Symmetric Key| E["git-crypt unlock"]
    E --> F["Decrypted Secrets"]
    
    F --> G["Docker Secrets"]
    G --> H["Docker Swarm Services"]
    
    I["Vercel Secrets UI"] --> J["View/Edit Secrets"]
    J --> K["Update Repository"]
    K --> B
    
    classDef user fill:#1976D2,color:white;
    classDef crypto fill:#FFC107,color:white;
    classDef storage fill:#FF9800,color:white;
    classDef service fill:#4CAF50,color:white;
    
    class A,I user;
    class B,E crypto;
    class C,F,G storage;
    class D,H,J,K service;
```

## API Integration Flow

```mermaid
flowchart TD
    A1["Aircall API"] -->|Webhook Event| B1["Aircall Cloudflare Worker"]
    A2["Guesty API"] -->|Webhook Event| B2["Guesty Cloudflare Worker"]
    
    B1 -->|Validate & Transform| C["Firestore Database"]
    B2 -->|Validate & Transform| C
    
    C -->|Document Write Trigger| D["Vectorize Cloud Function"]
    D -->|Create Vector Representation| E["Qdrant Vector Database"]
    
    classDef external fill:#666666,color:white;
    classDef cloudflare fill:#F6821F,color:white;
    classDef gcp fill:#4285F4,color:white;
    classDef qdrant fill:#00C4CC,color:white;
    
    class A1,A2 external;
    class B1,B2 cloudflare;
    class C,D gcp;
    class E qdrant;
```



## LLM Query Process Flow

```mermaid
flowchart TD
    A["RocketChat User"] -->|Ask Question| B["RocketChat"]
    B -->|Webhook| C["LLM Query Cloudflare Worker"]
    
    C -->|Query for Context| D["Qdrant Vector Database"]
    D -->|Return Relevant Context| C
    
    C -->|Query with Context| E["Groq LLM API"]
    E -->|Return Response| C
    
    C -->|Format Response| B
    B -->|Display Answer| A
    
    C -->|Store Q&A| F["Firestore"]
    F -->|Trigger| G["Vectorize Function"]
    G -->|Update Vector DB| D
    
    classDef user fill:#1976D2,color:white;
    classDef app fill:#2496ED,color:white;
    classDef cloudflare fill:#F6821F,color:white;
    classDef qdrant fill:#00C4CC,color:white;
    classDef groq fill:#FF4081,color:white;
    classDef gcp fill:#4285F4,color:white;
    
    class A user;
    class B app;
    class C cloudflare;
    class D qdrant;
    class E groq;
    class F,G gcp;
```