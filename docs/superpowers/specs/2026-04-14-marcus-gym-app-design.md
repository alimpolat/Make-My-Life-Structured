# Marcus — AI Gym Training Companion

**Date:** 2026-04-14
**Author:** Alim Polat + Claude
**Status:** Design approved, ready for implementation planning

---

## Overview

Marcus is a Progressive Web App (PWA) that serves as a personalized AI gym training companion. It connects to Coach Marcus (an AI agent powered by Gemini) who knows the user's health data, training plan, gym equipment, and session history. The app shows today's workout with exercise visuals and muscle maps, and lets the user chat with Marcus in natural language for real-time coaching guidance.

## Problem

The user has a comprehensive 12-week gym training plan (created by Coach Marcus based on a NEKO Health scan) but no way to follow it at the gym. Opening a markdown file on a phone is impractical. He needs:
- Visual exercise guidance (what does this movement look like, which muscles)
- Contextual coaching (swap exercises for injuries, adjust weights, answer questions)
- Persistent memory (Marcus remembers last session, adapts this one)

## User Profile

- 45M, AI engineer, 83.4 kg, BMI 28.9, grip strength 40 kg (bottom 18%)
- Heart age 42 (excellent cardiovascular health)
- Trains at Friskis&Svettis Sundbyberg + Solna (Stockholm)
- 4 gym days/week: Mon + Fri (Strength A: Push + Core), Wed + Sat (Strength B: Pull + Legs)
- 12-week progressive program: Phase 1 (Foundation), Phase 2 (Building), Phase 3 (Strength)
- Dedicated grip strength protocol running alongside every session

## Product Vision

**V1 (now):** Personal gym companion for one user.

**Future:** "Solution as a Service" — a new SaaS paradigm where every user gets their own AI coach that knows their body, health data, schedule, and constraints. Not software with dashboards. A solution that adapts to the individual.

---

## Core Screens

### 1. Today's Workout

The landing screen when you open the app. Shows:
- Current phase + week (e.g., "Phase 1 — Week 2")
- Workout name (e.g., "Strength A: Push + Core")
- List of exercises as cards, each showing: name, sets x reps, rest time, and a small muscle group indicator
- Warm-up and cool-down sections at top and bottom
- A floating chat button to open Marcus chat

On rest days (Tue, Thu, Sun), shows: "Rest day. Go for a walk. How are you feeling?" with chat available.

### 2. Exercise Detail

Tapped from the workout list. Full-screen view:
- Exercise illustration/animation from wger.de API
- Muscle map highlighting primary + secondary muscles
- Sets, reps, rest time
- Friskis-specific location note (e.g., "Lifting platform — barbell + squat rack")
- Coach note from Marcus (e.g., "Last time 12kg felt easy. Try 14kg today.")
- Swipe left/right to navigate between exercises

### 3. Chat with Marcus

Text chat interface. Marcus has full context:
- User's NEKO health data (heart age 42, grip 40kg, BMI 28.9)
- Current phase, week, and today's workout
- Last 3 session notes from gbrain
- Any injuries or swaps previously mentioned
- The full gym-training-plan context

Example interactions:
- "My shoulder feels off today" → Marcus offers swaps
- "That was easy, go heavier?" → Marcus recommends specific weight increase
- "I skipped Wednesday" → Marcus reassures and adjusts
- "What should I eat after this?" → Marcus gives nutrition guidance

### 4. Progress Overview

Simple view showing:
- Current phase and week number
- Total sessions completed
- A timeline/streak indicator
- Key notes from recent sessions (from gbrain)

No detailed charts or analytics in v1 — just enough to see momentum.

---

## Architecture

```
Phone (PWA)
    │
    │ HTTPS
    ▼
Cloud Run (FastAPI - Python)
    ├── /api/workout/today  → today's workout JSON
    ├── /api/exercise/{id}  → exercise detail + wger visual
    ├── /api/chat           → Marcus chat (Gemini + gbrain context)
    └── /api/progress       → progress summary
            │
    ┌───────┼──────────┐
    ▼       ▼          ▼
 Gemini   gbrain    wger.de
 API      (memory)  (exercise visuals)
```

### Frontend (PWA)

- **Tech:** Vanilla HTML/CSS/JS — no framework, no build step
- **Design:** Dark theme, mobile-first, one-hand operation between sets
- **Offline:** Service worker caches workout data + exercise images
- **Install:** Add to home screen via manifest.json
- **Hosting:** Firebase Hosting (free tier, fast CDN)

### Backend (FastAPI on Cloud Run)

- **Tech:** Python 3.12+, FastAPI
- **Container:** Single Dockerfile, deployed to Cloud Run
- **Scaling:** Scales to zero when not in use, cold start ~2 seconds
- **GCP Project:** gg-gcpsbprojs-004 (existing, credentials tested)

### Gemini Integration

- **Flash (default):** All in-session chat. Fast responses between sets.
- **Pro (on demand):** End-of-week reviews, plan adjustments, deeper analysis.
- **System prompt assembly:**
  1. Marcus personality (from health-coach.md agent definition)
  2. User profile (NEKO data, age, constraints)
  3. Today's workout context (phase, week, exercises)
  4. Recent session history (last 3 sessions from gbrain)
  5. Any active injuries or modifications

