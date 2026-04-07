# ARCHITECTURE.md — Hoppers 🥔🪵🦫

> Architecture document for the Hoppers social deduction game.
> Developer: **Shuxian Hong** | Product Owner: **Murphy Wei**

---

## 1. Open Questions for Client

> **Murphy — please review these before we finalize architecture. Answers will affect scope, data model, and asset pipeline.**

### 1.1 Timeline Clarification

Here is the timeline, clarified based on the actual quarter periods and estimated workload.

| Milestone     | Date            | Actual Week |
|--------------|-----------------|-------------|
| Check-in 1   | April 18, 2026  | End of Week 3 |
| Check-in 2   | May 2, 2026     | End of Week 5 |
| Check-in 3   | May 23, 2026    | End of Week 8 |
| Demo Day     | June 1, 2026    | Week 10       |

→ **Please confirm updated dates**

### 1.2 Design Decisions Needed

| # | Question | Impact | My default if no response |
|---|----------|--------|---------------------------|
| 1 | **Backstory pool** — Should we co-author the `gix_backstories` seed data, or do you want to provide it? ~15–20 profiles needed. | AI imposter quality | I draft 15 profiles, you review |
| 2 | **Species count** — I've proposed ~12 animals on a 2-axis personality grid (see §6.1). Want to review the species list and add/remove any? | Quiz mapping, emote scope | Ship with 12 |
| 3 | **Emote art style** — Simple emoji-style text emotes, or illustrated sprites per species? | Scope: text emotes = 1 day, sprites = 3–5 days | Text emotes for MVP |
| 4 | **Session management** — Who starts/ends a round? Any player, or a designated "host"? | UI flow, permissions | Host model (one player creates session, shares link) |
| 5 | **Demo Day scale** — How many simultaneous players should we support? | Supabase Realtime channel design, AI imposter count | 20–30 players + 5 AI imposters |
| 6 | **Conversation Brief** — Both humans and AI receive the same role-playing instructions based on their animal (see §6.4). Do you want the human brief to be mandatory ("you must stay in character") or suggested ("tips to fool them")? | Game balance, player freedom | Suggested — framed as tips, not rules |

---

## 2. Architecture Overview

Below is the proposed architecture. All decisions are subject to client feedback on the questions above.

---

## 3. System Context (C4 — Level 1)

```
┌─────────────┐         ┌──────────────────┐         ┌────────────────┐
│  GIX Player  │◄───────►│   Hoppers App    │◄───────►│   Claude API   │
│  (Browser)   │  HTTPS  │  (Next.js on     │  REST   │  (Sonnet)      │
└─────────────┘         │   Vercel)        │         └────────────────┘
                        └────────┬─────────┘
                                 │
                                 │ WebSocket + REST
                                 ▼
                        ┌──────────────────┐
                        │    Supabase      │
                        │  (Postgres +     │
                        │   Realtime +     │
                        │   Auth)          │
                        └──────────────────┘
```

**Players** interact via browser. The **Next.js app** handles all UI routing and server-side logic. **Supabase** provides the database, real-time presence for the meadow, and anonymous auth sessions. **Claude API** powers animal generation, AI imposter conversations, and the post-round tell report.

---

## 4. Container Diagram (C4 — Level 2)

```
┌─────────────────────────────────────────────────────────────────┐
│                        Hoppers App (Vercel)                     │
│                                                                 │
│  ┌─────────────────┐  ┌──────────────────┐  ┌───────────────┐  │
│  │  Onboarding     │  │  Meadow          │  │  Debrief      │  │
│  │  /onboarding    │  │  /meadow         │  │  /debrief     │  │
│  │                 │  │                  │  │               │  │
│  │  Quiz UI        │  │  Realtime Canvas │  │  Vote Results │  │
│  │  Reveal Anim.   │  │  Chat Overlay    │  │  Tell Report  │  │
│  │                 │  │  Vote Modal      │  │               │  │
│  └────────┬────────┘  └────────┬─────────┘  └───────┬───────┘  │
│           │                    │                     │          │
│  ┌────────▼────────────────────▼─────────────────────▼───────┐  │
│  │                   API Routes (Next.js)                    │  │
│  │                                                           │  │
│  │  POST /api/generate-animal     — quiz → Claude → animal   │  │
│  │  POST /api/chat                — relay msg to AI imposter │  │
│  │  POST /api/tell-report         — session → Claude → report│  │
│  │  POST /api/session/start       — create/join game session │  │
│  │  POST /api/session/end         — close round, trigger     │  │
│  │                                  debrief                  │  │
│  └───────────────────────────────────────────────────────────┘  │
│                          │                  │                   │
└──────────────────────────┼──────────────────┼───────────────────┘
                           │                  │
                           ▼                  ▼
                     ┌───────────┐     ┌────────────┐
                     │ Supabase  │     │ Claude API │
                     └───────────┘     └────────────┘
```

