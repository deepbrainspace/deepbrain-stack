Since you're planning to create a modified version of Docmost with additional features, the licensing flexibilities you have depend on the existing license of the Docmost codebase and how you intend to use or distribute your version. Let’s break this down based on the information from the Docmost GitHub repository (https://github.com/docmost/docmost) and general open-source licensing principles, keeping it relevant to your goal of adding features and integrating OIDC SSO as previously discussed.

---

### Docmost’s Current Licensing
Docmost is licensed under the **GNU Affero General Public License v3.0 (AGPL-3.0)** for its core codebase, with some enterprise features (like SSO with SAML and OIDC support) available under a separate **Docmost Enterprise License**. This dual-licensing model is common in open-source projects that offer both a free community version and a paid enterprise version. Here’s what this means for you:

1. **AGPL-3.0 (Core Codebase)**:
   - **Permissions**: You can use, modify, and distribute the core Docmost code freely, including adding your own features (like the generic OIDC SSO we implemented).
   - **Obligations**: 
     - If you distribute your modified version (e.g., as a binary or hosted service), you must make your source code available under the AGPL-3.0 to anyone who interacts with it over a network (e.g., users of your self-hosted instance).
     - You must retain the original copyright notices and the AGPL-3.0 license text in your version.
   - **Copyleft**: Any derivative work (your modified version) must also be licensed under AGPL-3.0, meaning you can’t make it proprietary unless you negotiate a different license with the original authors.

2. **Docmost Enterprise License (Enterprise Features)**:
   - The enterprise features (e.g., SSO as introduced in v0.9 per the release notes) are not under AGPL-3.0 but a proprietary license requiring a paid enterprise key.
   - Since we’ve added our own OIDC SSO implementation to the core codebase (not relying on their enterprise code), our modifications fall under AGPL-3.0, not the enterprise license.
   - If you were to integrate or adapt their enterprise features, you’d need to contact `sales@docmost.com` for a license, but our custom OIDC solution avoids this.

---

### Licensing Flexibilities for Your Version
Given that your modified version builds on the AGPL-3.0 core, here are your options and flexibilities:

#### 1. Keep It Open Source Under AGPL-3.0
- **What You Can Do**:
  - Add your features (e.g., OIDC SSO, plus anything else like enhanced commenting or mobile app support).
  - Self-host it for personal or organizational use without distributing it, avoiding the need to share your source code (AGPL’s network trigger only applies if users access it remotely).
  - Distribute it (e.g., via GitHub) as an open-source project under AGPL-3.0.
- **Flexibility**:
  - You have full freedom to modify the code and add features as long as you comply with AGPL-3.0 terms.
  - You can fork the project, give it a new name (e.g., “Docmost Plus”), and build a community around it.
- **Requirements**:
  - If you offer it as a service (e.g., hosted on a server accessible to others), you must provide the source code, including your changes, under AGPL-3.0.
- **Use Case**: Ideal if you’re happy keeping it open source and want to share your improvements with others or contribute back to the community.

#### 2. Private Use Without Distribution
- **What You Can Do**:
  - Modify Docmost with your features and use it internally (e.g., self-hosted for your team or company).
  - Since you’re not distributing it or making it accessible over a network to external users, you don’t have to share your source code.
- **Flexibility**:
  - You can keep your changes private and add as many features as you like without licensing constraints.
  - No need to worry about the AGPL’s copyleft obligations unless you later decide to distribute or host it publicly.
- **Requirements**:
  - None beyond respecting the original copyright notices if you ever share snippets informally.
- **Use Case**: Perfect if your goal is a custom internal tool without public release.

#### 3. Create a Dual-Licensed Version
- **What You Can Do**:
  - Since you’re adding original features (e.g., your OIDC SSO implementation), you own the copyright to those additions.
  - You could dual-license your version: keep the AGPL-3.0 core as-is and offer your new features under a different license (e.g., MIT, Apache 2.0, or a proprietary license).
- **Flexibility**:
  - You could offer a free AGPL-3.0 version and a paid proprietary version with extra features, similar to Docmost’s model.
  - Alternatively, use a more permissive license (e.g., MIT) for your additions to attract users who want flexibility without copyleft.
