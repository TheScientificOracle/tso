# TSO — The Scientific Oracle
## Project Context File
*Read this first in any new session. Last updated: V8 (March 2026).*

---

## What TSO Is

TSO is an AI-powered personalised fitness and nutrition coaching web app. It is not a workout generator — it is a PhD-level advisor that builds a complete, science-backed 8-week training and nutrition programme calibrated to the individual user's biology, training history, equipment, and goals. The programme is a byproduct of coaching, not the product itself.

**Core concept:** A polished, Apple × WHOOP × Equinox aesthetic powered by Claude (Anthropic) via API, with Supabase for persistent user data.

**Target user:** Serious recreational athletes and fitness-minded professionals who already train consistently and want evidence-based, high-context coaching without paying for a personal trainer. TSO is not for beginners.

**Current status:** V8 deployed. Being tested by founder (Rik).
**Live URL:** https://tso-livid.vercel.app/

---

## The V8 Pivot — What Changed and Why

V7 opened with a 23-question consultation questionnaire. V8 is a fundamental architecture change:

**Old flow:** Land → questionnaire (form-based, 23 questions, dark theme) → programme generated → Today tab unlocks.

**New flow:** Land → Oracle chat (conversational, light theme) → tabs unlock progressively and silently based on conversation state → programme offered mid-conversation when context is sufficient.

**Why:** The questionnaire was optimised for data extraction, not trust-building. TSO's real moat is advisor-level intelligence. Starting with a form framed TSO as a workout generator. Starting with a conversation frames TSO as a coach. The plan is a byproduct of that relationship, not the entry point.

---

## Product Philosophy

**1. The Brain Trust**
TSO's recommendations are grounded in peer-reviewed research filtered through a curated roster of evidence-based practitioners:

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

**2. Programme Consistency**
TSO builds an 8-week fixed block. It actively coaches against programme hopping. Mid-block structural changes are not permitted (except injury/equipment). Small execution refinements only. Week 8 = review and evolve.

**3. Whole Food First**
TSO never recommends supplements in meal plans. Food solutions always come first. Supplements only flagged where: (a) peer-reviewed evidence is strong, (b) gap cannot be closed through food, AND (c) always with explicit "consult your doctor or registered dietitian" caveat.

**4. Scope Boundary**
TSO is not a doctor. Clinical territory → acknowledge limits, direct to a qualified professional. Lab work briefs are GP conversation starters, not self-prescriptions.

**5. Traffic Reciprocity**
Brain Trust members are attributed inline in responses. Builds trust with users who recognise these names.

---

## Technical Architecture

**Stack:**
- Single HTML file (`index.html`, ~2030 lines) — all CSS, JS, and HTML in one file
- Vercel deployment with serverless function proxy (`api/proxy.js`)
- Supabase for persistent user data
- Claude API (`claude-sonnet-4-20250514`) via Vercel proxy (resolves CORS)

**Why single file:** Non-technical founder, simple deployment, no build step required.

**Files in repo:**
- `index.html` — the entire app
- `api/proxy.js` — Vercel serverless function for Claude API proxy
- `vercel.json` — routing config
- `CONTEXT.md` — this file

**Supabase tables:**
- `tso_profile` — user coaching data (built through Oracle conversation, not a form)
- `tso_memory` — conversation memory, programme structure JSON
- `tso_checkins` — daily check-in data (weight, HRV, recovery score)
- `tso_meals` — meal log entries

**Proxy auth:** `x-proxy-secret: %pBH$n5FJD8+Yk#` header. Secret stored as Vercel env var `PROXY_SECRET`. API key stored as `ANTHROPIC_API_KEY`.

**Multi-user:** URL parameter `?user=name` sets `USER_ID` for all Supabase reads/writes. Each user gets their own link. No authentication — intentional for MVP.

**Supabase URL:** `https://pzzafdhvbdabqjeurpna.supabase.co`

---

## App Structure (V8)

### Tab System — Progressive Disclosure

Tabs unlock silently and automatically. No fanfare. No gamification.

| Tab | When it unlocks |
|---|---|
| **Oracle** | Always visible. First and only tab on first load. |
| **Today** | After first Oracle exchange (tab nav fades in, Today appears) |
| **Programme** | After plan is generated from conversation OR a plan is uploaded |
| **Progress** | After the user's first check-in (Today tab → upload WHOOP + generate) |

Tab nav starts `opacity:0; pointer-events:none`. After first exchange: `transition: opacity 0.4s ease` fades it in.

**Returning users:** All tabs visible immediately, app goes straight to Today, briefing auto-generates.

**Start New Plan:** Clears Supabase profile + memory, hides all tabs, resets state, returns to Oracle.