### gbrain (Persistent Memory)

- Used as a Python library (not MCP server)
- Reads: NEKO scan data, workout plan, previous session notes
- Writes: Session summary after each gym visit (exercises done, weights, notes, Marcus observations)
- Each session becomes a brain page with timeline entry
- Enables Marcus to say: "Last Monday you did 12kg bench and said it was hard. How about trying 12kg again before going up?"

### Exercise API (wger.de)

- Free, open-source exercise database
- Provides: exercise illustrations, muscle group mappings, exercise descriptions
- Cached locally (exercises don't change)
- Fallback: static bundled images if API is unavailable

---

## Data Model

### Workout Plan (workout_plan.json)

Converted from gym-training-plan.md. Structure:

```json
{
  "start_date": "2026-04-14",
  "phases": [
    {
      "id": "phase-1",
      "name": "Foundation",
      "weeks": [1, 4],
      "workouts": {
        "strength-a": {
          "name": "Push + Core",
          "days": ["monday", "friday"],
          "exercises": [
            {
              "id": "goblet-squat",
              "name": "Goblet Squat",
              "sets": 3,
              "reps": 12,
              "rest_seconds": 60,
              "wger_exercise_id": 300,
              "muscles_primary": ["quadriceps", "glutes"],
              "muscles_secondary": ["core"],
              "friskis_location": "Free weights area — grab a dumbbell from the rack (start 10-14 kg)",
              "notes": "Hold at chest, squat to parallel."
            }
          ]
        },
        "strength-b": { "..." : "..." }
      }
    }
  ],
  "grip_protocol": {
    "every_session": ["dead-hang", "farmers-walk"],
    "twice_per_week": ["plate-pinch", "wrist-curls"],
    "daily_home": ["tennis-ball-squeeze", "towel-wring"]
  },
  "warmup": { "..." : "..." },
  "cooldown": { "..." : "..." }
}
```

### Session Page (gbrain)

After each gym visit, a brain page is created:

```markdown
---
type: session
title: Gym Session YYYY-MM-DD
tags: [gym, strength-a/b, phase-N, week-N]
---

[Session summary: exercises completed, weights used, feel/difficulty notes]

Notes from Marcus:
[AI observations, weight adjustment recommendations, injury flags]

---
- YYYY-MM-DD: Timeline entry
```

### Day Logic

No calendar sync needed:
1. Day of week → workout type (Mon/Fri = Strength A, Wed/Sat = Strength B, others = rest)
2. Current date - start_date → week number → phase
3. Serve appropriate workout

---

## API Endpoints

### GET /api/workout/today
Returns today's workout based on day-of-week and current phase/week.
Response: workout JSON with exercise list, sets, reps, rest times.
On rest days: returns `{ "rest_day": true, "message": "..." }`

### GET /api/exercise/{exercise_id}
Returns exercise detail including wger illustration URL, muscle groups, Friskis location note, and any personalized notes from Marcus (based on recent gbrain data).

### POST /api/chat
Request: `{ "message": "my shoulder hurts" }`
Response: Streamed Marcus response from Gemini (with full context injected into system prompt).
Side effect: significant exchanges are summarized and written to gbrain.

### GET /api/progress
Returns: current phase, week, sessions completed, recent session summaries.

---

## UI Design Principles

- **Dark theme** — gym-friendly, looks premium, saves battery on OLED
- **Mobile-first** — designed for one-hand use between sets
- **Large touch targets** — sweaty fingers, no tiny buttons
- **Minimal text** — visuals and numbers dominate, not paragraphs
- **Fast** — exercise images cached, workout data prefetched
- **Offline-capable** — service worker caches current workout, chat needs connectivity

---

## Deployment

- **Frontend:** Firebase Hosting (`firebase deploy`)
- **Backend:** Cloud Run (`gcloud run deploy marcus-api --source .`)
- **GCP Project:** gg-gcpsbprojs-004 (credentials already tested)
- **Domain:** None for v1 (use Firebase default URL + Cloud Run URL)

---

## Cost (Personal Use)

| Service | Monthly Cost |
|---------|-------------|
| Cloud Run | Free (2M requests/month) |
| Gemini Flash | ~$0.40 (4 sessions/week x 10 messages x 4 weeks) |
| Gemini Pro | ~$0.20 (4 weekly reviews) |
| Firebase Hosting | Free tier |
| wger.de API | Free |
| **Total** | **< $1/month** |

---

## Out of Scope for V1

- Set/rep logging and weight tracking (just "show me what's next")
- Voice input (text chat only)
- Multiple users / auth
- Other coaches (only Marcus)
- Full life coaching (gym training only)
- App store distribution (PWA only)
- Workout plan editing in-app (plan lives in repo, Marcus adapts via chat)

---

## Future Expansion Path

- **V2:** Add workout logging (weight, reps per set), progress charts
- **V3:** Voice input for hands-free coaching between sets
- **V4:** Other coaches (Kai for energy, Elena for family) as separate "modes"
- **V5:** Multi-user, onboarding flow, connect to health data APIs
- **Product:** "Solution as a Service" — every user gets their own AI coach that knows their body, adapts every session, remembers everything
