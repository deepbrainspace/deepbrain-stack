Here’s a comparison table of serverless providers and their features, covering AWS, Google Cloud Platform (GCP), Azure, Vercel, Cloudflare, and Bunny.net. I’ve included Vercel’s Serverless Functions and regular hosting (with free options), as well as GCP’s Cloud Functions, Cloud Run, Google App Engine (GAE), and Google Kubernetes Engine (GKE), alongside similar offerings from AWS and Azure. This matrix focuses on key attributes relevant to your Aircall-to-Guesty use case and broader application types, so we can later map specific workloads to the best fit.

| **Provider/Service**       | **Free Tier**                  | **Runtime Limit**         | **Languages**                  | **Deployment**            | **Edge/Regional** | **Scaling**         | **Triggers**                  | **Key Features**                          | **Cost Beyond Free**          |
|----------------------------|--------------------------------|---------------------------|--------------------------------|---------------------------|-------------------|---------------------|-------------------------------|-------------------------------------------|-------------------------------|
| **AWS Lambda**            | 1M reqs/mo, 400k GB-s         | 15 min                    | Node.js, Python, Java, C#, Go, Ruby, Custom | CLI, Console, ZIP         | Regional          | Auto (to 1000s)     | HTTP, S3, SNS, SQS, CloudWatch | Stateless, deep AWS integration          | $0.20/1M reqs + $0.0000167/GB-s |
| **AWS Fargate**           | None (ECS/EKS free tier applies) | None (task-defined)      | Any (Docker)                  | ECS/EKS, Console          | Regional          | Manual/Auto         | ECS/EKS events                | Serverless containers, flexible compute   | $0.04048/vCPU-hr, $0.004445/GB-hr |
| **AWS App Runner**        | 100 GB-hrs/mo (1st yr)        | None (app-defined)        | Node.js, Python, Custom       | Console, CLI, Git         | Regional          | Auto                | HTTP                          | Managed app hosting, CI/CD               | $0.007/hr + $0.001/GB-out |
| **GCP Cloud Functions**   | 2M invocations/mo             | 9 min                     | Node.js, Python, Go, Java, Custom | CLI, Console, ZIP         | Regional          | Auto                | HTTP, Pub/Sub, Storage        | Event-driven, lightweight                | $0.40/1M + compute (~$0.0000025/GB-s) |
| **GCP Cloud Run**         | 2M reqs/mo, 360k GB-s         | 60 min                    | Any (Docker)                  | CLI, Console, Git         | Regional          | Auto (to 1000s)     | HTTP, Events                  | Stateless containers, Knative-based       | $0.000024/GB-s + $0.40/1M reqs |
| **GCP App Engine (GAE)**  | 28 instance-hrs/day (standard) | None (app-defined)       | Node.js, Python, Java, PHP, Go, Ruby | CLI, Console, Git         | Regional          | Auto                | HTTP                          | Managed apps, versioning                 | $0.05-$0.30/instance-hr (tiered) |
| **GCP GKE (Serverless)**  | $74.40/mo credit             | None (pod-defined)        | Any (Docker)                  | GKE, CLI, Console         | Regional          | Auto (KEDA)         | Kubernetes events             | Kubernetes with auto-scaling             | $0.10/hr/node + compute |
| **Azure Functions**       | 1M executions/mo              | 10 min (Consumption)      | C#, JS, Python, Java, PowerShell, Custom | CLI, VS Code, Git         | Regional          | Auto                | HTTP, Blob, Queue, Timer      | Durable Functions, Azure integration     | $0.20/1M + $0.000016/GB-s |
| **Azure Container Apps**  | 180k vCPU-s, 360k GB-s/mo     | None (task-defined)       | Any (Docker)                  | CLI, Console, Git         | Regional          | Auto (KEDA)         | HTTP, Events                  | Serverless containers, microservices     | $0.047/vCPU-hr + $0.005/GB-hr |
| **Vercel Serverless Functions** | 100 GB-hrs/mo (Hobby)    | 10s (Hobby), 60s (Pro)    | Node.js, Python, Go, Ruby     | Git, CLI                  | Edge-like (CDN)   | Auto                | HTTP                          | Frontend focus, SSR, API routes          | $20/mo (Pro) + usage |
| **Vercel Hosting**        | Free (Hobby: 1 user, 100GB BW) | N/A (static hosting)     | N/A (static + functions)      | Git, CLI                  | Edge (CDN)        | Auto (static)       | N/A                           | Static hosting, free SSL, domains        | $20/mo (Pro) + $0.50/GB BW |
| **Cloudflare Workers**    | 100k reqs/day (~3M/mo)        | 100ms CPU (free), 30s (paid) | JS, TS, WASM (Rust, C)      | CLI, Git, Wrangler        | Edge (300+ PoPs)  | Auto                | HTTP                          | Edge compute, KV storage, low latency    | $5/10M reqs + $0.30/M extra |
| **Bunny.net Magic Containers** | None                     | None (Docker-defined)     | Any (Docker)                  | Console, Git, Docker      | 40+ regions       | Auto (AI-driven)    | HTTP, TCP/UDP                 | Global containers, AI provisioning       | ~$0.02/3,600 CPU-s + usage |
| **Bunny.net Edge Scripting** | None (free when idle)    | Seconds (HTTP-focused)    | JS, TS (Deno)                 | Git, Console              | Edge (119+ PoPs)  | Auto                | HTTP                          | Lightweight, CDN-integrated              | ~$0.01-$0.05/mo (1k runs, 100ms) |

