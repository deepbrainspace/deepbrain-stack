Creating mobile and desktop apps for your customized version of Docmost (with features like OIDC SSO and others you might add) is feasible, but the ease and approach depend on your goals, technical constraints, and the tools you choose. Since Docmost is a Ruby on Rails web application with a frontend built using Stimulus.js and Tailwind CSS, it’s inherently web-based, so you’ll need to bridge it to native platforms (macOS, Windows, iPhone, Android). Below, I’ll evaluate the options, feasibility, and steps for each platform, considering your existing Docker-based setup and AGPL-3.0 licensing.

---

### Overview of Options
You have two main strategies:
1. **Web-Based Apps (Wrapped)**: Wrap the existing web app in a native shell using tools like Electron (desktop) or WebView/Capacitor (mobile). This is the easiest and fastest approach since it reuses the current codebase.
2. **Native Apps**: Build fully native apps (e.g., Swift for iOS/macOS, Kotlin for Android, C# for Windows) that connect to the Docmost backend via an API. This is harder but offers better performance and native features.

Given Docmost’s architecture, the web-based approach is likely the most practical starting point, especially since you’re already customizing it and packaging it with Docker. Let’s break it down by platform.

---

### 1. Desktop Apps (Mac/Windows)
**Tool**: Electron  
**Ease**: Easy to Moderate  
**Why**: Electron lets you package a web app (HTML, CSS, JS) into a desktop executable, leveraging the existing Docmost frontend.

#### Steps
1. **Set Up Electron**:
   - Install Node.js and npm.
   - Create a new directory (e.g., `docmost-electron`) and initialize it:
     ```bash
     npm init -y
     npm install electron --save-dev
     ```
   - Create a basic `main.js`:
     ```javascript
     const { app, BrowserWindow } = require('electron');
     function createWindow() {
       const win = new BrowserWindow({
         width: 1200,
         height: 800,
         webPreferences: { nodeIntegration: true }
       });
       win.loadURL('http://localhost:3000'); // Point to your Docmost instance
       // For production, bundle the app with a local server or static files
     }
     app.whenReady().then(createWindow);
     app.on('window-all-closed', () => { if (process.platform !== 'darwin') app.quit(); });
     ```
   - Update `package.json`:
     ```json
     "scripts": {
       "start": "electron ."
     }
     ```

2. **Integrate with Docmost**:
   - **Development**: Point Electron to your running Docmost instance (`http://localhost:3000` from your Docker setup).
   - **Production**: Either:
     - Bundle a lightweight Rails server (e.g., using `rackup`) and the Docmost app inside Electron.
     - Export the frontend as static files (if feasible) and serve them locally, connecting to a remote Docmost backend via API.

3. **Add Native Features**:
   - Use Electron APIs for menus, notifications, or file system access (e.g., saving documents locally).
   - Example: Add a menu bar in `main.js`:
     ```javascript
     const { Menu } = require('electron');
     const template = [{ label: 'File', submenu: [{ label: 'Quit', role: 'quit' }] }];
     Menu.setApplicationMenu(Menu.buildFromTemplate(template));
     ```

4. **Build for Mac/Windows**:
   - Install `electron-builder`:
     ```bash
     npm install electron-builder --save-dev
     ```
   - Add to `package.json`:
     ```json
     "build": {
       "appId": "com.yourname.docmost",
       "mac": { "target": "dmg" },
       "win": { "target": "nsis" }
     },
     "scripts": {
       "dist": "electron-builder"
     }
     ```
   - Run `npm run dist` to generate `.dmg` (Mac) and `.exe` (Windows) installers.

#### Pros
- Reuses your existing web app with minimal changes.
- Single codebase for Mac and Windows.
- Fast to prototype (days to weeks).

#### Cons
- Larger app size (~100 MB+ due to Chromium).
- Not truly native—performance and UI might feel less polished than native apps.

#### Licensing
- Electron is MIT-licensed, so no conflicts with AGPL-3.0.
- If you distribute the Electron app, the Docmost core must remain AGPL-3.0, and you’d need to provide source code (e.g., a GitHub link).

---

### 2. Mobile Apps (iPhone/Android)
**Tool**: Capacitor (with Ionic) or WebView  
**Ease**: Moderate  
**Why**: Capacitor wraps your web app into a native iOS/Android shell, reusing the Docmost frontend while allowing access to native features.

#### Steps
1. **Set Up Capacitor**:
   - Ensure your Docmost app runs locally (e.g., via Docker at `http://localhost:3000`).
   - Install Ionic CLI and Capacitor:
     ```bash
     npm install -g @ionic/cli
     npm install @capacitor/core @capacitor/cli
     ```
   - Initialize Capacitor in your Docmost root (assuming the frontend is in `public/` or served dynamically):
     ```bash
     npx cap init
     ```
     - App Name: `Docmost`
     - App ID: `com.yourname.docmost`
     - Web Dir: `public` (or adjust if serving dynamically).

2. **Add Platforms**:
   - Install iOS and Android plugins:
     ```bash
     npm install @capacitor/ios @capacitor/android
     npx cap add ios
     npx cap add android
     ```
   - Point Capacitor to your app:
     - Edit `capacitor.config.json`:
       ```json
       {
         "appId": "com.yourname.docmost",
         "appName": "Docmost",
         "webDir": "public",
         "server": { "url": "http://localhost:3000" } // Dev mode
       }
       ```

3. **Run Locally**:
   - Start your Docmost server (`docker-compose up`).
   - Sync and open:
     ```bash
     npx cap sync
     npx cap open ios    # Xcode for iPhone
     npx cap open android # Android Studio
     ```

4. **Production Build**:
   - For standalone apps, either:
     - Serve the frontend statically (precompile assets with `rake assets:precompile`) and bundle them.
     - Use a local server (e.g., a lightweight Rails instance) within the app, connecting to a remote backend.
   - Build APKs (Android) and IPA files (iOS) via Xcode/Android Studio.

5. **Native Features**:
   - Add plugins (e.g., `@capacitor/camera` for photo uploads):
     ```bash
     npm install @capacitor/camera
     ```
     - Example usage in JS:
       ```javascript
       import { Camera } from '@capacitor/camera';
       const takePhoto = async () => {
         const image = await Camera.getPhoto({ quality: 90 });
         // Upload to Docmost backend
       };
       ```

#### Pros
- Reuses your web app with minimal refactoring.
- Cross-platform (iOS/Android) with one codebase.
- Access to native APIs (camera, push notifications).

#### Cons
- WebView performance isn’t as smooth as native apps.
- UI might need tweaking for mobile (e.g., responsive design adjustments in Tailwind CSS).

#### Licensing
- Capacitor is MIT-licensed, compatible with AGPL-3.0.
- Distribution requires AGPL-3.0 source code availability for the Docmost core.

---

### 3. Fully Native Apps (Alternative)
**Ease**: Hard  
**Why**: Building native apps (Swift for iOS/macOS, Kotlin for Android, C#/.NET for Windows) requires rewriting the frontend and connecting to the Docmost Rails backend via an API.

#### Feasibility
- **API Requirement**: Docmost’s backend would need a robust REST or GraphQL API. The current codebase has some API endpoints (e.g., for document management), but you might need to expand them (e.g., `/api/v1/documents`, `/api/v1/users`) for full app functionality.
- **Effort**: Months of development per platform unless you use a cross-platform framework like Flutter or React Native.

#### Steps (Simplified)
1. Expose APIs in Rails (e.g., in `config/routes.rb` and `app/controllers/api/`).
2. Build native UIs, calling the backend (e.g., `http://your-server:3000/api/...`).
3. Package with native tools (Xcode, Android Studio, Visual Studio).

#### Recommendation
- Avoid this unless you need top-tier performance or native-only features (e.g., offline mode). The web-wrapped approach is faster and leverages your existing work.

---

### Integration with Your Docker Setup
Your current Dockerized Docmost (with OIDC SSO) fits well with these apps:
- **Development**: Point Electron/Capacitor to `http://localhost:3000` (your `docker-compose` app service).
- **Production**: 
  - Host the Dockerized backend on a server (e.g., `https://your-docmost.com`).
  - Update app configs (`loadURL` in Electron, `server.url` in Capacitor) to the live URL.
  - Ensure OIDC redirect URIs include app-specific callbacks (e.g., `docmost://oauth-callback` for mobile).

---

### How Easy Is It?
- **Desktop (Electron)**: Very easy (1-2 weeks for a basic app, longer for polish). Reuses the web UI directly.
- **Mobile (Capacitor)**: Moderate (2-4 weeks). Requires some mobile-specific tweaks (e.g., touch gestures, screen sizes).
- **Timeline**: A solo developer could prototype both in a month; a polished release might take 2-3 months.
- **Skills Needed**: Basic JS/Node for Electron, some mobile dev knowledge (Xcode/Android Studio) for Capacitor.

#### Challenges
- **Responsive Design**: Docmost’s frontend might need CSS adjustments for mobile (Tailwind makes this manageable).
- **Offline Support**: Hard with a web app—native apps would need extra logic (e.g., local storage).
- **App Store Approval**: Apple/Google might scrutinize WebView apps; ensure a decent UX.

---

### Licensing Considerations
- Your apps must comply with AGPL-3.0 for the Docmost core if distributed:
  - Include a source code link (e.g., GitHub) in the app.
  - If you add proprietary features, separate them (e.g., in Electron/Capacitor plugins) and license them differently.
- App Store distribution is possible under AGPL-3.0 as long as source code is accessible.

---

### Recommendation
- **Start with Electron for Desktop**: Quickest way to a Mac/Windows app. Test it with your Docker setup.
- **Follow with Capacitor for Mobile**: Reuse the same web base, tweak for iPhone/Android.
- **Enhance Gradually**: Add native features (e.g., notifications) as needed.

This approach minimizes effort while maximizing reuse of your customized Docmost. Want to dive deeper into one platform or a specific feature? Let me know!