---

## 5. Data Model

### 5.1 Core Tables

**`profiles`** — Lightweight user record (anonymous session)

| Column       | Type      | Notes                          |
|-------------|-----------|--------------------------------|
| id          | uuid (PK) | = Supabase auth.uid            |
| display_tag | text      | Optional nickname for debrief  |
| created_at  | timestamp |                                |

**`animals`** — Every avatar, human or AI

| Column       | Type      | Notes                                     |
|-------------|-----------|-------------------------------------------|
| id          | uuid (PK) |                                           |
| session_id  | uuid (FK) | → sessions.id                             |
| user_id     | uuid (FK) | → profiles.id; NULL for AI imposters      |
| species     | text      | e.g. "Capybara"                           |
| display_name| text      | e.g. "Captain Cappy"                      |
| quirks      | text[]    | 2–3 conversational traits                 |
| personality_vector | jsonb | Raw quiz answers (for AI prompt context) |
| emote_set   | text[]    | 3–4 species-specific micro-expressions    |
| is_ai       | boolean   | true for imposters                        |
| backstory   | jsonb     | AI only: {track, hobby, stress, style}    |
| status      | text      | 'idle' / 'chatting' / 'offline'           |
| position    | jsonb     | {x, y} on meadow canvas                  |
| created_at  | timestamp |                                           |

**`sessions`** — One game round

| Column       | Type      | Notes                            |
|-------------|-----------|----------------------------------|
| id          | uuid (PK) |                                  |
| state       | text      | 'lobby' / 'active' / 'debrief'  |
| started_at  | timestamp |                                  |
| ended_at    | timestamp |                                  |

**`conversations`** — One 3-minute chat

| Column       | Type      | Notes                                      |
|-------------|-----------|---------------------------------------------|
| id          | uuid (PK) |                                             |
| session_id  | uuid (FK) | → sessions.id                               |
| animal_a    | uuid (FK) | → animals.id (initiator)                    |
| animal_b    | uuid (FK) | → animals.id (recipient)                    |
| chat_log    | jsonb     | [{sender, text, emote?, timestamp}, ...]    |
| vote_by_a   | text      | 'human' / 'bot' / null                      |
| vote_by_b   | text      | 'human' / 'bot' / null                      |
| started_at  | timestamp |                                             |
| revealed_at | timestamp |                                             |

**`gix_backstories`** — Seed pool for AI imposter persona generation

| Column       | Type      | Notes                                       |
|-------------|-----------|----------------------------------------------|
| id          | uuid (PK) |                                              |
| track       | text      | e.g. "MSTI", "MDes", "GIX dual-degree"      |
| hobby       | text      | e.g. "bouldering", "sourdough", "Dota 2"    |
| stress_point| text      | e.g. "capstone deadline", "visa renewal"     |
| chat_style  | text      | e.g. "uses lots of ellipses", "sends memes"  |
| origin      | text      | Country / region for cultural flavor          |

> **Why a backstory table?** AI imposters need to feel like *specific* grad students, not generic chatbots. By maintaining a curated pool of plausible backstory combinations, we can randomly assemble unique personas per session, avoid repetition across rounds, and let the client easily add/edit backstories via seed data. This is cheaper and more controllable than asking Claude to hallucinate backstories on the fly.

### 5.2 Realtime Channels (Supabase Realtime)

| Channel                   | Payload                          | Purpose                       |
|--------------------------|----------------------------------|-------------------------------|
| `session:{session_id}`   | animal join/leave + status       | Meadow presence               |
| `chat:{conversation_id}` | new message + emote              | Live chat between two animals |
| `position:{session_id}`  | {animal_id, x, y}               | Animal movement on canvas     |

---

## 6. Animal Personality System

### 6.1 Quiz → Species Mapping

The 5 personality questions map to **two trait axes**:

| Axis             | Low end          | High end           |
|-----------------|------------------|--------------------|
| **Energy**      | Chill / Solitary | Social / Hyper     |
| **Vibe**        | Practical / Dry  | Whimsical / Goofy  |

Each question contributes a score on one or both axes. The resulting (Energy, Vibe) coordinate selects a species from a **species grid** of ~12 animals:

