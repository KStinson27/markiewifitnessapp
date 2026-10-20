# Markiewi Fitness Platform — Architecture Plan

**Status:** Approved (v3, 2026-10-20) · **Owner:** Markiewi Fitness LLC
**Built by:** Claude · **Maintained by:** contracted developer

A private coaching platform (iOS, Android, web) that replaces Trainerize for a
single business: programming, nutrition, habits, messaging, challenges, and
billing — with room to add contract coaches later.

---

## 1. Business context

| Fact | Design consequence |
|---|---|
| ~20 clients today; growth through additional coaches | Single business, but every client has an assigned coach from day one |
| 1 trainer (owner) + 2 admins; contract coaches later | Five roles: Owner, Admin, Coach, Client, Community Member |
| Mostly in-person (Michigan), some online clients in other US states | US-only; no GDPR; state consumer-health privacy laws apply |
| Booking via Calendly; group classes via Calendly + Eventbrite (public) | Booking stays external; app shows a synced class schedule and deep-links to book |
| Sells monthly coaching, pay-in-full, add-ons, small-group class packages | Stripe subscriptions + one-time payments; class package session tracker |
| Class packages currently tracked by hand | In-app package balance, auto-decremented from Calendly/Eventbrite bookings |
| Challenges must be easy to run and open to non-clients; free | Challenge-scoped habits; Community Member role; no payment for challenges |
| Mostly iPhone; some Apple Watch, some Garmin, some Oura | Apple Health integration in MVP (captures Garmin/Oura via their Apple Health sync) |
| Budget $50–100/month | Managed services on free/low tiers; target ≈ $35–60/month |

## 2. Tech stack

| Layer | Choice | Notes |
|---|---|---|
| Mobile | React Native + Expo (EAS Build/Submit/Update) | One TypeScript codebase for iOS + Android |
| Web | Next.js (React) on Cloudflare Pages / Workers | Coach/Admin dashboard and client web portal |
| UI — web | **MUI (Material UI)** + MUI X Data Grid; `@mui/icons-material` | Material Design components for the staff dashboard and client web portal |
| UI — mobile | **React Native Paper** (Material Design 3) + Material icons via `@expo/vector-icons` | MUI is web-only; Paper gives the same Material look on iOS/Android |
| Design tokens | One shared theme package (colors, type, shape) feeding both MUI and Paper | Brand colors swapped in once the brand kit arrives |
| Monorepo | Turborepo + pnpm | `apps/mobile`, `apps/web`, `packages/*` (types, validation, UI, api client) |
| Backend | Supabase: Postgres, Auth, Storage, Realtime, Edge Functions | Standard Postgres, no lock-in |
| Validation | Zod schemas shared by client and server | |
| Payments | Stripe Billing, Checkout, Customer Portal | Stripe Connect only if coach payouts are automated (Phase 9) |
| Exercise media | Short MP4/WebP loops in Supabase Storage (CDN) | Upgrade to Mux/Cloudflare Stream only if needed |
| Body map / heatmap | Rive (`@rive-app/react-webgl2` on web; React Native Rive runtime, verified in a Phase 2 spike) | Owner-licensed Fitness Visuals anatomy file |
| Charts (trends, macros, reports) | Visual style from **Orion** (Setproduct Figma kit, owner-supplied); built with MUI X Charts on web and a React Native chart library (e.g. Victory Native) on mobile | Orion is design-only; components are recreated in code to match it |
| Food data | USDA FoodData Central + Open Food Facts (barcodes); FatSecret as fallback | Local cache table; attribution per licences |
| Wearables | Apple HealthKit (MVP); Google Health Connect, Garmin Health API, Oura API (post-launch) | |
| Scheduling | Calendly API + webhooks; Eventbrite API + webhooks | Calendly webhooks require Standard plan or higher |
| Notifications | Expo Push (APNs/FCM); Resend for email | |
| Bot protection | Cloudflare Turnstile | Public challenge sign-up |
| Observability | Sentry, PostHog (free tiers) | |
| CI/CD | GitHub Actions; Cloudflare preview deploys; EAS channels | |

### Estimated monthly cost (≈ $35–60)

