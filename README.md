# Hoppers 🥔🪵🦫

> **Can you tell who's real?**  
> A shared meadow where you chat with animals and guess if they're your classmate or an AI.

---

## What Is Hoppers?

Hoppers is a real-time social deduction game built for GIX students. You answer 5 personality questions and get assigned a unique animal avatar — a species, a name, and a set of conversational quirks that are distinctly yours. Then you enter the Meadow: a live canvas populated by your classmates' animals and a handful of AI imposters instructed to blend in.

Click any animal, chat for 3 minutes, then vote: **human or bot?**  
The reveal is instant. The debrief is ruthless.

By demo day, every GIX student will have a Hoppers animal — and nobody will agree on which ones were real.

---

## Features

- 🐾 **AI Animal Generation** — 5-question personality quiz → Claude assigns your species, name, and quirks
- 🌿 **Live Meadow** — Real-time canvas (Supabase Realtime) where animals wander and wait to be chatted with
- 🕵️ **AI Imposters** — Claude-powered personas seeded into the meadow with GIX-student backstories
- 🗳️ **Vote & Reveal** — Cast your guess after every chat; see the truth immediately
- 📊 **Tell Report** — End-of-session AI analysis of who fooled whom and why

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Next.js (App Router) |
| Database & Realtime | Supabase |
| AI | Claude API |
| Deployment | Vercel |

---

## Team

| Role | Name |
|------|------|
| Product Owner | Murphy Wei |
| Developer | Shuxian Hong |

---

## Development Timeline

> Agreed between Murphy Wei (Product Owner) and the Developer.

### Check-in 1 — Week 1 End (by **April 13, 2026**)
**Required progress:**
- GitHub repo set up with agreed folder structure
- Supabase project created; `animals` and `conversations` tables migrated
- Animal generation onboarding flow (5-question UI → Claude API call → species/name/quirks returned)
- Basic reveal animation working end-to-end (even with hardcoded data)

### Check-in 2 — Week 5 End (by **May 2, 2026**)
**Required progress:**
- Live Meadow view functional: animals appear in real time via Supabase Realtime
- Chat overlay opens on animal click; 3-minute countdown timer enforced
- Vote modal fires at end of chat; vote stored in `conversations` table
- At least 1 AI imposter persona active in the meadow and responding via Claude API

### Check-in 3 — Week 7 End (by **May 15, 2026**)
**Required progress:**
- Post-round debrief page complete with vote accuracy summary per chat
- AI "tell report" generated via Claude API and displayed on debrief page
- All three views (Onboarding → Meadow → Debrief) connected end-to-end
- App deployed to Vercel at a public URL

### Final Delivery — Demo Day (**June 1, 2026**)
- All must-have acceptance criteria met (see `SPEC.md`)
- App accessible via public URL
- At least 5 AI imposter personas seeded for demo

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