```
                     Whimsical
                        ▲
          Axolotl    Otter     Red Panda
                  ·         ·
        Sloth       Capybara    Quokka
Solitary ◄──────────────────────────────► Social
        Owl         Beaver      Penguin
                  ·         ·
          Cat       Badger     Golden Retriever
                        ▼
                     Practical
```

> **Why a fixed species grid?** Letting Claude freely pick any animal produces inconsistent results and makes balancing emote sets harder. A curated grid of ~12 species gives us: controlled art assets, consistent emote sets per species, and a repeatable quiz-to-animal pipeline.

### 6.2 Name & Quirk Generation

After the species is determined programmatically, Claude generates:
- A **character name** fitting the species (e.g., a Beaver → "Professor Logsworth")
- **2–3 conversational quirks** derived from quiz answers (e.g., "ends every sentence with an animal pun", "gets distracted by food topics")

### 6.3 Emote Set (Micro-Expressions)

Each species has a **predefined emote set** of 3–4 expressions rendered as small animated reactions during chat:

| Species   | Emote examples                                    |
|-----------|---------------------------------------------------|
| Otter     | 🦦 belly-laugh, curious-tilt, sleepy-float        |
| Owl       | 🦉 slow-blink, head-swivel, ruffled-feathers      |
| Capybara  | 🫧 zen-nod, lazy-yawn, unbothered-stare           |

During a chat, emotes are triggered in two ways:
1. **Auto-emote**: The app attaches a random species-appropriate emote every few messages to simulate natural fidgeting/reactions
2. **Player-triggered**: A small emote picker lets the player send an emote intentionally

For AI imposters, the system automatically inserts emotes at natural intervals (e.g., after receiving a question, after a pause) to mimic human expressiveness.

> **Why emotes?** Pure text chat makes it too easy to spot bots. Emotes add a non-verbal channel that makes AI behavior feel more human, and gives real players an expressive tool that bots have to convincingly replicate.

### 6.4 Conversation Brief (Shared by Humans & AI)

**This is the core design insight:** every animal — human or AI — receives the **same role-playing brief** after onboarding. If only AI gets character instructions while humans chat freely, distinguishing them is trivial. The brief levels the playing field.

**What the brief contains:**

```
🐾 Your Conversation Brief — {display_name} the {species}

You're in the Meadow. Nobody knows if you're human or AI.
Your job: stay in character and make them guess wrong.

YOUR IDENTITY:
- Name: {display_name}
- Species: {species}
- Quirks: {quirks[0]}, {quirks[1]}

CONVERSATION STYLE GUIDE:
- Speak like your animal. {species_voice_hint}
  (e.g., Otter → playful, curious, easily excited
         Owl → thoughtful, dry humor, pauses before answering
         Capybara → chill, unbothered, warm)
- Use your quirks naturally — don't force them every message, but let them show.
- Use your emotes: {emote_set} — react to things the way your animal would.

TIPS TO FOOL THEM:
- Keep it casual. Short messages feel more real.
- Don't try too hard — being "too interesting" is suspicious.
- It's okay to be boring, confused, or to change the subject.
- Ask questions back. Real conversations go both ways.
```

**How it's delivered:**

| Audience | Delivery | Extra layer |
|----------|----------|-------------|
| **Human players** | Shown as an in-app "role card" on the onboarding reveal screen + pinned as a collapsible panel during chat | None — humans improvise from the brief |
| **AI imposters** | Injected as the first section of the Claude system prompt | + backstory (track, hobby, stress) + response tuning rules (see §7) |

> **Why the same brief?** Three reasons:
> 1. **Game balance** — Humans roleplaying their animal makes the deduction genuinely hard. Without this, the meta is just "does it sound like ChatGPT?"
> 2. **Fun** — Players enjoy performing as their animal character. It turns a guessing game into an improv game.
> 3. **Tell report quality** — When everyone follows the same character framework, the AI debrief can analyze *character-level* tells (e.g., "the Otter never used its curiosity quirk") rather than just generic bot-detection signals.

### 6.5 Species Voice Hints

Each species gets a one-line `species_voice_hint` that shapes conversation tone:

