## Phase II Features

### Guest Dashboard

**Purpose:** Provide a dedicated, modern chat interface (similar to screenshot provided) for guests/end-users, integrated directly within RocketChat as a custom app. This dashboard will display relevant information aggregated from external systems (Aircall, Guesty via Firestore) and allow AI-powered querying of business data.

**Technology:**
- **Frontend:** React (using Vite for build tooling) with a modern component library (e.g., Mantine UI, Chakra UI) for rapid UI development, styled potentially with Tailwind CSS.
- **Backend (BFF):** A dedicated Cloudflare Worker acting as a Backend-for-Frontend. This handles authentication context, orchestrates data fetching from Firestore, and proxies AI queries to the LLM Query Worker.
- **Integration:** Deployed as a RocketChat App (likely using the embedded webview/iframe capability, passing user context).
- **Data Source:** Reads processed data directly from GCP Firestore.
- **AI Querying:** Leverages the existing LLM Query Process Flow via the LLM Query Cloudflare Worker (which interacts with Qdrant and Groq).

**Deployment:**
- The React frontend can be deployed as a static site using Cloudflare Pages (leveraging generous free tier).
- The BFF logic resides in a new Cloudflare Worker.

**Impact on Existing Components:**
- **RocketChat:** Requires configuration to add the custom app, potentially defining how it's displayed (e.g., a tab).
- **GCP Firestore:** Security rules need adjustment to allow authenticated read access from the Guest Dashboard BFF worker based on guest context.
- **Cloudflare Workers:** Addition of a new BFF worker. The LLM Query Worker might need minor adjustments if specific context/permissions handling for guests is required.
- **Keycloak:** May need configuration if used for guest authentication/role mapping within RocketChat.