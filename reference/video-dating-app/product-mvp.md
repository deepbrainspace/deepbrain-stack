Let’s define the exact products your video dating app with streams will offer, outline the key user journeys, and pinpoint the Minimum Viable Product (MVP) features for the fastest market entry. This builds on our finalized Hetzner CCX63/Docker Swarm plan, targeting 5M users with a free tier, and keeps it lean for rapid launch.

---

### Exact Products Offered
Your app delivers a "Tinder of Video Dating" experience with unlimited usage, blending real-time video and AI-driven features. Here’s what you’re offering:

1. **1-on-1 Video Calls**:
   - **Description**: Private, P2P WebRTC calls for dating/matching.
   - **Value**: Instant, authentic connections vs. text-only swiping.
   - **Tech**: Rust (signaling), Mediasoup (fallback relay if P2P fails).

2. **Group Video Calls**:
   - **Description**: SFU-based calls for up to 10 users—social “speed dating” or friend hangouts.
   - **Value**: Casual, low-pressure meetups.
   - **Tech**: Mediasoup (SFU), Rust (room management).

3. **Livestreams**:
   - **Description**: 1-to-500+ viewer broadcasts—users showcase personality or host events.
   - **Value**: Standout feature for engagement, viral potential.
   - **Tech**: Mediasoup (relay), Rust (stream control).

4. **Real-Time Chat**:
   - **Description**: Text messaging during/between calls/streams.
   - **Value**: Keeps users connected, builds rapport.
   - **Tech**: Postgres (storage), Rust (WebSocket delivery).

5. **AI Matchmaking & Concierge**:
   - **Description**: Qdrant vectors for profile matching, Phi-3 for chat assistance (e.g., icebreakers).
   - **Value**: Personalized, effortless pairing.
   - **Tech**: Qdrant (vectors), Phi-3 (NLP).

6. **SSO Authentication**:
   - **Description**: Google login for seamless signup.
   - **Value**: Low friction, modern UX.
   - **Tech**: Keycloak (OAuth2).

---

### User Journeys
Here are the primary journeys for your key personas—Singles (daters) and Streamers (content creators)—mapped to products:

1. **Singles Seeking Matches**:
   - **Goal**: Find and connect with compatible dates.
   - **Journey**:
     1. Sign up via Google SSO (Keycloak).
     2. Complete profile (name, photo, prefs—stored in Postgres).
     3. Browse AI-suggested matches (Qdrant).
     4. Initiate 1-on-1 call (Rust/Mediasoup).
     5. Chat during/after call (Rust/Postgres).
     6. Join group call for casual meetups (Mediasoup SFU).
   - **Pain Points**: Slow signup, bad matches—solved by SSO and AI.

2. **Streamers Building Audience**:
   - **Goal**: Gain followers, showcase personality.
   - **Journey**:
     1. Sign up via Google SSO (Keycloak).
     2. Set up profile (Postgres).
     3. Start livestream (Mediasoup relay).
     4. Chat with viewers (Rust/Postgres).
     5. Promote via AI concierge (Phi-3 suggestions).
   - **Pain Points**: Low visibility—solved by AI promotion, stream ease.

3. **Social Explorers**:
   - **Goal**: Meet people casually, not just date.
   - **Journey**:
     1. SSO login (Keycloak).
     2. Join group call (Mediasoup SFU).
     3. Chat in real-time (Rust/Postgres).
     4. Watch livestreams (Mediasoup).
   - **Pain Points**: Awkward intros—solved by group format, chat.

---

### MVP for Fastest Market Entry
To hit the market ASAP, we’ll focus on a lean MVP that validates core value (video dating + streams) and gathers feedback, targeting 10K users/3K peak on 1 CCX13 ($14.09/month). Here’s the minimum:

#### Features
1. **SSO Signup**:
   - Google login via Keycloak—fast onboarding.
   - Profile: Name, photo, 3 prefs (e.g., age, location, interests)—Postgres.

2. **1-on-1 Video Calls**:
   - Rust (signaling), Mediasoup (relay)—basic P2P with fallback.
   - Match manually via profile list (no AI yet).

3. **Real-Time Chat**:
   - Text during calls—Rust (WebSocket), Postgres (storage).

4. **Livestreams**:
   - 1-to-50 viewers (Mediasoup relay)—cap for CCX13.
   - Start/stop stream UI.

#### Why This MVP?
- **Speed**: 2-3 months dev time—SSO + calls + chat = core dating loop; streams add viral hook.
- **Validation**: Tests if users want video-first dating and streaming—key hypotheses.
- **Cost**: $14.09 (CCX13) + $3 (OVH proxy) = ~$17/month—minimal risk.
- **Scalability**: Docker Swarm-ready—add CCX63 at 100K users.

#### Exclusions (Later Iterations)
- **Group Calls**: SFU complexity delays launch—add post-MVP.
- **AI Matchmaking**: Qdrant/Phi-3 ($150-$200) not critical for 10K—add at 500K.
- **HA**: Single CCX13, no replication—scale to 2 CCX63 at 5M.

#### Tech Stack
- **Rust**: Signaling, chat, stream control.
- **Mediasoup**: Video relay.
- **Postgres**: Profiles, chat storage.
- **Redis**: Session cache.
- **Keycloak**: SSO.
- **HAProxy**: Traffic routing.
- **Swarm**: Single-node deploy.

---

### User Journeys in MVP
1. **Singles**:
   - SSO → Profile → Browse list → Call → Chat.
2. **Streamers**:
   - SSO → Profile → Start stream → Chat with viewers.

---

### Next Steps
- **Build**: 2-3 months—Rust team codes MVP, deploys on CCX13.
- **Launch**: Seed 1K users (e.g., college campuses), measure calls/streams.
- **Iterate**: Add group calls/AI if 50%+ retention.

**Takeaway**: MVP = SSO + 1-on-1 calls + chat + streams—fast, cheap, validates core. Start here, scale to full vision. Thoughts on tweaking features?