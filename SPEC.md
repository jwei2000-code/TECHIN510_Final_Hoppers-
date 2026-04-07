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

---

## Issue 1: Set up project structure and database schema

### Tasks
- Initialize Next.js project with App Router
- Connect Supabase project
- Create `animals` table (id, display_name, species, quirks, is_ai, user_id)
- Create `conversations` table (id, animal_a, animal_b, chat_log, vote_by_a, vote_by_b)
- Set up environment variables (.env.local)

### Acceptance Criteria
- [ ] Both tables exist in Supabase
- [ ] App runs locally with `npm run dev`

---

## Issue 2: Build animal generation onboarding flow

### Tasks
- Design 5-question personality quiz UI
- Call Claude API with quiz answers to generate species, name, and 2–3 quirks
- Store generated animal in `animals` table
- Build reveal animation screen

### Acceptance Criteria
- [ ] User completes quiz in under 60 seconds
- [ ] Claude returns species, name, and quirks
- [ ] Animal saved to database
- [ ] Reveal animation displays correctly

---

## Issue 3: Build Live Meadow view with real-time presence

### Tasks
- Build canvas/grid UI showing all active animals
- Use Supabase Realtime to show animals joining/leaving
- Animals appear within 2 seconds of joining

### Acceptance Criteria
- [ ] Meadow updates in real time
- [ ] Each animal displays their species and name
- [ ] Works with multiple browser tabs simultaneously

---

## Issue 4: Implement chat overlay with 3-minute timer and vote modal

### Tasks
- Click on animal → opens chat overlay
- 3-minute countdown timer enforced
- Chat auto-closes when timer hits 0
- Vote modal appears: "Human or Bot?"
- Vote stored in `conversations` table
- Reveal shown immediately after vote

### Acceptance Criteria
- [ ] Timer enforces 3-minute limit
- [ ] Vote is recorded before reveal is shown
- [ ] Reveal displays who/what the animal actually was

---

## Issue 5: Seed and manage AI imposter personas

### Tasks
- Generate AI personas via Claude API (GIX track, hobby, stress point)
- Store AI animals in `animals` table with `is_ai = true`
- AI responds to chat messages via Claude API within 3 seconds
- AI instructed to sound like a grad student, not a chatbot

### Acceptance Criteria
- [ ] At least 3 AI imposters active in meadow
- [ ] AI responds within 3 seconds
- [ ] AI persona stays consistent throughout conversation

---

## Issue 6: Build post-round debrief page with AI tell report

### Tasks
- Display vote accuracy summary for each chat in the session
- Call Claude API to generate "tell report" analyzing the session
- Show which phrases gave away bots, which humans were mistaken for bots

### Acceptance Criteria
- [ ] Debrief shows results for all chats in session
- [ ] Tell report generated within 10 seconds of session end
- [ ] Tell report is specific to that session's conversations

---

## Issue 7: Deploy to Vercel and end-to-end testing

### Tasks
- Deploy app to Vercel
- Set all environment variables in Vercel dashboard
- Test full flow: onboarding → meadow → chat → vote → debrief
- Share public URL

### Acceptance Criteria
- [ ] App accessible via public Vercel URL
- [ ] All three views work end-to-end in production
- [ ] At least 5 AI imposters seeded for demo day

