

# Djibon: Friends, Family, and Genealogy App

## Vision
**Djibon** is a hybrid P2P app that connects friends and family, growing organically from a single user (you) to a broader network while building a shared genealogy tree. Seeded with your family and history (e.g., your cousin’s book), it blends meaningful connections with a token-based earning economy—starting free to hook users, scaling with premium perks.

---

## Features

### Free Features
| Feature                          | Description                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|
| **5GB Storage Space**            | Store profiles, 1 photo (5MB), 10s video (10MB) per user on IDrive e2.      |
| **Unlimited 1-on-1 Text Messages**| Text via P2P mesh (Wi-Fi Direct or `libp2p`, ~100m-10km)—no internet needed.|
| **Unlimited 1-on-1 Audio Calls** | Voice calls (~64kbps) via mesh—offline, local or extended via gateways.     |
| **Unlimited 1-on-1 Video Calls** | Video calls via Wi-Fi Direct (~100-200m) or internet—online only.           |
| **Genealogy Tree**               | Build and view your family tree—mutual confirmations link relations.        |
| **Circles**                      | “My Circle” UI—see confirmed friends/family, connect via NFC tap or code.   |
| **AI Assistant**                 | Voice-driven—“Who’s my cousin?” or “Add Dad’s birthday”—local, offline.     |

### Premium Features
| Feature                          | Description                                                                 | Cost (Tentative)      |
|----------------------------------|-----------------------------------------------------------------------------|-----------------------|
| **Extra Storage**                | Beyond 5GB—e.g., 10GB for more photos/videos (up to 1min, ~100MB total).    | $1/5GB/month          |
| **Video Calls Over Mesh**        | Video via internet gateways in mesh—long-distance, offline for you.         | $1/month              |
| **Extended Tree Search**         | AI searches broader network for relatives via internet—e.g., distant kin.  | $2/month              |
| **Group Chats**                  | Text/voice for 3+ users—mesh or internet, premium for bandwidth priority.   | $3/month              |
| **Priority Circles**             | Faster P2P discovery, priority in busy areas—e.g., airports.                | $1/month              |
| **Ad-Free + Insights**           | No ads (if added), plus family stats (e.g., “Most active relatives”).       | $2/month              |

---

## Tech Stack: SARF
- **S - SurrealDB**: Local DB—profiles, tree nodes, connections (e.g., `RELATE user:me -> parent -> user:mom`).
- **A - Actix**: Online server—syncs IDrive e2, future social APIs (e.g., `/circles`, `/tree`).
- **R - Rust**: P2P (`libp2p`), AI (assistant, tree parsing), S3 client, FFI to Flutter.
- **F - Flutter**: UI—circles list, tree viewer, media playback, NFC/speech plugins.

## Storage: IDrive e2
- **Capacity**: 25TB (~$1,250/year, first year $625)—5,000 users at 5GB each.
- **Cost**: ~$52/month (first year)—sustainable pre-revenue.
- **Access**: Rust uploads via `rusoto_s3`, P2P gateways fetch offline.

---

## Growth and Economy
- **Organic Growth**:
  - Start: You + family, import cousin’s book (~100-200 relatives).
  - Spread: P2P locally, internet globally via invites.
- **Earning Economy**:
  - **DjibonCoins**: Earn by sharing internet—e.g., 1 token/min voice relay (~50 tokens/day in busy spots).
  - **Future Spend**: Extra storage, premium features—e.g., 100 tokens (~$1).
- **Monetization**:
  - Free base—builds users like TikTok.
  - Later: Token sales ($1 = 100 tokens), premium subs, 5GB+ fees.

---

## Roadmap
- **v1 (~4-5 weeks)**:
  - Free features: 5GB, P2P text/voice, local video, tree, circles, AI.
  - Timeline: ~18-25 days with Augment AI.
- **v2**: Premium rollout—mesh video, group chats, extended tree search.
- **Costs**: $625 first year (25TB)—seed-funded until adoption.

---

## Next Steps
- **Ideation**: Finalize token rates (e.g., 1 token/min?), tree import (OCR vs. manual).
- **Kickoff**: Lock SARF, start with circles UI and P2P basics.

