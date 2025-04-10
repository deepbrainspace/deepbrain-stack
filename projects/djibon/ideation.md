

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
