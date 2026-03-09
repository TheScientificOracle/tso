# TSO — The Scientific Oracle
## Project Context File
*Read this first in any new session. Last updated: V7.*

---

## What TSO Is

TSO is an AI-powered personalised fitness and nutrition coaching web app. It is not a generic workout generator — it builds a complete, science-backed 8-week training and nutrition programme calibrated to the individual user's biology, training history, equipment, and goals.

The core concept: a polished, Apple-inspired consumer interface powered by Claude (Anthropic) via API, with Supabase for persistent user data.

**Target user:** Serious recreational athletes and fitness-minded professionals who want evidence-based programming without paying for a personal trainer.

**Current status:** V7 deployed. Being tested by founder (Rik).

---

## Product Philosophy

**1. The Brain Trust**
TSO's recommendations are grounded in peer-reviewed research filtered through a curated roster of evidence-based practitioners. These are not celebrity endorsements — every member is vetted for scientific rigour:

- Jeff Nippard — evidence-based hypertrophy, full ROM, scientific framing
- Dr Mike Israetel — MEV/MAV/MRV volume landmarks, mesocycle structure (RP methodology)
- Layne Norton — flexible dieting, protein timing, PHAT principles
- Andrew Huberman — morning protocols, sleep optimisation, cortisol/dopamine management
- Eric Trexler — supplement evidence hierarchy, nutrition research
- Alan Thrall — practical strength training
- Dr Stacy Sims — female physiology, cycle syncing ("women are not small men")
- Stephanie Buttermore — female bodybuilding and science
- Dr Abbie Smith-Ryan — exercise physiology research
- Lauren Pak — evidence-based female training
- Dr Gabrielle Lyon — muscle-centric medicine
- Shona Vertue — mobility, training sustainability, mental health of fitness

Users select which members they follow during consultation. TSO weights their specific methodology more heavily. No selection = full Brain Trust applied equally.

**2. Programme Consistency**
TSO builds an 8-week fixed block. It actively coaches against programme hopping. Mid-block structural changes are not permitted (except injury/equipment). Small execution refinements only. Week 8 = review and evolve.

**3. Whole Food First**
TSO never recommends supplements in meal plans. Food solutions always come first. Supplements only flagged where: (a) peer-reviewed evidence is strong, (b) gap cannot be closed through food, AND (c) always with explicit "consult your doctor or registered dietitian" caveat.

**4. Scope Boundary**
TSO is not a doctor. Clinical territory (medical conditions, medication interactions, diagnosed deficiencies) → TSO acknowledges limits and directs to a qualified professional. Lab work briefs are GP conversation starters, not self-prescriptions.

**5. Traffic Reciprocity**
Brain Trust members are attributed inline in responses. This builds trust with users who recognise these names and creates a referral dynamic.

---

## Technical Architecture

**Stack:**
- Single HTML file (`index.html`) — all CSS, JS, and HTML in one file
- Vercel deployment with Edge Function proxy (`api/proxy.js`)
- Supabase for persistent user data
- Claude API (`claude-sonnet-4-20250514`) via Vercel proxy to resolve CORS

**Why single file:** Non-technical founder, simple deployment, no build step required.

**Vercel proxy:** Required because the Claude API cannot be called directly from the browser (CORS). The proxy at `/api/proxy` forwards requests server-side.

**Files in repo:**
- `index.html` — the entire app
- `api/proxy.js` — Vercel Edge Function for Claude API proxy
- `vercel.json` — routing config
- `TSO-CONTEXT.md` — this file

**Supabase tables:**
- `tso_profile` — user consultation data (includes workoutTime from V7)
- `tso_memory` — conversation memory, programme structure
- `tso_checkins` — daily check-in data (weight, HRV, recovery)
- `tso_meals` — meal log entries

**Multi-user:**
URL parameter `?user=name` sets the USER_ID for all Supabase reads/writes. Each user gets their own link. No authentication — intentional for MVP. Full auth is V2.

**Live URL:** https://tso-livid.vercel.app/

---

## App Structure (V7)

**Three tabs (post-consultation):**
1. **Today** — daily hub (briefing card + upload slots + chat)
2. **Programme** — training plan, nutrition targets, 8-week progression
3. **Progress** — body composition trends, HRV, nutrition log, progress photos

**Consult tab** is hidden after consultation completes. Restored when user clicks "Start a New Training Plan."

**During first visit:** Only the Consult tab is shown. After consultation completes, Today becomes the default and Consult is hidden.

### Today Tab — 3 Zones