- **Requirements**:
  - The original Docmost core must remain AGPL-3.0, so users of your version must still have access to that part’s source code if distributed or hosted.
  - Clearly separate your new code (e.g., in distinct files or modules) and specify its license to avoid confusion.
- **Challenges**:
  - Legal complexity increases—consulting a lawyer might be wise to ensure compliance.
  - Users might need to comply with both licenses (AGPL for the core, your license for additions), which could deter adoption.
- **Use Case**: Viable if you want to monetize your version or offer it under different terms than AGPL-3.0.

#### 4. Negotiate a Commercial License with Docmost Authors
- **What You Can Do**:
  - If you want to make your version proprietary (no source code sharing) and distribute it, contact the Docmost team (e.g., via `sales@docmost.com`) to negotiate a commercial license for the core codebase.
  - Combine this with your own features under your chosen license.
- **Flexibility**:
  - Full control over distribution—no need to release your source code.
  - You could sell your version or use it in a closed-source product.
- **Requirements**:
  - Likely involves a fee or revenue-sharing agreement with the Docmost team.
  - You’d need to clarify whether your OIDC SSO additions (which mimic their enterprise feature) conflict with their proprietary offerings.
- **Use Case**: Best if you’re building a commercial product and want to avoid open-source obligations.

#### 5. Contribute Back to Docmost
- **What You Can Do**:
  - Add your features (e.g., OIDC SSO) and submit them as pull requests to the official Docmost repo.
  - Your contributions would be licensed under AGPL-3.0 (or their enterprise license if accepted as an enterprise feature).
- **Flexibility**:
  - You’d benefit from community maintenance and updates to your features.
  - No need to manage your own fork.
- **Requirements**:
  - The Docmost team must accept your changes, and you’d assign copyright or agree to their licensing terms.
- **Use Case**: Good if you want your features in the main project and don’t mind them being AGPL-3.0.

---

### Practical Steps for Your Version
Assuming you want to add features (e.g., OIDC SSO plus a few more like nested commenting or API enhancements) and build a new Docker image, here’s how licensing impacts your workflow:

1. **Modify the Code**:
   - Add your features to the existing AGPL-3.0 codebase.
   - Keep your changes in separate files or clearly documented modules if you plan to dual-license them.

2. **Update Docker Setup**:
   - Your `Dockerfile` and `docker-compose.yml` (as outlined previously) remain AGPL-3.0 since they’re part of the build process for the core app.
   - If you add custom scripts or configs for your features, you can license those separately.

3. **Choose Your Path**:
   - **Open Source Fork**: Publish your version on GitHub under AGPL-3.0 with your features included.
   - **Private Use**: Build and deploy your Docker image internally, no licensing hassle.
   - **Dual License**: License your new features differently (e.g., MIT) and document this in a `LICENSE` file, keeping the core AGPL-3.0.
   - **Commercial**: Negotiate with Docmost for a proprietary license.

4. **Document Licensing**:
   - Add a `README` or `LICENSE` file clarifying that the core is AGPL-3.0 and your additions (if any) are under your chosen license.

---

### Additional Features and Licensing Impact
The AGPL-3.0 gives you freedom to add whatever features you want (e.g., mobile app support, advanced search, or integrations). The licensing flexibility hinges on distribution:
- **No Distribution**: Add anything, keep it private, no worries.
- **Distribution**: Your new features must be AGPL-3.0 unless you separate them legally (dual-licensing) or get a commercial deal.

For example, if you add a feature like nested commenting:
- Code it in `app/controllers` or `app/views`, build your Docker image, and it’s AGPL-3.0 if shared.
- If you want it proprietary, isolate it in a plugin or module and license it separately.

---

### Recommendation
Since you’re customizing Docmost with OIDC SSO and more, I’d suggest:
- **Start with a Private Fork**: Build your version, test it internally, and use the Docker setup we discussed. No licensing burden yet.
- **Decide Later**: If you want to share it, go AGPL-3.0 for simplicity or explore dual-licensing for flexibility. If it’s a business venture, talk to the Docmost team.

This keeps your options open while you develop. What additional features are you thinking of? I can tailor the advice further!