| Item | Cost |
|---|---|
| Supabase Pro | $25 |
| Cloudflare Pages/Workers | $0 (free tier; $5 Workers Paid if limits are reached) |
| Apple Developer ($99/yr) + domain | ≈ $10 |
| Expo EAS, Resend, Sentry, PostHog, Turnstile, food data | $0 (free tiers) |
| Storage/egress overage (photos, videos) | $0–10 |
| *Not in budget:* Stripe fees (~2.9% + 30¢; Billing ~0.7%), Calendly Standard plan, Google Play $25 one-time | |

## 3. Identity & access management

### Roles

| Role | Can | Cannot |
|---|---|---|
| **Owner** | Everything: billing configuration, refunds, exports, role management, audit log | — |
| **Admin** | Client contact info and membership *status*; class schedule; package session balances and attendance; challenge setup; Support inbox | Health data (food, photos, measurements, check-ins, workout history); coach↔client chat; refunds/billing changes; role changes; exports |
| **Coach** (contractor) | Their assigned clients' programming, nutrition, habits, check-ins, progress, chat | Other coaches' clients; business financials |
| **Client** | Own data; class schedule; package balance; challenges; billing portal | Anyone else's data |
| **Community Member** | Challenges they joined; class schedule; own profile | Coaching features; billing; anyone else's data |

### Authentication
- Supabase Auth. Clients: passkey, email magic link, Sign in with Apple, Google.
- **MFA (TOTP) mandatory** for Owner, Admin, Coach; step-up re-auth for sensitive actions.
- Sign-up is **invite-only**, except via a challenge join link, which creates a
  Community Member and is protected by email verification, Turnstile and rate limits.
- Tokens stored with `expo-secure-store` (Keychain/Keystore); short-lived access tokens.
- In-app account deletion (App Store requirement 5.1.1(v)).

### Authorization
- **Postgres Row-Level Security on every table** is the primary control.
- Role and coach assignment carried as JWT custom claims; policies reference them.
- Health data is segregated into tables Admins have no policy for.
- Policy test suite (pgTAP) runs in CI for every role × table.
- `audit_log` records reads/writes of sensitive records and privileged actions.

### Data protection
- TLS in transit; encryption at rest; private storage buckets; short-lived signed URLs for photos/documents.
- HealthKit data used only to serve the user and their coach — never marketing/analytics (App Store 5.1.2(vi), 5.1.3).
- Data minimization, consent capture, export and deletion on request (state consumer-health laws, e.g. WA MHMDA).

## 4. Billing & card safety

- Card data entered only in Stripe-hosted surfaces → **PCI DSS SAQ A**. Only Stripe IDs stored.
- Products: monthly coaching (subscription), pay-in-full (one-time, grants fixed-term access),
  add-ons (one-time or recurring), small-group class packages (one-time → session balance).
- Webhooks: signature-verified, idempotent (event-id ledger), mirrored into Postgres.
- Dunning: Stripe Smart Retries + emails; configurable grace period, then access is restricted.
- Revenue attributed to a coach on every payment (for future contractor payouts).
- **App Store:** 1:1 coaching is a person-to-person service (3.1.3(d)) and in-person
  classes are consumed outside the app (3.1.3(e)), so Stripe is permitted. US-storefront
  apps may also link out to web checkout. Default: checkout happens on the web portal;
  the app links to it.
- **Trainerize migration:** Trainerize has used direct Stripe account connections since
  2022, so existing customers/cards may already live in the owner's Stripe account
  (verify in Phase 0). Otherwise clients re-enter cards via an emailed link.

## 5. Feature set (MVP)

- **Training:** exercise library (open-licensed starter set + own videos), workout and
  program builders, offline-capable workout player, PRs, adherence.
- **Two kinds of plans:** a *program* is what the client does on their own (assigned
  on the calendar, followed in the client app). A *session plan* is only what coach and client
  do together in person; it is built from separate session templates, attached to a Calendly
  1:1 or semi-private booking, run in coach-led session mode, and never changes the
  program. Both feed the client's workout history and PRs.
- **Session planner:** week view of Calendly sessions (a client may have several per week),
  each with plan status (not planned / draft / ready / done); plan from a session template,
  by copying the client's last session, or blank. **Couples sessions** use one shared plan
  with weights set per person; each person's results go only to their own profile.
- **Session results for clients:** after a session, the client sees it under *My profile →
  Sessions*: their own sets and weights, PRs, and the coach notes marked as shared. Results
  are released only after the coach reviews them ("End & review" → share).