### Notes on Key Columns
1. **Free Tier**: Indicates if there’s a no-cost entry point and its limits. Vercel’s Hosting and Serverless Functions share the Hobby tier; Bunny.net lacks a traditional free tier but is free when idle for Edge Scripting.
2. **Runtime Limit**: Maximum execution time per invocation. Cloudflare’s 100ms is CPU time (wall-clock can exceed); others are total duration.
3. **Languages**: Supported runtimes or container flexibility.
4. **Deployment**: How code is pushed live (e.g., CLI, Git).
5. **Edge/Regional**: Execution location—edge (global PoPs) vs. regional (data centers).
6. **Scaling**: Auto-scaling behavior and limits.
7. **Triggers**: What initiates execution (e.g., HTTP, events).
8. **Key Features**: Unique strengths or focus.
9. **Cost Beyond Free**: Rough estimate for 1,000 daily runs at 100ms each, unless specified.

### Observations
- **Free Tier Winners**: Cloudflare (100k reqs/day), GCP Cloud Functions (2M/mo), and Vercel (100 GB-hrs) stand out. AWS Lambda and Azure Functions also cover small workloads free.
- **Runtime Flexibility**: AWS Lambda (15 min), GCP Cloud Run (60 min), and container options (Fargate, Azure Container Apps, Magic Containers) suit longer tasks. Cloudflare’s 100ms CPU limit is tightest.
- **Edge Focus**: Cloudflare (300+ PoPs), Bunny.net Edge Scripting (119+ PoPs), and Vercel (CDN-backed) prioritize low latency. Others are regional.
- **Ease**: Vercel, Bunny.net Edge Scripting, and Cloudflare excel with Git-based deploys; AWS/GCP/Azure require more config.

### Next Steps
This table sets us up to match application types to platforms. For your Aircall-to-Guesty script:
- **Lightweight, HTTP-driven**: Cloudflare Workers or Bunny.net Edge Scripting (low latency, cheap).
- **Free Tier Priority**: Cloudflare or AWS Lambda.
- **Longer Tasks**: AWS Lambda, GCP Cloud Run, or Azure Functions.

What app types are you thinking about (e.g., real-time, batch, frontend-heavy)? I can refine the fit based on that!