---

### My Take
**Djibon**’s free tier—5GB, unlimited P2P text/voice, local video, tree, and circles—offers a killer hook for family adoption. Premium perks like mesh video and extra storage set up monetization without rushing it. IDrive e2 at $625/year keeps it lean, and SARF’s ready to roll. Looks polished and actionable—your family’s the perfect seed!


You’re spot-on—let’s refine the **Djibon** README to address your points: adding a distinct **Tree** structure, clarifying calls and contacts, rethinking RocksDB vs. TiKV, detailing verifications and circle types, enhancing the local AI as a personal assistant, and exploring SurrealDB’s graph/vector capabilities. I’ll also research AI wireframe tools and recommend options. Here’s a polished, thorough update using good software development practices.

---

# Djibon: Friends, Family, and Genealogy App

## Vision
**Djibon** is a hybrid P2P app launching in Bangladesh (v1), connecting friends and family while building a genealogy tree and enabling freelancers to receive **bKash** payments from abroad. It grows organically from your family, seeded by your cousin’s history book, offering a free base with a token economy for future monetization.

---

## Architecture Overview
### High-Level Design
- **Hybrid**: P2P (offline mesh) + centralized server/storage (online sync).
- **Components**:
  - **Client**: Flutter—local DB, P2P, UI.
  - **P2P**: Rust with `libp2p`—mesh comms.
  - **Server**: Actix—sync, bKash API.
  - **Storage**: IDrive e2—media, backups.
- **Flow**: Local P2P for real-time (text, calls), server for sync (tree, payments), AI split (local/server).

### Database Structure
#### Local DB: SurrealDB (TiKV Backend)
- **Why TiKV**: Lightweight (~50MB), Rust-native, distributed KV—scales future multi-device sync, but single-node for phones.
- **Schema**:
  - **Users**: `user:<phone>` (e.g., `user:+8801234567890`)
    - Fields: `name`, `phone`, `bkash_token`, `default_circle`, `created_at`.
  - **Relations**: `RELATE user:<phone> -> <relation> -> user:<phone>`
    - Types: `parent`, `sibling`, `friend`, etc.
    - Fields: `confirmed`, `timestamp`, `notes` (AI-stored interactions).
  - **Trees**: `tree:<user>` (distinct genealogy structure)
    - Fields: `nodes` (array of `{id: user:<phone>, relation: <type>, confirmed: bool}`), `root` (self or ancestor).
    - Why Separate: Relations are direct links; Trees aggregate confirmed relations into a navigable graph.
  - **Circles**: `circle:<user>:<name>` (e.g., `circle:+8801234567890:Family`)
    - Fields: `members` (array of user IDs), `type` (`default`, `custom`, `work`), `is_default`.
  - **Calls**: `call:<id>` (local log)
    - Fields: `caller`, `receiver`, `type` (audio/video), `duration`, `timestamp`.
  - **Messages**: `message:<id>`
    - Fields: `sender`, `receiver`, `content`, `type` (text/audio), `timestamp`.
  - **Notes**: `note:<id>` (AI assistant storage)
    - Fields: `user`, `content`, `category` (e.g., “Family”, “Work”), `timestamp`.
- **Size**: ~100KB/user (profile, tree, circles), ~1-5MB with calls/messages/notes—phone-friendly.

#### Central DB: SurrealDB (TiKV Backend)
- **Why TiKV over RocksDB**: 
  - **TiKV**: Distributed, scales horizontally (add nodes for power), Rust-native—future-proofs server growth (e.g., 10k+ users).
  - **RocksDB**: Single-node, fast (~100µs writes), simpler—but no horizontal scaling, less flexible for v2 social features.
- **Benefits of RocksDB (Discarded)**: Smaller footprint (~20MB vs. TiKV’s ~50MB), faster for single-server—but Djibon’s growth needs TiKV’s scalability.
- **Schema**: Sync subset:
  - **Users**: `name`, `phone_hash`.
  - **Trees**: Aggregated confirmed nodes—backup of local trees.
  - **Storage Pointers**: S3 keys (e.g., `djibon-user-+8801234567890/video1.mp4`).