- **Muscle-map exercise filter:** in the program builder's exercise library, a clickable
  front/back body (female or male model) filters exercises by primary muscle, optionally
  including secondary muscles; an equivalent muscle chip list keeps it keyboard/screen-reader
  accessible. Uses the starter dataset's `primaryMuscles`/`secondaryMuscles` tags (17 groups);
  custom exercises get the same tags. Body artwork: the owner's **Fitness Visuals "Human
  Anatomy Advanced v3.0" Rive file** (female/male, front/back, 29 tappable muscles, 0–4
  intensity). Also powers a **program heatmap** (muscles shaded by planned weekly sets).
  Details: `docs/phase-0/MUSCLE_HEATMAP.md`.
- **Program templates:** reusable templates (weeks, days/week, level, equipment, tags) in a
  template library; edit, duplicate, and assign.
- **Calendar assignment:** per-client month/week calendar; drag a template onto a start date,
  choose training days, see a preview alongside the current program and Calendly class
  bookings; on overlap choose *replace remaining workouts* or *run both*; optional
  "client may move workouts within the week".
- **Client self-select:** per-template "Clients can choose this" switch; clients browse allowed
  templates in the app, pick a start date and workout days; business setting decides
  *start immediately + notify coach* or *wait for coach approval* (request queue on the
  template library page); visibility for active clients and optionally Community Members.
- **Nutrition:** per-client mode — *Habits*, *Portions*, or *Full Macros*; food search,
  barcode scan, custom foods, meal plans, meal photos, water.
- **Habits:** ongoing (no end date) or challenge-scoped; daily check-off; streaks.
- **Progress:** check-in forms; weight and body measurements (waist, hips, chest, arms,
  thighs, body fat %, custom) logged by the client, by the coach during a session, or synced
  from Apple Health (smart scales); weight card on the client Today dashboard; client
  Progress screen (trend chart vs goal, measurement changes, private progress photos with
  compare); coach Progress tab per client (stat tiles, weekly trend with goal line and hover
  values, measurement table with sparklines, photo compare, table view); per-client tracking
  settings (which metrics, weigh-in reminder cadence, *hide scale number — show trends only*).
- **Communication:** private coach chat; shared Support inbox (Owner + Admins); broadcasts; push/email.
- **Schedule:** Upcoming Classes synced from Calendly + Eventbrite with "Book" deep-links;
  1:1 sessions shown from Calendly.
- **Class flyers:** Owner/Admin upload a flyer (JPG/PNG/PDF, ≤10 MB) per class or per recurring
  class; the original is stored and a small thumbnail is generated for the class list; tapping
  opens the full flyer. Challenges can also carry a cover image/flyer.
- **Class packages:** balance per client, auto-decrement from bookings matched by email,
  admin attendance/no-show/manual adjust, low-balance alert and renewal link.
- **Challenges:** challenge-scoped habits auto-applied to all participants for the challenge's
  full duration (late joiners included); open to non-clients via link/QR; individual, team,
  or community; scoring on habits, workouts, steps, attendance, custom check-ins; leaderboard,
  feed, reactions, badges; templates and clone; bulk add; "upgrade to coaching" CTA. Free.
- **Apple Health:** steps, workouts, sleep, weight, heart rate, active energy.

## 6. Core data model

```
profiles(role) ─┬─ clients(coach_id, status, nutrition_mode, goals)
                ├─ community_members
                └─ staff(coach | admin | owner)

exercises → workout_templates → programs(phases, weeks, client_selectable, tags)
          → program_assignments(start_date, training_days, overlap_mode, source: coach | client_request)
          → scheduled_workouts → workout_logs → set_logs
program_requests(client, program, start_date, training_days, status)
session_templates(audience: 1:1 | couple | any)
          → session_plans(external_booking, date, focus, status: none | draft | ready | done)
          → session_participants(client, per-exercise load overrides)
          → session_logs(per participant) → set_logs (shared with program logs for history/PRs)
session_notes(participant | all, shared_with_client)   -- RLS: a client reads only their own logs

nutrition_targets, meal_plans, foods(cache), food_logs, water_logs
habits(scope: client | challenge) → habit_assignments → habit_logs
checkin_forms → checkin_responses; progress_photos(pose, taken_at)
body_metrics(client, metric, value, unit, measured_at, source: manual | coach | apple_health)
client_tracking_settings(metrics[], reminder_cadence, show_scale_weight, units)
conversations(kind: coach | support) → messages; notifications
challenges → challenge_teams, challenge_participants, challenge_metrics,
             challenge_points, challenge_feed_items, reactions
external_events(source: calendly | eventbrite, series_key) → external_bookings
class_flyers(series_key | event_id, original_path, thumb_path)
class_packages → package_ledger(purchase | booking | attendance | adjust | expiry)
billing_customers, products, prices, subscriptions, payments(coach_id), invoices
health_samples(source: apple_health | manual | garmin | oura)
audit_log, stripe_events, webhook_events
```

