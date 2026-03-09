# Step Counter + Global Ranking — Project Task Plan (Android + iOS)

## Repo status
- Current repository contains only `README.md` (“Simple Step Counter App”).
- No mobile app code, backend, or infrastructure exists yet.

---

## 1) MVP scope

### Core features
- **Step counting** on-device (battery-conscious, background-friendly).
- **Daily totals** (and optionally week/month history).
- **Global ranking / leaderboard**.
- **User accounts** (ranking tied to a person, not just a device).
- **Sync** (device → backend) to keep ranking consistent.

### MVP screens
- Onboarding (account + permissions)
- Home (today’s steps + goal)
- Leaderboard (global rank)
- Profile / Settings

### MVP non-functional requirements
- Works offline (local storage + later sync)
- Robust around timezones/day boundaries
- Basic anti-cheat
- Clear privacy messaging and store compliance

---

## 2) Required decisions (pick 1 option per section)

### A. App technology
1. **Flutter (cross-platform)**
   - Pros: single codebase, consistent UI, faster iteration.
   - Cons: still requires platform integrations for HealthKit/Health Connect; background behavior edge cases.
2. **React Native (cross-platform)**
   - Pros: single codebase, large ecosystem.
   - Cons: native module complexity; background constraints still require native work.
3. **Native (Swift + Kotlin)**
   - Pros: best OS integration, fewer plugin risks, simplest access to health APIs.
   - Cons: two codebases.

**Default recommendation for a small team:** Flutter.

### B. Step data source (“source of truth”)
1. **OS health platform (recommended)**
   - iOS: HealthKit (+ optional CoreMotion for live updates)
   - Android: Health Connect (preferred going forward) or Google Fit
   - Pros: better accuracy, merges phone + wearables, typically lower battery.
   - Cons: permissions + platform policy requirements.
2. **Raw sensors (accelerometer-based)**
   - Pros: more direct control.
   - Cons: accuracy + battery + spoofing risk.

### C. Leaderboard window
Choose one, or explicitly support multiple:
- **Daily** (per user timezone)
- **Weekly**
- **Monthly**
- **All-time**

### D. Authentication
- **Apple + Google sign-in** (common for consumer apps)
- Email/password
- Anonymous + optional upgrade
- Enterprise SSO (if applicable)

### E. Backend approach
1. **Firebase** (fastest MVP)
   - Auth + Firestore + Cloud Functions
2. **Custom backend** (more control)
   - API (e.g., FastAPI/NestJS/Go) + Postgres + Redis (optional)

---

## 3) Recommended architecture (baseline)

### Mobile
- Read steps from OS APIs.
- Store **daily aggregates** locally.
- Periodically sync daily totals to backend with **idempotent upserts**.

### Backend
- Store user profile and **daily step totals**.
- Provide leaderboard reads (top N + “my rank”).
- Maintain a cached/materialized leaderboard for performance.

### Ranking model (suggested)
- Store only aggregates (per day) rather than raw motion events.
- Leaderboard computed from server-side aggregates for each window.

---

## 4) Workstreams and tasks

## 4.1 Product & UX
- Define acceptance criteria:
  - step source, update frequency, offline behavior
  - leaderboard windows, tie-breaking, pagination
- Wireframes for:
  - onboarding (auth + permissions)
  - home (steps + goal)
  - leaderboard (top list + my rank)
  - profile/settings
- Define user privacy controls:
  - hide from leaderboard (optional)
  - display name and avatar rules

## 4.2 Mobile — Step counting integration

### iOS (HealthKit / CoreMotion)
- Implement permission flow:
  - HealthKit authorization screen
  - clear rationale text
- Implement step reads:
  - today’s total
  - history (for weekly/monthly leaderboards)
- Implement day-boundary logic:
  - timezone aware “local day” calculations
  - handle timezone changes
- Background strategy:
  - periodic refresh + reconcile on app open

### Android (Health Connect / Google Fit)
- Implement permissions:
  - runtime permissions + Health Connect permission flow
- Implement step reads by date range
- Background strategy:
  - WorkManager periodic sync + reconcile on app open
- Validate on OEMs with aggressive background limits

### Shared (both platforms)
- Local persistence schema:
  - `daily_steps(date, steps, source, updated_at, last_synced_at, version/hash)`
- Reconciliation strategy:
  - wearables/health platforms sometimes adjust totals retroactively
  - implement re-read and overwrite by version

## 4.3 Mobile — Leaderboard & profile
- Leaderboard UI:
  - global top N (cursor pagination)
  - “my rank” view even if not in top N
- Profile & settings:
  - display name/avatar edit
  - goal settings
  - privacy toggle (if supported)
- Error handling:
  - permissions denied → graceful degraded UX
  - offline mode + sync status

## 4.4 Backend — Core services

### Data model (example)
- `users(id, display_name, avatar_url, created_at)`
- `daily_steps(user_id, date, steps, source, updated_at)` with unique constraint `(user_id, date)`
- `leaderboard_cache(window, bucket_date, user_id, steps, rank)` (optional materialization)

### API contract (example)
- `POST /steps/daily` — upsert a day’s total (idempotent)
- `GET /leaderboard?window=daily&date=YYYY-MM-DD&cursor=...`
- `GET /me/rank?window=daily&date=YYYY-MM-DD`

### Backend tasks
- Implement auth and request validation
- Idempotent writes + conflict handling
- Leaderboard calculation:
  - incremental update on write, or scheduled recompute
- Observability:
  - structured logs + error reporting
- Rate limits

## 4.5 Anti-cheat (MVP level)
- Prefer OS-provided totals (HealthKit / Health Connect).
- Server sanity checks:
  - configurable max steps/day
  - spike detection vs. baseline (optional)
- Store only aggregates (privacy-first).

## 4.6 Privacy, legal, store readiness
- Privacy policy:
  - what is collected (daily step totals, account ID, display name)
  - purpose (leaderboards)
  - retention + deletion
- Store compliance:
  - iOS HealthKit usage declarations
  - Android health data declarations
  - permission rationale screens

## 4.7 Testing
- Unit:
  - date boundary + timezone logic
  - sync idempotency
- Integration:
  - mock health API reads
  - backend endpoints
- E2E:
  - onboarding → steps → sync → leaderboard rank
- Device matrix:
  - iOS: latest + previous major
  - Android: 2–3 OS versions + at least one aggressive-OEM device

## 4.8 CI/CD & environments
- Environments: dev / staging / prod
- CI pipelines:
  - lint + tests
  - signed builds to TestFlight / Play Internal Testing
- Secrets management:
  - API keys, signing keys, backend credentials

---

## 5) “First 10 tasks” (lowest-risk execution order)
1. Choose app technology (Flutter / RN / Native).
2. Choose step source (HealthKit + Health Connect vs sensors).
3. Choose leaderboard window(s) (daily/weekly/monthly/all-time).
4. Choose backend approach (Firebase vs custom).
5. Draft API contract + data model for daily step aggregates and leaderboard.
6. Implement onboarding: auth + permissions.
7. Implement local daily aggregation on iOS and Android.
8. Implement sync to backend (idempotent upserts) + server validation.
9. Implement leaderboard read endpoints + UI (top N + my rank).
10. Add privacy policy + store compliance checklist.
