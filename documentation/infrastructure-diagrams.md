# Infrastructure Architecture and Process Flows

## System Architecture

```mermaid
graph TB
    subgraph "Cloudflare"
        CF_DNS["DNS"]
        CF_R2["R2 Storage"]
        CF_Workers["Workers"]
    end

    subgraph "Hetzner Cloud"
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
                App_RocketChat["RocketChat"]
                App_MongoDB["MongoDB"]
                App_Keycloak["Keycloak"]
                App_Sync["Sync App"]
            end
        end
        
        HZ_Backup["Backup Server"]
        HZ_Network["Private Network"]
        HZ_Firewall["Firewall"]
    end

    subgraph "GCP"
        GCP_Firestore["Firestore"]
        GCP_Functions["Cloud Functions"]
        GCP_CloudRun["Cloud Run"]
    end

    subgraph "Vercel"
        Vercel_SecretsUI["Secrets UI"]
    end

    subgraph "External Services"
        Qdrant["Qdrant Cloud\nVector DB"]
        Groq["Groq Cloud\nInference"]
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
    HZ_Backup <--> HZ_Network
    
    Core_SeaweedFS <--> App_RocketChat
    Core_SeaweedFS <--> App_MongoDB
    Core_SeaweedFS <--> App_Keycloak
    Core_SeaweedFS <--> App_Sync
    
    Core_Traefik --> HZ_Firewall
    
    HZ_Backup --> CF_R2
    
    GCP_Functions <--> DS_Node1
    GCP_CloudRun <--> DS_Node1
    
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
    
    class CF_DNS,CF_R2,CF_Workers cloudflare;
    class DS_Node1,DS_Node2,DS_Node3,HZ_Backup,HZ_Network,HZ_Firewall hetzner;
    class Core_SeaweedFS,Core_Traefik,Core_Netdata core;
    class App_RocketChat,App_MongoDB,App_Keycloak,App_Sync apps;
    class GCP_Firestore,GCP_Functions,GCP_CloudRun gcp;
    class Vercel_SecretsUI vercel;
    class Qdrant,Groq external;
```

## Deployment Process Flow

```mermaid
flowchart TD
    A[Developer\nCommits Changes] -->|Push to Branch| B[Create Pull Request]
    B --> C{GitHub Actions\nPR Validation}
    C -->|Run Ansible Check Mode| D[Generate Diff]
    D --> E[Post Diff to PR]
    E --> F{PR Review}
    F -->|Approved| G[Merge PR]
    F -->|Changes Requested| H[Update PR]
    H --> C
    G --> I{GitHub Actions\nDeployment}
    I -->|Run Terraform| J[Provision/Update\nInfrastructure]
    J --> K[Run Ansible\nPlaybooks]
    K --> L{Deployment\nSuccessful?}
    L -->|Yes| M[Verify Services]
    L -->|No| N[Automatic Rollback]
    N --> O[Create Issue\nfor Failed Deployment]
    M -->|All Healthy| P[Deployment Complete]
    M -->|Issues Detected| Q[Manual Intervention]
    Q --> R[Fix Issues]
    R --> S[Update Repository]
    S --> A
```

## Configuration Management Flow

```mermaid
flowchart TD
    A[Git Repository] --> B{git-crypt}
    B -->|Encrypt| C[Encrypted Secrets]
    B -->|Decrypt| D[Decrypted Secrets]
    
    A --> E[Ansible Templates]
    A --> F[Environment Variables]
    
    subgraph "GitHub Actions"
        G[Checkout Code]
        H[Decrypt Secrets]
        I[Process Templates]
        J[Deploy to Swarm]
    end
    
    G --> H
    H --> I
    I --> J
    
    D --> H
    E --> I
    F --> I
    
    J --> K[Docker Swarm]
    
    K --> L[Core Services]
    K --> M[Applications]
    
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
    A[Scheduled Backup Job] --> B{Backup Type}
    B -->|Docker Swarm Data| C[Restic on Swarm Nodes]
    B -->|SeaweedFS Volumes| D[Backup Server\nMounts Volumes]
    
    C --> E[Backup /opt/core]
    D --> F[Backup /opt/apps]
    
    E --> G[Cloudflare R2]
    F --> G
    
    G --> H{Retention Policy}
    H -->|Daily| I[Keep 7 Days]
    H -->|Weekly| J[Keep 4 Weeks]
    H -->|Monthly| K[Keep 12 Months]
    
    L[Backup Verification] --> M[Restore Test]
    M --> N{Verification\nSuccessful?}
    N -->|Yes| O[Log Success]
    N -->|No| P[Alert Team]
    
    classDef process fill:#4CAF50,color:white;
    classDef storage fill:#FF9800,color:white;
    classDef decision fill:#9C27B0,color:white;
    classDef alert fill:#F44336,color:white;
    
    class A,C,D,E,F,L,M process;
    class G,I,J,K,O storage;
    class B,H,N decision;
    class P alert;
```

## Secrets Management Flow

```mermaid
flowchart TD
    A[Developer] -->|GPG Key| B[git-crypt]
    B -->|Encrypt| C[Encrypted Secrets\nin Repository]
    
    D[GitHub Actions] -->|Symmetric Key| E[git-crypt unlock]
    E --> F[Decrypted Secrets]
    
    F --> G[Docker Secrets]
    G --> H[Docker Swarm Services]
    
    I[Vercel Secrets UI] --> J[View/Edit Secrets]
    J --> K[Update Repository]
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