**Zone 1 — Daily Briefing card:**
- Gold-tinted background (`#fdf6eb`), `rgba(200,169,110,0.25)` border
- Atomic orbit avatar (32px), Cormorant Garamond 15px text
- Oracle-generated daily message (context-aware: week in block, training day, cycle phase if applicable)
- Status pills (green/gold/blue)
- Upload progress bar (X of 4)
- Refresh button to regenerate briefing

**Zone 1 continued — Upload slots:**
- 2×2 grid: WHOOP Sleep / Workout / Nutrition / Body Metrics
- Dashed border pending, solid green + tick badge when done
- Ad-hoc "Add another screenshot" row below grid
- WHOOP and Body Metrics uploads feed into existing data extraction pipeline

**Zone 2 — Chat:**
- Continuous thread, iMessage-style
- Quick reply chips (horizontal scroll): Today's Meal Plan / Programme Check / Next Session / Micronutrient Audit
- Standard textarea input

### Programme Tab
- Day tabs, exercise list with ⇄ swap button per exercise
- 8-week progression table
- Nutrition targets (calories / protein / carbs / fat)
- Consistency banner (dynamic message by week)
- "Start a New Training Plan" button → confirmation modal

### Progress Tab
- Body composition trend (7-day weight chart)
- Recovery & readiness trend (14-day HRV)
- Menstrual cycle phase card (female users only)
- Progress photos (4 slots)
- Today's nutrition log + macro progress bars
- Lab nudge card (frequency based on age)

---

## Oracle Consultation — Full Question Flow (V7)

Conversational, one question at a time. Dark theme. Atomic orbit avatar animates throughout.

1. Intro / confirm
2. Biological sex (cards)
3. Age (number)
4. Weight in kg (number)
5. Height in cm (number)
6. *(Female branch — inserted dynamically after height):*
   - Cycling status (cards: Currently cycling / On hormonal contraception / Not currently cycling)
   - Average cycle length (chips)
7. Body composition / body fat estimate (cards)
8. Training experience (cards: Just starting out / Getting serious / Experienced lifter / Competitive athlete)
9. Training days per week (chips: 3/4/5/6 days)
10. **Preferred workout time** *(added V7)* (chips: Early morning 5–7am / Morning 7–9am / Lunchtime 12–2pm / After work 5–7pm / Evening 7–9pm / Varies)
11. Session length (chips: ~45/~60/~75/~90 min)
12. Equipment (cards: Full Commercial Gym / Home Gym with weights / Barbell + Rack Only / Bands & Bodyweight)
13. Injuries / physical limitations + equipment constraints (free text)
14. Diet type (cards: Everything / Vegetarian / Vegan / Low carb/Keto / Mediterranean)
15. Eating structure per day (chips: meal+snack combos e.g. "3 meals + 1 snack")
16. Average sleep (chips: <6 / 6–7 / 7–8 / 8+ hrs)
17. Current stress load (cards: Low / Moderate / High / Very high — anchored descriptions)
18. Food allergies / intolerances (free text)
19. Current supplements (multi-select chips, optional — disclosure only)
20. Primary goal for 8-week block (cards: Build muscle / Lean bulk / Body recomposition / Get stronger / Athletic performance / Health & longevity)
21. Commitment timeline (cards: 3 months / 6 months / 12 months / Lifestyle)
22. Barriers / what has derailed training before (free text)
23. Brain Trust selection (multi-select chips, optional — 12 practitioners)

---

## Key Architecture Notes

- `beginConsultation()` maps `userData` (Oracle flow) → `consultationData` object → saves to Supabase
- `workoutTime` flows into: system prompt MEAL TIMING rule, `buildConsultPrompt`, `generateStructuredProgramme`, `generateBriefing`, `profileStr`
- `programmeData` full JSON injected into every system prompt — TSO never invents exercises
- `SWAP: [Name]` protocol — Oracle first-line response format triggers programme tab update + Supabase save
- `openSwapModal(exerciseName, dayIdx, exIdx)` — name captured into local var before `closeSwapModal()` nulls globals
- `USER_ID` from `?user=` URL param, defaults to `tso-user-1`
- Supabase URL: `https://pzzafdhvbdabqjeurpna.supabase.co`

### System Prompt Key Rules
- **MEAL TIMING:** All schedules built around user's `workoutTime`. Hard block on generic templates that ignore training window.
- **PROGRAMME CONSISTENCY:** Never suggest structural changes before week 8 unless new injury or equipment change.
- **WHOLE FOOD FIRST:** Supplements only where peer-reviewed evidence is strong AND gap can't be closed through food.
- **CYCLE SYNCING:** Full phase protocol for female users (Follicular / Ovulatory / Early Luteal / Late Luteal / Menstrual).
- **SCOPE BOUNDARY:** Clinical territory → acknowledge limits, direct to professional.

