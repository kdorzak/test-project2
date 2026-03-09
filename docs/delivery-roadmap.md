# Delivery Roadmap (Milestones + Timeline)

This roadmap turns the project plan into a delivery sequence with estimated timelines.

## Assumptions (for the estimates below)
- **App tech:** cross-platform (Flutter)
- **Step source:** iOS **HealthKit** and Android **Health Connect** (or Google Fit where HC is unavailable)
- **Leaderboard windows:** **Daily** leaderboard for MVP
- **Backend:** **Firebase** (Auth + Firestore + Cloud Functions)
- **Team:** 2 engineers (1 mobile-focused, 1 backend/full-stack) + part-time design

If you choose **native apps** or a **custom backend**, see “Alternate estimates” below.

---

## Milestones (with exit criteria)

### M0 — Product decisions + UX baseline (0.5–1 week)
**Deliverables**
- MVP spec locked: leaderboard window, tie-breaking, pagination rules
- Wireframes for onboarding/home/leaderboard/profile
- Copy for permissions + privacy

**Exit criteria**
- Decisions are documented
- Screens and flows are agreed

---

### M1 — Repo bootstrap + CI + environments (0.5–1 week)
**Deliverables**
- Monorepo structure (e.g., `apps/`, `services/`, `docs/`)
- CI for lint/test/build
- Dev/Staging/Prod configuration approach (separate Firebase projects)

**Exit criteria**
- One-command local run for app
- CI builds on every PR

---

### M2 — Identity + user profile (0.5–1.5 weeks)
**Deliverables**
- Auth: Apple Sign-In (iOS) + Google Sign-In
- User profile document (display name, avatar url, privacy flag)
- Profile screen (view/edit)

**Exit criteria**
- User can sign in/out, edit profile, and data persists

---

### M3 — Step reading + local aggregation (1.5–3 weeks)
**Deliverables**
- iOS step reads (HealthKit) for “today” and history (date range)
- Android step reads (Health Connect / Fit) for “today” and history
- Local persistence for **daily totals**
- Timezone-aware day boundary logic

**Exit criteria**
- Steps displayed correctly for today
- History matches OS health app totals for the same period

---

### M4 — Sync to backend (1–2 weeks)
**Deliverables**
- `POST /steps/daily` equivalent (Firebase callable function / HTTP function)
- Idempotent daily upsert (by `userId + date`)
- Background sync mechanism (WorkManager on Android; periodic reconcile on open for iOS)
- Basic server validation and rate limits

**Exit criteria**
- Daily totals consistently appear in backend
- Resyncs do not duplicate or corrupt data

---

### M5 — Leaderboard (1–2 weeks)
**Deliverables**
- Leaderboard computation strategy:
  - simplest: query + indexed ordering (if feasible)
  - or materialized leaderboard documents updated by Cloud Functions
- Endpoints:
  - get top N
  - get “my rank”
  - cursor pagination
- Leaderboard UI: top list + “my rank” card

**Exit criteria**
- Leaderboard loads fast and consistently
- User sees their rank even when not in top N

---

### M6 — Hardening: anti-cheat, privacy, reliability (1–2 weeks)
**Deliverables**
- Sanity checks (max steps/day; spike detection thresholds)
- Privacy toggle (hide from leaderboard) if required
- Improved offline UX (sync status)
- Logging, error tracking

**Exit criteria**
- Clear handling for denied permissions/offline
- Basic abuse controls in place

---

### M7 — QA, store readiness, release (1–2.5 weeks)
**Deliverables**
- Test coverage for day boundary + sync idempotency
- Manual device matrix validation
- App Store / Play Store metadata + health data declarations
- Privacy policy + account deletion flow
- Release builds to TestFlight + Play Internal Testing → production

**Exit criteria**
- Store submissions accepted
- Release checklist completed

---

## Timeline summary (under the assumptions)
- **Total:** ~6–13 weeks
  - Faster end: narrow MVP (daily only, minimal profile, Firebase)
  - Slower end: multiple leaderboard windows, privacy modes, more QA devices

---

## Critical path dependencies
- Health permissions + step access (can block step counting entirely)
- Day-boundary/timezone correctness (impacts rankings and trust)
- Backend write idempotency (prevents double-counting)
- Store policy compliance for health-related data use

---

## Alternate estimates (if you choose different options)

### Native apps (Swift + Kotlin) instead of Flutter
- Add **+30–70%** engineering time overall (two UI implementations; duplicated logic).
- Reduce risk on OS edge cases and background constraints.

### Custom backend (API + Postgres) instead of Firebase
- Add **+1.5–4 weeks** depending on infra, auth, and operational requirements.
- Gain:
  - more control over leaderboard materialization, caching, analytics
  - portability and vendor independence

### Multiple leaderboard windows (daily + weekly + all-time)
- Add **+0.5–2 weeks** for schema + caching + UI complexity.

---

## Suggested release stages
- **Alpha (internal):** step counting accuracy + basic sync
- **Beta (limited):** leaderboard correctness + performance
- **GA:** store release after compliance + QA matrix pass