---

### Oracle Tab (V8 — primary entry point)

**Header:** Small atomic orbit animation (32px) + "ORACLE" label + italic status text.

**Opening experience (new users only):**
1. TSO sends its opening message automatically:
   > *"I work at the intersection of sports science and nutrition research. PhD level. Evidence-based. I don't follow trends — I follow data. I help athletes like you bring the best out of their daily routine by optimising their training, nutrition and recovery. Most people training seriously are doing more right than they realise — and leaving more on the table than they know. The gap is rarely effort. It's usually one or two things working quietly against everything else. What brought you here?"*

2. Six clickable opening chips appear:
   - "My effort isn't matching my results"
   - "I don't know if my current plan is working"
   - "I want to reach the next level"
   - "My nutrition feels like the weak link"
   - "I'm returning from injury"
   - "Let's just talk"

3. After chip selection → contextual TSO reply → free text input appears.

**Free text input:** Textarea with auto-resize + send button + "Upload Plan" button. Plan upload accepts image/PDF/txt.

**Plan offer:** After 5+ exchanges AND conversation contains goal + training keywords → TSO automatically renders offer block with "Build My Programme →" and "Upload Existing Plan" buttons.

**Plan upload (gap analysis):** User uploads image/PDF of their current plan → TSO extracts content via vision API → reviews vs conversation context → scores out of 100 → identifies gaps → unlocks Programme tab with gap analysis rendered.

**Post-plan chips:** "View My Programme", "What should I eat this week?", "Walk me through Week 1".

**Messaging style:** Oracle messages use Cormorant Garamond serif, 17px. User messages use Plus Jakarta Sans, black bubble.

---

### Today Tab

Three zones:

**Zone 1 — Daily Briefing card:**
- Gold-tinted background (`#fdf6eb`), Cormorant Garamond text
- Oracle-generated daily message (context-aware: week in block, training day, cycle phase if female)
- Status pills (green/gold/blue), upload progress bar, refresh button

**Zone 1 continued — Upload slots:**
- 2×2 grid: WHOOP Sleep / Workout / Nutrition / Body Metrics
- Dashed border pending → solid green + tick badge when uploaded
- Ad-hoc "Add another screenshot" row

**Zone 2 — Chat:**
- Continuous thread, iMessage-style
- Quick reply chips: Today's Meal Plan / Programme Check / Next Session / Micronutrient Audit
- Standard textarea input

---

### Programme Tab

- Day tabs with exercise list and ⇄ swap button per exercise
- 8-week progression table
- Nutrition targets (calories / protein / carbs / fat)
- Consistency banner (dynamic by week)
- **Empty state:** "Talk to Oracle →" button (not "Start Consultation" — that flow is gone)
- "↺ Start a New Training Plan" button → confirmation modal

---

### Progress Tab

- Body composition trend (7-day weight chart)
- Recovery & readiness trend (14-day HRV)
- Menstrual cycle phase card (female users only)
- Progress photos (4 slots)
- Today's nutrition log + macro progress bars
- Lab nudge card (frequency based on age)

---

## V8 JavaScript Architecture

### Key State Variables

```javascript
let oracleBusy = false;            // prevents double-sends
let oracleExchanges = 0;           // counter for plan offer trigger
let coachingContext = {};          // populated from conversation for plan generation
let planOffered = false;           // prevents duplicate plan offers
let tabsVisible = { oracle: true, today: false, programme: false, progress: false };
let consultationData = null;       // profile object (from plan generation or Supabase load)
let conversationHistory = [];      // full Oracle conversation for API calls
let programmeData = null;          // structured programme JSON
let programmeStartDate = null;
```

### Oracle Flow Functions

| Function | Purpose |
|---|---|
| `initNewUser()` | Fires for new users — sends opening message, calls `showOpeningChips()` |
| `showOpeningChips()` | Renders 6 clickable chips in `#o-input` |
| `showFreeTextInput()` | Replaces chips with textarea + send + upload button |
| `handleOpeningChipResponse(option)` | Returns contextual reply to each chip |
| `sendOracleMessage()` | Main send handler — calls `callTSO`, updates history, checks plan offer |
| `afterFirstExchange()` | Fades in tab nav, unlocks Today tab |
| `unlockTab(tabId)` | Silently shows a tab button (checks `tabsVisible` to prevent re-unlock) |
| `checkPlanOffer()` | After 5+ exchanges: checks for goal+training keywords, calls `offerPlan()` |
| `offerPlan()` | Renders plan offer block in chat (Build / Upload buttons) |
| `generatePlanFromConversation()` | Extracts context from conversation → calls `generateStructuredProgramme(d)` → unlocks Programme |
| `showPostPlanChips()` | Renders follow-up action chips after plan is generated |
| `sendOracleQuick(text)` | Injects quick-action text and calls `sendOracleMessage()` |
| `handlePlanUpload(input)` | Reads uploaded plan via vision API → gap analysis → unlocks Programme |