---

## Design Decisions & Rationale

| Decision | Rationale |
|---|---|
| Apple light palette (`#F5F5F7` bg, `#FFFFFF` surfaces, `#0066FF` accent) | Clean, trustworthy, familiar. Matches target user's device aesthetic. |
| Dark theme strictly for Oracle consultation only | Consultation is a premium, immersive moment. Main app stays clean. |
| Atomic orbit avatar (3 elliptical orbits, pulsing gold nucleus) | Scientific metaphor — always animating, never static. Gold = premium signal. |
| Orbits A+C clockwise, B counter-clockwise | Visual complexity and perceived intelligence without being chaotic. |
| Cormorant Garamond for briefing card text | Serif creates editorial authority. Contrast with Plus Jakarta Sans body copy. |
| Gold (`#c8a96e`) as accent only, never dominant | Gold is a premium signal — overuse cheapens it. |
| Consult tab hidden post-consultation | Reduces cognitive load. Consultation is a one-time event. |
| Today tab as post-onboarding home | Daily hub pattern — users return daily, not just to review their programme. |
| Upload slots as named tiles not generic dropzone | Named slots set clear expectation of what data TSO wants. Reduces friction. |
| No free text for Brain Trust | Quality control — prevents users entering non-evidence-based influencers. |
| Stress as anchored cards not 1-10 | 1-10 is subjective without anchors. Cards with descriptions are calibrated. |
| Supplements as disclosure chips | TSO is not a supplement recommendation engine. Whole food first. |
| 8-week fixed block | Science: meaningful adaptation takes 6-8 weeks. Programme hopping is the enemy. |
| No mid-block structural changes | Coached consistency > constant optimisation. |
| URL-based user IDs | Simplest possible multi-user without authentication overhead. Full auth is V2. |
| Single HTML file | Non-technical founder, no build pipeline, easy to iterate in Claude. |
| Vercel proxy | CORS requirement for client-side Claude API calls. |
| `programmeData` JSON in every system prompt | Prevents TSO hallucinating exercises not in the user's actual programme. |
| `SWAP: [Name]` protocol on first line | Parseable format that lets the app intercept and update the programme tab without a separate API call. |
| `workoutTime` as hard rule in system prompt | Prevents TSO generating schedules incompatible with the user's actual day (e.g. 11am gym for someone who works 9–5). |

---

## V7 Bug Fixes (all deployed)

1. Auto-scroll after each Oracle question
2. Brain Trust chips wrapping — `overflow-x: hidden` on `#o-input`
3. Exercise swap loses exercise name — captured before `closeSwapModal()` nulls globals
4. Exercise swap doesn't update programme tab — `SWAP:` protocol + re-render + Supabase save
5. TSO hallucinating programme contents — full `programmeData` JSON in every system prompt
6. Injuries question expanded to include equipment limitations
7. Meals question updated to meal+snack combos (e.g. "3 meals + 1 snack")

---

## Monetisation Plan (discussed, not built)

- **Free tier:** Consultation + programme + 2 weeks basic access
- **Paid ~$15–20/month:** Daily hub, chat history, cycle syncing, nutrition tracking, progress
- **Moat:** Brain Trust methodology, conflict logic, female physiology integration, programme consistency rules
- **App Store:** Capacitor wrapper for fastest path; React Native for proper V2+
- **WHOOP/Eufy:** Currently screenshot-based; proper API integrations planned for paid tier

---

## V2 Product Roadmap (post-MVP)

- Proper authentication (email/password or OAuth)
- Two-way data binding — Oracle chat writes programme changes directly to Supabase, live re-renders
- Programme preview before acceptance — psychological commitment mechanic
- Plan archiving — store completed plans rather than overwriting
- YouTube exercise demos — curated Brain Trust video library per exercise
- WHOOP and Eufy API integrations (replace screenshot uploads)

---

## V8 Queue
*Nothing confirmed yet — populate after reviewing V7 live.*

---

## Deployment Workflow

1. Build/edit code in Claude chat
2. Download `index.html`
3. Open in Cursor → replace existing file content → save
4. Terminal: `git add . && git commit -m "message" && git push`
5. Vercel auto-deploys on push — check dashboard for green tick

`proxy.js` and `vercel.json` rarely change — only touch if API routing changes.

---

## How to Resume in a New Session

1. Upload this file at the start of the chat
2. Say what you want to build or fix
3. Claude will have full context immediately — no other files needed