## 7. Phases

MVP = Phases 0–7. Each task ships as its own reviewable pull request.

### Phase 0 — Discovery & accounts
- P0-1 Business accounts under the LLC (D-U-N-S, Apple, Google Play, GitHub org, Supabase, Stripe, Cloudflare, domain email)
- P0-2 Trainerize export audit
- P0-3 Stripe/card migration assessment
- P0-4 Licence checks: exercise dataset, food sources, integrations
- P0-5 Wireframes for key flows
- P0-6 Brand kit (logo, colors, fonts)
- P0-7 Privacy policy, terms, waiver (attorney review)

### Phase 1 — Foundation & IAM
- P1-1 Monorepo, tooling, shared packages
- P1-2 Supabase environments (dev/staging/prod), migrations in git
- P1-3 Auth methods; mandatory staff MFA
- P1-4 Roles, coach assignment, invites
- P1-5 RLS on every table + pgTAP policy tests
- P1-6 Audit log
- P1-7 CI/CD (Actions, Cloudflare previews, EAS internal builds)
- P1-8 Sentry, PostHog, secrets management
- P1-9 App shells (client mobile, client web, staff dashboard)
- P1-10 Developer handbook & runbooks

### Phase 2 — Clients & training
- P2-1 Profiles, goals, intake, e-signed waivers
- P2-2 Exercise library (seed + uploads, business vs coach scope, muscle tags, muscle-map filter)
- P2-3 Workout builder
- P2-4 Program builder & template library (tags, duplicate, client-selectable switch)
- P2-5 Calendar assignment (month/week view, drag to start date, training days, overlap handling)
- P2-5b Client program browsing, self-start requests, coach approval queue
- P2-6 Workout player (offline sync, rest timer, PRs)
- P2-7 Adherence views

### Phase 3 — Billing
- P3-1 Products & prices from dashboard
- P3-2 Checkout: subscriptions, pay-in-full, add-ons, class packages
- P3-3 Webhooks + billing mirror
- P3-4 Entitlements, grace period, dunning emails
- P3-5 Customer Portal, invoices
- P3-6 Revenue dashboard with coach attribution
- P3-7 PCI SAQ A

### Phase 4 — Nutrition, habits & progress
- P4-1 Nutrition modes
- P4-2 Food search, barcode, custom foods, cache
- P4-3 Meal plans, photos, water
- P4-4 Habit engine (ongoing + challenge-scoped), streaks
- P4-5 Check-in forms
- P4-6 Body metrics & progress photos: client logging, coach logging, Apple Health weight import,
  Today weight card, client Progress screen, coach Progress tab with trends, tracking settings

### Phase 5 — Communication & schedule
- P5-1 Coach chat; Support inbox; broadcasts
- P5-2 Push & email notifications
- P5-3 Calendly + Eventbrite sync; Upcoming Classes; class flyer upload + thumbnails
- P5-4 Class package tracker
- P5-5 Review inbox & at-risk flags

### Phase 6 — Challenges & Apple Health
- P6-1 Challenge engine (types, metrics, scoring)
- P6-2 Community Member join flow (link/QR, Turnstile)
- P6-3 Teams, leaderboard, feed, reactions, badges
- P6-4 Templates, clone, bulk add, upgrade CTA
- P6-5 Apple HealthKit sync + manual entry

### Phase 7 — Migration & launch
- P7-1 Import clients, goals, history
- P7-2 Import exercise library & programs
- P7-3 Billing cutover
- P7-4 Security pass, backup restore drill
- P7-5 Store listings, privacy labels, review
- P7-6 Beta (3–5 clients) → full cutover → cancel Trainerize

### Post-launch
- **Phase 8 — Wearables+:** Google Health Connect; direct Garmin and Oura APIs
- **Phase 9 — Multi-coach:** coach onboarding, per-coach dashboards, contractor payouts
  (reports or Stripe Connect with 1099s), automations
