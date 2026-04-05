# SPEC.md — Hoppers 🥔🪵🦫

> **"Can you tell who's real?"**  
> A shared meadow where you chat with animals and guess if they're your classmate or an AI.

---

## Client

**Murphy Wei**  

## Developer

**Shuxian Hong**  

---

## Agreed Development Fee

**50 GIX Bucks**

---

## Project Overview

Hoppers is a real-time social deduction game for GIX students. Each player is assigned a unique animal avatar generated from a short personality quiz. Players enter a shared "meadow" and have anonymous 3-minute chats with other animals — some of which are real classmates and some of which are AI imposters. After each chat, you vote: human or bot? At the end of the session, an AI-generated "tell report" reveals who fooled whom and why.

---

## User Stories

### Onboarding
- **As a new user**, I want to answer 5 short personality questions so that the system generates a unique animal avatar (species, name, 2–3 conversational quirks) that represents me in the meadow.
- **As a new user**, I want to see my animal revealed in a satisfying reveal animation so that I feel connected to my persona before entering the meadow.

### Meadow (Live Play)
- **As a player**, I want to see a real-time canvas of the meadow with other animals moving around so that the space feels alive and social.
- **As a player**, I want to click on any animal in the meadow to open a 3-minute anonymous chat window so that I can interact without knowing who or what I'm talking to.
- **As a player**, I want to see a countdown timer during each chat so that I know when the conversation will end.
- **As a player**, I want to cast a "human or bot" vote immediately after each chat ends so that I can record my guess before the reveal.
- **As a player**, I want to immediately see the reveal (who or what that animal was) after voting so that I get instant feedback on my guess.

### AI Imposters
- **As a player**, I want AI animal personas to be seeded into the meadow alongside real users so that I cannot be certain which animals are real people.
- **As an AI imposter animal**, I want to be given a generated backstory (GIX track, hobby, stress point) and instructed to converse naturally so that I pass as a plausible grad student.

### Post-Round Debrief
- **As a player**, I want to see a session debrief page after the round ends so that I can review my vote accuracy across all chats.
- **As a player**, I want an AI-generated "tell report" that analyzes which phrases gave away bots, which humans were mistaken for bots, and what the most common false signals were so that I can improve my detection in future rounds.

---

## Desired Specifications

### Views / Pages

| View | Description |
|------|-------------|
| **1. Onboarding / Animal Generation** | 5-question personality flow → AI assigns species, name, quirks → reveal animation |
| **2. Live Meadow** | Real-time canvas, clickable animals, chat overlay with timer, vote modal |
| **3. Post-Round Debrief** | Vote result summary per chat + AI "tell report" paragraph |

### AI Features

1. **Animal Generation** — User answers 5 personality questions → Claude API assigns a species, a name, and 2–3 unique conversational quirks.
2. **AI Imposters** — Claude API personas seeded into the meadow alongside real users; each given a generated backstory (GIX track, hobby, stress point) and instructed to impersonate a plausible grad student, not a chatbot.
3. **Post-Round Tell Report** — After each session, Claude API generates an analytical debrief identifying which phrases gave away bots, which humans were mistaken for bots, and the most common false signals.

### Data Model

**`animals` table**
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | Primary key |
| display_name | text | Animal's generated name |
| species | text | AI-assigned species |
| quirks | text[] | 2–3 conversational traits |
| is_ai | boolean | True for AI imposters |
| user_id | uuid | Null for AI personas |
| created_at | timestamp | |

**`conversations` table**
| Column | Type | Notes |
|--------|------|-------|
| id | uuid | Primary key |
| animal_a | uuid | FK → animals |
| animal_b | uuid | FK → animals |
| chat_log | jsonb | Array of {sender, text, timestamp} |
| vote_by_a | text | "human" or "bot" |
| vote_by_b | text | "human" or "bot" |
| revealed_at | timestamp | When reveal was shown |

### Tech Stack

- **Frontend**: Next.js (App Router)
- **Backend / DB**: Supabase (Postgres + Supabase Realtime for live meadow presence)
- **AI**: Claude API (claude-sonnet model) for animal generation, imposter personas, and tell report
- **Deployment**: Vercel

### Must-Have Feature (Definition of Done)

> A user enters the meadow, has a 3-minute anonymous chat with another animal, casts their vote (human or bot), and immediately sees whether they were right — and who or what that animal actually was.

### Out of Scope (for this sprint)

- Persistent user accounts / login (can use anonymous Supabase sessions)
- Mobile-native app
- Voice or video chat
- Leaderboards or persistent scoring across sessions

---

## Acceptance Criteria

- [ ] Animal generation flow completes in under 60 seconds for any user
- [ ] Meadow shows real-time presence (animals appear/disappear within 2 seconds of join/leave)
- [ ] Chat timer enforces 3-minute limit and auto-closes the chat window
- [ ] Vote is recorded before reveal is shown
- [ ] AI imposters respond within 3 seconds of a message
- [ ] Tell report is generated and displayed within 10 seconds of session end
- [ ] App is accessible via public URL on demo day

---

## Complexity

**Ambitious** — Real-time multiplayer + AI personas + Claude API integration in ~40–60 developer hours.