### Supporting Functions (unchanged from V7)

| Function | Purpose |
|---|---|
| `oSay(text)` | Animates typing indicator, then renders Oracle bubble with delay |
| `oAddMsg(who, html)` | Appends message row to `#o-chat` |
| `oShowTyping()` | Shows animated typing dots |
| `eyeState(s)` | Sets Oracle status text (thinking/speaking/null) |
| `generateStructuredProgramme(d)` | Calls Claude → parses JSON → saves to Supabase → renders Programme tab |
| `getSystemPrompt()` | Builds system prompt from consultationData, memory, programme JSON, cycle phase |
| `callTSO(messages)` | Posts to `/api/proxy` with system prompt + conversation history |
| `loadProfile()` | On init: checks Supabase for existing profile → returning user OR `initNewUser()` |
| `saveCheckin(checkinData)` | Saves to `tso_checkins`, calls `loadProgressData()`, calls `unlockTab('progress')` |
| `startNewPlan()` | Wipes Supabase profile/memory, resets all state, returns to Oracle fresh |
| `switchTab(tab)` | Shows correct page, updates active tab button |

### Removed in V8 (do not try to re-add)

- `STEPS` array — the 23-question questionnaire
- `FEMALE_STEPS` array — female branch questions
- `oStep`, `oSteps` variables
- `userData` object
- `initOracle()`, `oSelect()`, `oAdvance()`, `oFinish()` — old step-by-step flow
- `buildStepList()`, `oRenderInput()`, `setProgress()` — questionnaire rendering
- `checkConflict()`, `oShowConflict()` — stress/sleep conflict logic
- `logConsultStep()` — Supabase drop-off tracking
- `$oprog` — progress bar reference
- `beginConsultation()` — form submission handler
- `buildConsultPrompt()` — form-to-prompt mapper
- `page-consult` — the consultation HTML page
- `tab-consult` — the consultation tab button

---

## System Prompt Key Rules

The system prompt is built by `getSystemPrompt()` and injected on every `callTSO()` call.

- **PROFILE:** Falls back gracefully to "Profile emerging through coaching conversation" if `consultationData` is null (new user mid-conversation).
- **PROGRAMME DATA:** Full `programmeData` JSON injected verbatim. TSO never invents exercises not in the user's plan.
- **PROGRAMME CONSISTENCY:** Never suggest structural changes before week 8 unless new injury or equipment change.
- **MEAL TIMING:** All schedules built around user's actual training schedule.
- **WHOLE FOOD FIRST:** Supplements only where evidence is strong AND gap can't be closed through food.
- **CYCLE SYNCING:** Full phase protocol for female users.
- **SCOPE BOUNDARY:** Clinical territory → acknowledge limits, direct to professional.
- **SWAP PROTOCOL:** `SWAP: [New Exercise Name]` on first response line — app intercepts this, updates programme tab, saves to Supabase.

---

## Design System (V8 — Full Light Theme)

```css
--bg: #FAFAFA;
--surface: #FFFFFF;
--surface2: #F5F5F5;
--border: #EBEBEB;
--border2: #D8D8D8;
--text: #0A0A0A;
--text2: #3C3C3C;
--muted: #6E6E73;
--muted2: #B8B8BC;
--accent: #0066FF;
--accent-light: #EBF2FF;
--green: #22C55E;
--orange: #F97316;
--red: #EF4444;
```

**Typography:**
- Body / UI: Plus Jakarta Sans (300, 400, 500, 600, 700)
- Oracle messages / briefing: Cormorant Garamond (300, 400, 600 + italics)

**Oracle message bubbles:** White background, light border, 3px border-radius top-left (speech bubble feel), Cormorant Garamond 17px.

**User message bubbles:** `#0A0A0A` background, white text, Plus Jakarta Sans 13px bold.

**Atomic orbit animation:** 3 elliptical SVG orbits. A+C clockwise, B counter-clockwise. Dark nucleus. Available in 32px (Oracle header) and 72px (splash/loading) sizes.

---

## Design Decisions & Rationale

