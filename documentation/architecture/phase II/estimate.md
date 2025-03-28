## Phase II: Guest Dashboard - Work Breakdown & Estimate (AI-Accelerated)

**Assumptions:**
*   Utilizing a component library significantly speeds up UI construction.
*   AI coding assistants handle much of the boilerplate for React components, CF Worker setup, and SDK usage.
*   Focus is on integration, data flow, and specific logic.
*   Estimates are rough and assume familiarity with the chosen tools is gained quickly.

| Task                         | Description                                                                                                | Estimated Hours (AI-Accelerated) | Notes                                                                 |
| :--------------------------- | :--------------------------------------------------------------------------------------------------------- | :------------------------------- | :-------------------------------------------------------------------- |
| **1. Design & Planning**     | Finalize UI layout based on screenshot/libs, define data models, confirm RocketChat integration method.      | 4 - 8                            | Quick mockups, refine requirements.                                   |
| **2. Frontend Dev (React)**  | Setup Vite/React, integrate component library (Mantine/Chakra), build UI components (chat, lists, data views). | 24 - 40                          | AI writes basic components, dev focuses on layout, state, styling.    |
| **3. Backend Dev (BFF Worker)** | Setup CF Worker, add Firestore SDK, implement data fetching logic, add endpoint to call LLM worker, auth handling. | 16 - 24                          | AI helps with SDK usage, routing, basic auth checks.                  |
| **4. RocketChat Integration**| Create RocketChat app manifest, configure embedding URL, handle context passing (user/convo ID).            | 8 - 16                           | Depends on RocketChat App framework complexity.                       |
| **5. Firestore Security**    | Define and test Firestore security rules for authenticated guest read access via BFF.                      | 4 - 8                            | Critical for security.                                                |
| **6. LLM Worker Adj. (Opt.)** | If needed, modify LLM worker to handle guest-specific context or permissions.                               | 0 - 12                           | May not be needed if current flow is sufficient.                      |
| **7. Integration Testing**   | Test the end-to-end flow: RC -> React -> BFF -> Firestore/LLM Worker -> Back.                             | 8 - 16                           | Manual and potentially automated tests.                               |
| **8. Deployment**            | Setup GitHub Actions (or manual process) to deploy React app to CF Pages & BFF Worker.                     | 4 - 8                            | Relatively straightforward for static sites & workers.                |
| **9. Documentation**         | Update `architecture.md` with final details, add Mermaid diagram.                                          | 2 - 4                            | As done during the process.                                           |
| **Total Estimated Hours:**   |                                                                                                            | **70 - 136 Hours**               | **Approx. 2-4 person-weeks** depending on focus & chosen complexity. |

## Cost Considerations

*   **Cloudflare Pages/Workers:** Likely covered entirely by the generous free tiers unless traffic becomes substantial.
*   **GCP Firestore/Functions:** Minor increase in reads/invocations based on guest usage. Still likely very low cost.
*   **Qdrant/Groq:** Costs scale with AI query volume, which this feature will increase. Monitor usage.
*   **Development Time:** The primary cost is the development effort outlined above.