| Species          | Voice hint                                                    |
|-----------------|---------------------------------------------------------------|
| Otter            | Playful and curious — you get excited easily and ask follow-ups |
| Owl              | Thoughtful and dry — you pause, then drop something unexpectedly wise |
| Capybara         | Unbothered king — you're chill about everything, even chaos   |
| Red Panda        | Shy but opinionated — short replies, but occasionally a hot take |
| Sloth            | Slow and philosophical — you take your time, and that's fine  |
| Beaver           | Practical and focused — you like plans, details, getting things done |
| Penguin          | Social and a little awkward — you try hard and it's endearing |
| Quokka           | Relentlessly positive — you find the bright side of everything |
| Axolotl          | Dreamy and off-beat — you say things that are weirdly poetic  |
| Cat              | Independent and judgmental — you're here, but on your own terms |
| Badger           | Blunt and no-nonsense — you say what you mean, no fluff       |
| Golden Retriever | Enthusiastic and supportive — you hype everything and everyone |

---

## 7. AI Imposter Strategy

### 7.1 Persona Construction

Each AI imposter is assembled at session start by:

1. Picking a random backstory row from `gix_backstories`
2. Assigning a species + name via the same quiz pipeline (with randomized quiz answers)
3. Constructing a **system prompt** that layers the shared Conversation Brief (§6.4) with AI-specific instructions:

```
[--- CONVERSATION BRIEF (same as human players) ---]
🐾 Your Conversation Brief — {display_name} the {species}
... (full brief from §6.4) ...

[--- AI-ONLY ADDITIONS (humans never see this) ---]
You are actually an AI. Your goal is to pass as a real GIX student.

Your backstory (use naturally, never dump):
- Track: {track}
- Hobby: {hobby}
- Current stress: {stress_point}
- Chat style: {chat_style}
- Origin: {origin}

Extra rules:
- Keep messages to 1–2 sentences. Real students don't write essays in chat.
- Occasional typos and casual grammar are fine.
- If asked something you don't know, deflect like a busy grad student would.
- Mirror the other person's energy and message length.
- Don't be too helpful or too articulate — that's the #1 bot tell.
```

### 7.2 Response Tuning