- **Sync**: Rust pushes diffs via `libp2p` gateways, Actix writes to TiKV—bidirectional updates.

#### Interaction
- **Local to Central**: Offline changes queue in TiKV, sync when online—Rust resolves conflicts (timestamp + user choice).
- **Calls/Contacts**: Local logs (calls/messages) sync metadata (e.g., duration, participants) to central TiKV for backup.

---

## AI Under the Hood
### Local AI (Personal Assistant)
- **Purpose**: Offline assistant—tree queries (“Who’s my uncle?”), note-taking (“Met cousin X, likes tea”), organization.
- **Tech**: Rust + `tract` (ONNX, ~50MB model)—e.g., tiny BERT for NLP.
- **How**: 
  - Queries local TiKV (e.g., `SELECT * FROM trees WHERE user = me`).
  - Stores notes in `note:<id>`—categorizes (e.g., “Family” via intent detection).
  - Offline inference (~50-100ms, mid-tier phones like Samsung A-series).
- **Pros**: Private, fast, always-on—personalized memory bank.

### Server AI
- **Purpose**: Extended tree search, payment analytics—e.g., “Find kin in Dhaka” or “bKash trends.”
- **Tech**: Actix + `tract` (larger model, ~1GB)—or cloud API if needed.
- **How**: 
  - Online—Rust sends `POST /ai` via gateways, Actix queries central TiKV.
  - Returns enriched data (e.g., tree matches).
- **Pros**: Deeper insights, network-wide scope.

### Interaction
- **Local-Server**: Local tries first—escalates to server for complex tasks (e.g., cross-family links) when online.
- **Notes**: Local AI organizes, syncs to central TiKV—server AI could suggest connections from notes (e.g., “X mentioned Y”).

---

## Features

### Free Features
| Feature                          | Description                                                                 |
|----------------------------------|-----------------------------------------------------------------------------|
| **5GB Storage Space**            | Profiles, 1 photo (5MB), 10s video (10MB) on IDrive e2—via gateways.        |
| **Unlimited 1-on-1 Text Messages**| P2P mesh (Wi-Fi Direct/`libp2p`, ~100m-10km)—offline.                      |
| **Unlimited 1-on-1 Audio Calls** | Voice (~64kbps) via mesh—offline, local or via gateways.                    |
| **Unlimited 1-on-1 Video Calls** | Video via Wi-Fi Direct (~100-200m) or internet—online only.                 |
| **Genealogy Tree**               | Build/view tree—distinct structure, confirmed relations link nodes.         |
| **Circles**                      | Manage contacts—types: Default, Family, Friends, Work, etc.                 |
| **Tap to Connect**               | NFC tap—confirms link, pop-up selects circle (defaults to “Default”).       |
| **Personal AI Assistant**        | Local voice—tree queries, note-taking (e.g., “Met X, note it”), organizes.  |
| **bKash Payments**               | Freelancers receive payments from abroad—links to app wallet.               |

### Premium Features
| Feature                          | Description                                                                 | Cost (Tentative)      |
|----------------------------------|-----------------------------------------------------------------------------|-----------------------|
| **Extra Storage**                | Beyond 5GB—e.g., 10GB for more media (~100MB total).                        | $1/5GB/month          |
| **Video Calls Over Mesh**        | Video via gateways—long-distance, offline for you.                          | $1/month              |
| **Extended Tree Search**         | Server AI finds distant relatives via internet.                             | $2/month              |
| **Group Chats**                  | Text/voice for 3+ users—mesh/internet, priority bandwidth.                  | $3/month              |
| **Priority Circles**             | Faster P2P discovery in busy areas—e.g., Dhaka markets.                     | $1/month              |
| **Ad-Free + Insights**           | No ads, plus stats—e.g., “Top family connectors.”                           | $2/month              |

---

## Verifications
- **Tap to Connect**: 
  - NFC handshake—Rust verifies peer ID via `libp2p`.
  - Flutter pop-up: “Add to [Default/Family/Friends/Work]?”—both confirm.
