# Infrastructure Architecture and Process Flows

## System Architecture

```mermaid
graph TD
    %% Main Cloud Providers
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
        GCP_Scheduler["Cloud Scheduler"]
        GCP_BackupFunction["Backup Function"]
        GCP_VectorizeFunction["Vectorize Function"]
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
    class GCP_Firestore,GCP_Scheduler,GCP_BackupFunction,GCP_VectorizeFunction gcp;
    class Vercel_SecretsUI vercel;
    class Qdrant,Groq,Aircall,Guesty external;
    class HZ_Network network;
```

## Deployment Process Flow

```mermaid
flowchart TD
    A["Developer\nCommits Changes"] -->|"1. Push to Branch"| B["Create Pull Request"]
    B --> |"2. Trigger CI"| C{"GitHub Actions\nPR Validation"}
    C -->|"3a. Run Ansible Check Mode"| D["Generate Diff"]
    C -->|"3b. Validate Functions"| E["Test Functions Locally"]
    D --> |"4a. Add Results"| F["Post Diff to PR"]
    E --> |"4b. Add Results"| F
    F --> |"5. Review"| G{"PR Review"}
    G -->|"6a. Approved"| H["Merge PR"]
    G -->|"6b. Changes Requested"| I["Update PR"]
    I --> |"7. Re-trigger CI"| C
    H --> |"8. Trigger CD"| J{"GitHub Actions\nDeployment"}
    J -->|"9. Run Terraform"| K["Provision/Update\nInfrastructure"]
    K --> |"10. Configure"| L["Run Ansible\nPlaybooks"]
    L --> |"11. Deploy"| M["Deploy Serverless Functions"]
    M --> |"12. Check"| N{"Deployment\nSuccessful?"}
    N -->|"13a. Yes"| O["Verify Services"]
    N -->|"13b. No"| P["Automatic Rollback"]
    P --> |"14. Report"| Q["Create Issue\nfor Failed Deployment"]
    O -->|"15a. All Healthy"| R["Deployment Complete"]
    O -->|"15b. Issues Detected"| S["Manual Intervention"]
    S --> |"16. Fix"| T["Fix Issues"]
    T --> |"17. Commit"| U["Update Repository"]
    U --> |"18. Restart Process"| A
```

## Configuration Management Flow

```mermaid
flowchart TD
    A["Git Repository"] --> |"1a. Secure"| B{"git-crypt"}
    B -->|"2a. Encrypt"| C["Encrypted Secrets"]
    B -->|"2b. Decrypt"| D["Decrypted Secrets"]
    
    A --> |"1b. Store"| E["Ansible Templates"]
    A --> |"1c. Define"| F["Environment Variables"]
    
    subgraph "GitHub Actions"
        G["Checkout Code"]
        H["Decrypt Secrets"]
        I["Process Templates"]
        J["Deploy to Swarm"]
    end
    
    G --> |"3. Get"| H
    H --> |"4. Provide"| I
    I --> |"5. Execute"| J
    
    D --> |"6. Use"| H
    E --> |"7a. Use"| I
    F --> |"7b. Use"| I
    
    J --> |"8. Configure"| K["Docker Swarm"]
    
    K --> |"9a. Start"| L["Core Services"]
    K --> |"9b. Start"| M["Applications"]
    
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
    A["Google Cloud Scheduler"] -->|"1. Daily Trigger"| B["Backup Cloud Function"]
    B --> |"2. Provision"| C["Create Server from Snapshot"]
    C --> |"3. Initialize"| D["Server Boots with Ansible Configuration"]
    
    D --> |"4. Execute"| E["Restic Runs Backup Based on Configuration"]
    E --> |"5. Store"| F["Upload to Cloudflare R2"]
    
    F --> |"6. Manage"| G["Apply Retention Policy"]
    G -->|"7a. Daily"| H["Keep 7 Days"]
    G -->|"7b. Weekly"| I["Keep 4 Weeks"]
    G -->|"7c. Monthly"| J["Keep 12 Months"]
    
    E --> |"8. Record"| K["Update Status in Firestore"]
    K --> |"9. Cleanup"| L["Self-Destruct Server"]
    
    M["Monitoring Checks Status"] --> |"10. Verify"| N{"Backup Successful?"}
    N -->|"11a. Yes"| O["Log Success"]
    N -->|"11b. No"| P["Alert Team"]
    
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
    A["Developer"] -->|"1. Authenticate with GPG Key"| B["git-crypt"]
    B -->|"2. Encrypt"| C["Encrypted Secrets\nin Repository"]
    
    D["GitHub Actions"] -->|"3. Provide Symmetric Key"| E["git-crypt unlock"]
    E --> |"4. Decrypt"| F["Decrypted Secrets"]
    
    F --> |"5. Create"| G["Docker Secrets"]
    G --> |"6. Inject into"| H["Docker Swarm Services"]
    
    I["Vercel Secrets UI"] --> |"7. Access"| J["View/Edit Secrets"]
    J --> |"8. Commit Changes"| K["Update Repository"]
    K --> |"9. Re-encrypt"| B
    
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
    A1["Aircall API"] -->|"1a. Send Webhook Event"| B1["Aircall Cloudflare Worker"]
    A2["Guesty API"] -->|"1b. Send Webhook Event"| B2["Guesty Cloudflare Worker"]
    
    B1 -->|"2a. Validate & Transform"| C["Firestore Database"]
    B2 -->|"2b. Validate & Transform"| C
    
    C -->|"3. Trigger on Document Write"| D["Vectorize Cloud Function"]
    D -->|"4. Create Vector Representation"| E["Qdrant Vector Database"]
    
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
    A["RocketChat User"] -->|"1. Ask Question"| B["RocketChat"]
    B -->|"2. Send Webhook"| C["LLM Query Cloudflare Worker"]
    
    C -->|"3. Query for Context"| D["Qdrant Vector Database"]
    D -->|"4. Return Relevant Context"| C
    
    C -->|"5. Query with Context"| E["Groq LLM API"]
    E -->|"6. Return Response"| C
    
    C -->|"7. Format Response"| B
    B -->|"8. Display Answer"| A
    
    C -->|"9. Store Q&A"| F["Firestore"]
    F -->|"10. Trigger Function"| G["Vectorize Function"]
    G -->|"11. Update Vector DB"| D
    
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