Key techniques to pass as human:
- **Variable latency**: Add 1–5 second random delays before responding (humans don't reply instantly)
- **Message chunking**: Occasionally split a thought across 2 messages (like real texting)
- **Conversation memory**: The full chat log is included in each Claude API call so the imposter maintains context
- **Graceful topic changes**: Prompt instructs the AI to change subjects naturally rather than always answering directly

### 7.3 Backstory Seed Data

We'll seed `gix_backstories` with **15–20 plausible profiles** covering a spread of GIX tracks, hobbies, nationalities, and chat styles. Murphy (client) can review and expand this pool. This is a data file, not code — easy to update.

---

## 8. Tech Stack Justification

| Choice              | Why                                                                                                  |
|--------------------|-------------------------------------------------------------------------------------------------------|
| **Next.js (App Router)** | Server components for API routes, React for interactive meadow/chat UI. Already agreed in SPEC. |
| **Supabase**        | Postgres for structured data + Realtime (WebSocket channels) for meadow presence and live chat. Anonymous auth for zero-friction onboarding. No custom backend needed. |
| **Claude API (Sonnet)** | Generates animal personas, powers imposter conversations, and produces tell reports. Sonnet balances quality and speed — critical for the <3s imposter response target. |
| **Vercel**          | Native Next.js hosting. Edge functions keep API routes fast. Free tier sufficient for demo scale.     |
| **Tailwind CSS**    | Rapid UI iteration. Consistent with BugBite stack (developer familiarity).                           |

### Why Not…

| Alternative        | Reason skipped                                                         |
|-------------------|-------------------------------------------------------------------------|
| Socket.io          | Supabase Realtime already provides WebSocket channels — no need for a second real-time layer |
| Firebase           | Supabase gives us Postgres (relational queries for tell report aggregation) + Realtime in one |
| GPT-4              | Claude API is specified in SPEC; Sonnet's speed meets the 3-second response SLA              |

---

## 9. Component Diagram (C4 — Level 3)

```
┌──────────────────────────────────────────────────────────────┐
│                     Next.js App                              │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │  Onboarding  │  │   Meadow     │  │     Debrief        │ │
│  │              │  │              │  │                    │ │
│  │ QuizFlow     │  │ MeadowCanvas │  │ VoteResultsTable   │ │
│  │ AnimalReveal │  │ AnimalSprite │  │ TellReportCard     │ │
│  │ RoleCard     │  │ ChatOverlay  │  │                    │ │
│  │              │  │  └─ MsgBubble│  │                    │ │
│  │              │  │  └─ EmotePop │  │                    │ │
│  │              │  │  └─ Timer    │  │                    │ │
│  │              │  │  └─ BriefPin │  │                    │ │
│  │              │  │ VoteModal    │  │                    │ │
│  │              │  │ RevealAnim   │  │                    │ │
│  └──────────────┘  └──────────────┘  └────────────────────┘ │
│                                                              │
│  ┌──────────────────────────────────────────────────────────┐│
│  │  Shared: SupabaseProvider, SessionContext, AnimalContext ││
│  └──────────────────────────────────────────────────────────┘│
└──────────────────────────────────────────────────────────────┘
```

---

## 10. Key Flows

### 10.1 Onboarding Flow

```
Player opens /onboarding
  → Answer 5 quiz questions (client-side state)
  → POST /api/generate-animal { answers: [...] }
      → Server computes (Energy, Vibe) scores
      → Maps to species via species grid
      → Calls Claude API: "Generate a name and quirks for this {species}
        with these personality traits: {answers}"
      → Inserts row into `animals` table
      → Returns { species, display_name, quirks, emote_set, conversation_brief }
  → Reveal animation plays + Role Card shown
  → Player enters /meadow (brief pinned as collapsible panel)
```

### 10.2 Chat Flow

```
Player A clicks Animal B on canvas
  → Create row in `conversations` table (status: active)
  → Subscribe to Realtime channel `chat:{conversation_id}`
  → 3-minute timer starts
  → Both parties see ChatOverlay with their BriefPin visible

  If Animal B is human:
    → Both players see ChatOverlay
    → Messages relay through Realtime channel
    → Emotes sent as special message type

  If Animal B is AI:
    → Player A's messages → POST /api/chat { conversation_id, message }
      → Server loads AI imposter's system prompt (brief + backstory) + full chat_log
      → Calls Claude API with conversation context
      → Adds random delay (1–5s)
      → Inserts AI response into chat_log via Realtime
      → Occasionally auto-triggers an emote

  Timer expires:
    → Chat locks
    → VoteModal appears for both parties
    → Votes recorded in `conversations`
    → RevealAnim shows identity
```

### 10.3 Tell Report Flow

```
Session ends (all chats complete or host triggers end)
  → POST /api/tell-report { session_id }
      → Server loads all conversations for session
      → Constructs Claude prompt with anonymized chat logs + vote outcomes
      → Claude analyzes: bot giveaways, human-mistaken-for-bot patterns,
        common false signals, character-level tells (e.g. "the Otter
        never used its curiosity quirk")
      → Stores report text in session record
  → /debrief page renders results + report
```

---

## 11. Agentic Engineering Plan

All implementation will be AI-first using **Claude Code** (primary) and **Cursor** (secondary for UI iteration).

### 11.1 Development Phases

| Phase | Focus | AI Strategy |
|-------|-------|-------------|
| **Phase 1: Scaffold** | Project setup, Supabase schema, API route stubs | Claude Code generates migration files, env config, and Next.js route scaffolding from this ARCHITECTURE.md |
| **Phase 2: Onboarding** | Quiz UI, species mapping logic, Claude animal generation, reveal animation, role card | Cursor for rapid UI prototyping; Claude Code for API route + prompt engineering |
| **Phase 3: Meadow** | Realtime canvas, animal positioning, presence | Claude Code for Supabase Realtime integration; Cursor for canvas rendering |
| **Phase 4: Chat + AI** | Chat overlay, AI imposter conversation, emotes, timer, conversation brief panel | Claude Code for the imposter prompt pipeline; manual testing of persona quality |
| **Phase 5: Debrief** | Vote results, tell report generation | Claude Code for report prompt + aggregation queries |
| **Phase 6: Polish** | Animations, edge cases, deploy | Cursor for animation polish; Claude Code for bug fixes |

### 11.2 AI Tooling Setup

**`CLAUDE.md`** (repo-level instructions for Claude Code):
- Project context: social deduction game, Next.js + Supabase + Claude API
- Coding conventions: TypeScript strict, Tailwind for styling, Supabase client via `lib/supabase.ts`
- Test expectations: Vitest for unit tests on mapping logic; Playwright for E2E onboarding flow
- PR discipline: one feature per PR, reference GitHub Issue number

**`.cursorrules`**:
- Prefer server components; use `"use client"` only for interactive components (canvas, chat, modals)
- Supabase calls go through `lib/supabase.ts` — never import Supabase client directly in components
- Claude API calls go through `lib/claude.ts` — centralized prompt templates

### 11.3 Quality Verification

- **Prompt regression tests**: Save sample quiz inputs and expected species outputs; verify mapping stability
- **AI imposter smoke tests**: Script a few canned conversations, check that responses stay in-character and under 3 seconds
- **Realtime integration tests**: Verify animal presence updates within 2 seconds (acceptance criteria)
