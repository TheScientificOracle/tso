# TSO — The Scientific Oracle
## Project Context File
*Read this first in any new session. Last updated: V5.*

---

## What TSO Is

TSO is an AI-powered personalised fitness and nutrition coaching web app. It is not a generic workout generator — it builds a complete, science-backed 8-week training and nutrition programme calibrated to the individual user's biology, training history, equipment, and goals.

The core concept: **TradFi in the front, science in the back.** A polished, Apple-inspired consumer interface powered by Claude (Anthropic) via API, with Supabase for persistent user data.

**Target user:** Serious recreational athletes and fitness-minded professionals who want evidence-based programming without paying for a personal trainer.

**Current status:** MVP prototype, V5. Being tested by founder (Rik) ahead of a 5-person beta next week.

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

Users select which members they follow. TSO weights their specific methodology more heavily. No selection = full Brain Trust applied equally.

**2. Programme Consistency**
TSO builds an 8-week fixed block. It actively coaches against programme hopping. Mid-block structural changes are not permitted (except injury/equipment). Small execution refinements only. Week 8 = review and evolve.

**3. Whole Food First**
TSO never recommends supplements in meal plans. Food solutions always come first. Supplements only flagged where: (a) peer-reviewed evidence is strong, (b) gap cannot be closed through food, AND (c) always with explicit "consult your doctor or registered dietitian" caveat.

**4. Scope Boundary**
TSO is not a doctor. Clinical territory (medical conditions, medication interactions, diagnosed deficiencies) → TSO acknowledges limits and directs to a qualified professional. Lab work briefs are GP conversation starters, not self-prescriptions.

**5. Traffic Reciprocity**
Brain Trust members are attributed inline in responses (e.g. "Jeff Nippard's evidence-based approach"). This is intentional — it builds trust with users who recognise these names and creates a referral dynamic.

---

## Technical Architecture

**Stack:**
- Single HTML file (`index.html`) — all CSS, JS, and HTML in one file
- Vercel deployment with Edge Function proxy (`api/proxy.js`)
- Supabase for persistent user data
- Claude API (claude-sonnet-4-20250514) via Vercel proxy to resolve CORS

**Why single file:** Non-technical founder, simple deployment, no build step required.

**Vercel proxy:** Required because the Claude API cannot be called directly from the browser (CORS). The proxy at `/api/proxy` forwards requests server-side.

**Files in repo:**
- `index.html` — the entire app
- `api/proxy.js` — Vercel Edge Function for Claude API proxy
- `vercel.json` — routing config
- `README.md` — basic project description
- `CONTEXT.md` — this file

**Supabase tables:**
- `tso_profile` — user consultation data
- `tso_memory` — conversation memory, programme structure
- `tso_checkins` — daily check-in data (weight, HRV, recovery)
- `tso_meals` — meal log entries

**Multi-user (beta):**
URL parameter `?user=name` sets the USER_ID for all Supabase reads/writes. Each beta tester gets their own link. No authentication — this is intentional for MVP. Full auth is V2.

Example links:
- `your-tso.vercel.app?user=rik`
- `your-tso.vercel.app?user=james`

---

## App Structure

**Four tabs:**
1. **Consult** — initial consultation form (run once, generates programme)
2. **Oracle** — AI chat with The Scientific Oracle
3. **Daily Check-In** — WHOOP + Eufy upload, generates daily protocol
4. **Progress** — body composition trends, HRV, nutrition log, progress photos

**My Programme tab** (within Oracle view):
- Training plan with day tabs and exercise list
- 8-week progression table
- Nutrition targets
- Consistency banner (dynamic by week)
- "Start a New Training Plan" button

---

## Consultation Form (V5)

**Section 1 — Biometrics:** Age, sex, weight, height, body fat %, wrist circumference, cycle data (female)

**Section 2 — Training:**
- Experience: 4 cards (Just starting out / Getting serious / Experienced lifter / Competitive athlete)
- Days per week, session length dropdowns
- Equipment: dropdown (Full Gym / Dumbbells only / Barbell + rack / Resistance bands / Home gym / No equipment)
- Injuries: free text (kept because injuries are too varied for chips)
- Muscle groups (lagging): chip grid of 8

**Section 3 — Nutrition & Lifestyle:**
- Diet type: dropdown
- Meals per day: dropdown
- Sleep: number input
- Stress: 4 anchored cards (Low / Moderate / High / Very high) — replaces 1-10 input
- Allergies: free text
- Supplements: 15 chips (disclosure only) + overflow text field

**Section 4 — Goals:**
- Goal: 6 cards (Build muscle / Lean bulk / Change body shape / Get stronger / Athletic performance / Health & longevity)
- Timeline: dropdown
- Barriers: free text

**Section 5 — Brain Trust:** 12 chips, optional

**Header:** "How TSO works" dark banner (4 bullet points setting expectations before form)

---

## Key Design Decisions & Rationale

| Decision | Rationale |
|---|---|
| Selected state = blue border + light blue bg + bold blue text | Matches Apple-style clarity. Solid blue fill was too harsh. |
| No free text for Brain Trust | Quality control — prevents users entering non-evidence-based influencers |
| Stress as cards not 1-10 | 1-10 is subjective without anchors. Cards with descriptions are calibrated |
| Supplements as disclosure chips | TSO should not be a supplement recommendation engine. Whole food first. |
| 8-week fixed block | Science: meaningful adaptation takes 6-8 weeks. Programme hopping is the enemy. |
| No mid-block structural changes | Coached consistency > constant optimisation |
| URL-based user IDs for beta | Simplest possible multi-user without authentication overhead |
| Single HTML file | Non-technical founder, no build pipeline, easy to iterate |
| Vercel proxy | CORS requirement for client-side Claude API calls |

---

## Current Version: V5

**V5 changes from V4:**
- "How TSO works" explainer banner added to consultation
- Stress 1-10 input → 4 anchored option cards
- Supplements free text → 15 disclosure chips + overflow field
- Whole food first + supplement caveat + scope boundary rules added to system prompt
- Post-consultation action chips (View My Programme / Explain My Nutrition / First Two Weeks)
- Exercise swap button (⇄) on hover → modal with 3 paths (equipment / pain / preference)
- "Start a New Training Plan" button → confirmation modal → overwrites plan, preserves check-in history
- URL parameter user IDs for beta (`?user=name`)
- Muscle group selection highlight fixed (double-toggle bug resolved)

---

## V2 Product Roadmap (post-MVP)

- **Proper authentication** — email/password or OAuth login
- **Two-way data binding** — Oracle chat writes programme changes directly to Supabase, live re-renders My Programme tab
- **Programme preview before acceptance** — psychological commitment mechanic, requires two-way binding first
- **Plan archiving** — store completed plans rather than overwriting on new plan start
- **YouTube exercise demos** — curated Brain Trust video library per exercise (~50-100 exercises)

---

## How to Resume Work in a New Session

1. Read this file
2. Check `/mnt/user-data/outputs/index.html` or the GitHub repo for current code
3. Continue from the V5 feedback log or start a fresh V6 log
4. Build changes here → paste into Cursor → commit → push → Vercel auto-deploys

---

## Deployment Workflow

1. Build/edit code in this Claude chat
2. Download `index.html`
3. Open in Cursor (File → Open File from Downloads)
4. Open existing `index.html` in repo, Cmd+A, paste, Cmd+S
5. Source Control → commit message → Commit → Sync/Push
6. Vercel auto-deploys (check dashboard for green tick)

`proxy.js` and `vercel.json` rarely change — only touch if API routing changes.
