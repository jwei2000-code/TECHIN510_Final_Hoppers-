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

## Project Structure (planned)

```
hoppers/
├── app/
│   ├── onboarding/      # 5-question quiz + animal reveal
│   ├── meadow/          # Live canvas + chat overlay
│   └── debrief/         # Vote results + tell report
├── lib/
│   ├── supabase.ts      # Supabase client
│   └── claude.ts        # Claude API helpers
├── supabase/
│   └── migrations/      # DB schema migrations
├── SPEC.md
└── README.md
```

---

## Getting Started (Developer Setup)

```bash
# 1. Clone the repo
git clone <repo-url>
cd hoppers

# 2. Install dependencies
npm install

# 3. Set environment variables
cp .env.example .env.local
# Fill in: NEXT_PUBLIC_SUPABASE_URL, SUPABASE_SERVICE_KEY, ANTHROPIC_API_KEY

# 4. Run locally
npm run dev
```

---

## License