| Decision | Rationale |
|---|---|
| Full light theme in V8 | Apple × WHOOP × Equinox. Dark was premium-feeling but created friction. Light is the default aesthetic of serious fitness apps. |
| Oracle-first, no questionnaire | TSO is an advisor, not a form. Starting with chat frames the relationship correctly from the first second. |
| Tab nav starts invisible | Clean, minimal. Nothing to confuse a new user. One thing: talk to the Oracle. |
| Progressive tab unlock | Each tab unlock represents earned context. Today = you've talked to me. Programme = I know enough to train you. Progress = you've checked in. |
| Silent tab unlocks | No fanfare, no gamification. Understated is more TSO than a celebration animation. |
| Plan offered mid-conversation | TSO decides when it knows enough — not the user clicking a button. More advisor-like. |
| Plan upload + gap analysis | Brings users with existing coaches or PT-written plans into the TSO ecosystem. Scores their plan, highlights gaps, builds credibility. |
| "Talk to Oracle →" in Programme empty state | Replaces old "Start Consultation →". No dead links. Correct redirect. |
| Returning users straight to Today | Consultation is one-time. Daily use is the Today hub. |
| Oracle messages in serif | Cormorant Garamond creates editorial authority. Feels like receiving advice, not reading a chatbot. |

---

## V8 Bug Fixes (verified this session)

1. Programme empty state "Start Consultation →" button → replaced with "Talk to Oracle →" pointing to `switchTab('oracle')`
2. Progress tab never unlocking → `saveCheckin()` now calls `unlockTab('progress')` after successful insert
3. Dead `beginConsultation()` function referencing removed `userData` → removed
4. Dead `buildConsultPrompt()` function → removed

---

## Deployment Workflow

1. Edit code in Claude chat (Cowork mode)
2. File saves directly to the mounted folder
3. Open Cursor → the file is already updated → `git add . && git commit -m "message" && git push`
4. Vercel auto-deploys on push — check dashboard for green tick

`proxy.js` and `vercel.json` rarely change — only touch if API routing changes.

---

## Monetisation Plan (discussed, not built)

- **Free tier:** Oracle access + programme generation + 2 weeks basic Today access
- **Paid ~$15–20/month:** Full Today hub, chat history, cycle syncing, nutrition tracking, progress tracking
- **Moat:** Brain Trust methodology, conversational onboarding, female physiology integration, programme consistency coaching
- **App Store:** Capacitor wrapper for fastest path; React Native for proper V2+
- **WHOOP/Eufy:** Currently screenshot-based; proper API integrations planned for paid tier

---

## Issues Backlog (from TSO-Product-Strategy-V8.pdf)

The strategy doc identified 8 issues. Status after V8:

| # | Issue | Status |
|---|---|---|
| 1 | Onboarding drop-off | ✅ **Resolved in V8** — questionnaire replaced with Oracle conversation |
| 2 | Retention | ⏳ Not yet addressed |
| 3 | Data input friction | ✅ **Partially resolved** — conversational onboarding removes form friction |
| 4 | Programme trust | ⏳ Not yet addressed |
| 5 | Security / privacy | ⏳ Not yet addressed |
| 6 | Pricing strategy | ⏳ Not yet addressed (strategy only) |
| 7 | Brain Trust credibility | ⏳ Not yet addressed |
| 8 | Technical debt | ⏳ Not yet addressed |

---

## V9 Queue (next session priorities)

Carry forward from strategy doc — issues 2, 4, 5, 7, 8 remain. Suggested order:

1. **Retention** — return nudges, weekly review prompt, streak logic
2. **Programme trust** — peer review indicators, Brain Trust citation inline, confidence scores
3. **Brain Trust credibility** — surface practitioner context more prominently
4. **Technical debt** — code review, performance, error handling
5. **Security** — authentication (proper email/password or OAuth), data ownership clarity

---

## V2 Product Roadmap (post-MVP)

- Proper authentication (email/password or OAuth)
- Two-way data binding — Oracle chat writes programme changes directly to Supabase, live re-renders
- Programme preview before acceptance — psychological commitment mechanic
- Plan archiving — store completed plans, view history
- YouTube exercise demos — curated Brain Trust video library per exercise
- WHOOP and Eufy API integrations (replace screenshot uploads)
- React Native app (Capacitor wrapper as interim)

---

## How to Resume in a New Session

1. Upload this file (`CONTEXT.md`) at the start of the chat
2. Also upload `index.html` if you want code changes
3. State what you want to build or fix
4. Claude will have full context immediately

**Key things to know before touching code:**
- The questionnaire is gone. Do not re-add it.
- `userData` does not exist. Profile data lives in `consultationData`, built from `coachingContext` via conversation.
- Tab visibility is managed by `tabsVisible` object + `unlockTab(tabId)`. Do not manipulate tab buttons directly.
- The Oracle tab is always tab index 0. Always present. Cannot be hidden.
- `generateStructuredProgramme(d)` still works the same — it takes a profile object `d` and generates JSON. It's called by `generatePlanFromConversation()`.