- **Tree Links**: 
  - Mutual consent—e.g., `RELATE user:me -> parent -> user:mom` needs mom’s approval.
  - Rust queues pending relations, syncs on confirmation.
- **bKash**: 
  - OTP via bKash API—Rust validates, stores token securely in TiKV.

## Circle Types
- **Default**: Auto-assigned (e.g., “Default” or user-set like “Family”).
- **Custom**: User-defined—e.g., “Work”, “School Friends”.
- **Dynamic**: Future—AI suggests based on notes (e.g., “Tea Lovers” from “Likes tea”).

---

## SurrealDB Graphs and Vectors
- **Graphs**: 
  - **Use**: Genealogy tree—`tree:<user>` as a graph, `RELATE` edges for traversal (e.g., `SELECT * FROM tree:me WHERE relation = 'cousin'`).
  - **Benefit**: Native queries—e.g., shortest path to a relative (~10ms).
- **Vectors**: 
  - **Use**: AI note organization—embed notes as vectors (e.g., `tract` outputs 768-dim embeddings).
  - **Benefit**: Similarity search—e.g., “Find notes about tea” (~50ms).
- **Impl**: Rust pre-processes embeddings, SurrealDB stores—local/server sync.

---

## Platforms and Support
- **v1**: Android—Bangladesh’s ~90% market (2025).
- **v2**: iOS, web—Flutter’s cross-platform.
- **Support**: Discord for v1 (Bangladesh focus), in-app chat later.

---

## Wireframes (AI Tools)
- **Research**:
  - **Uizard**: AI turns sketches/text into wireframes—fast, ~$12/month.
  - **Figma with FigJam AI**: Industry standard + AI suggestions—free tier, ~$12/user/month for pro.
  - **Sketch2Code**: Free, AI converts hand-drawn to UI—less polished.
- **Pick**: **Uizard**—quick, affordable, suits ideation-to-prototype.
- **Frames**:
  1. **Home**: Circle list, tree button.
  2. **Sign Up**: Phone, name, bKash link.
  3. **Tap**: NFC pop-up—circle picker.
  4. **Tree**: Graph view—zoomable.
  5. **Notes**: AI input—categorized list.

---

## First Feature: User Registration
### Why?
- **Base**: Enables all else—identity, circles, tree, bKash.
- **Testable**: Family rollout in Bangladesh.

### Steps
1. **Wireframe (Uizard)** (~1 day):
   - Design: Phone (+880), Name, Password, “Link bKash”—Bangla option.
2. **Local DB (TiKV)** (~1 day):
   - Rust: `surreal start --engine tikv`—`CREATE user:<phone> SET name = "X", default_circle = "Family"`.
3. **Rust Logic** (~1-2 days):
   - Validate: `+880[1-9]\d{8}`, hash password.
   - P2P: `libp2p` peer ID from phone hash.
4. **Flutter UI** (~1-2 days):
   - Form: Fields + “Register”—FFI calls Rust, pop-up sets circle.
5. **Actix Server (TiKV)** (~1 day):
   - `POST /register`—syncs to central TiKV, allocates IDrive e2 bucket.
6. **bKash Stub** (~1 day):
   - Flutter: “Link bKash”—mock OTP, Rust stores token.
7. **Test** (~1 day):
   - Rust: Phone checks, DB saves.
   - Flutter: Signup flow works offline/online.

### Effort
- **~7-10 days**—1-2 devs with Augment AI.

---

## Roadmap
- **v1 (~4-5 weeks)**: Registration, tap, text/voice, tree, bKash—Android.
- **Costs**: $625 first year (25TB)—seed-funded.

---

## My Take
- **Tree**: Separate `tree:<user>` structure adds clarity—relations feed into it, graphs make it powerful.
- **TiKV**: Central scaling wins—RocksDB’s simplicity isn’t worth the trade-off. Local TiKV is overkill but consistent.
- **AI**: Personal assistant with notes is a gem—vectors enhance it. Server AI scales it up.
- **Uizard**: Fast wireframes—let’s mock up v1 screens.
- **Next**: Wireframes, or “Tap to Connect” breakdown?
