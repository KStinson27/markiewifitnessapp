# Markiewi Fitness Platform — Architecture Plan

**Status:** Approved (v3, 2026-10-20) · MVP scope expanded and phases rebuilt (v4, 2026-10-20) · **Owner:** Markiewi Fitness LLC
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
| Payments | Stripe Billing, Checkout, Customer Portal | Stripe Connect only if coach payouts are automated (Phase 11) |
| Exercise media | Short MP4/WebP loops in Supabase Storage (CDN) | Upgrade to Mux/Cloudflare Stream only if needed |
| Body map / heatmap | Rive (`@rive-app/react-webgl2` on web; React Native Rive runtime, verified in a Phase 2 spike) | Owner-licensed Fitness Visuals anatomy file |
| Charts (trends, macros, reports) | MUI X Charts on web; a React Native chart library (e.g. Victory Native) on mobile | Orion (Setproduct) styling explored and shelved for now |
| Colour roles | Teal = brand, main actions, navigation · Blue `#2A78D6` = charts & progress data · Violet `#6250D6` = challenges & community · Orange `#EB6834` = nutrition (carbs) · Amber = personal records · Macros: protein blue, carbs orange, fat green `#1BAF7A` | Shared theme tokens for MUI and React Native Paper |
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
- **Coach calendar & to-dos:** one Day/Week/Month calendar per coach combining Calendly 1:1 and
  couples sessions (with session-plan status), classes from Calendly/Eventbrite (with bookings),
  and the coach's own to-dos (all-day or timed, optionally linked to a client, with reminders);
  filters per type; a to-do list beside it with quick add and done/undo. **Admins can view coach
  calendars** (names, times, event types, class bookings; never session-plan contents or health
  data) and to-dos marked "visible to admins" (on by default).
- **Google Calendar sync (per coach, optional):** OAuth connect from the calendar page. The app
  writes to-dos and app-only events to a dedicated "Markiewi Fitness" Google calendar
  (`calendar.events` scope) and reads the coach's other calendars as free/busy only, shown as
  grey "Busy" blocks. Calendly bookings are not re-sent (Calendly already writes them to Google);
  events are de-duplicated by Calendly event ID. Changes flow both ways for synced to-dos via
  Google push notifications (watch channels) handled by an Edge Function.
- **Social media planner (admin role):** a 4-week content calendar of posts for Instagram
  (feed and Stories) and Facebook, shown next to classes and challenges. Statuses: idea → draft →
  owner review → scheduled → posted. The post editor has the caption, a character count,
  hashtag sets, and media from the flyer library or auto-made challenge cards. "Ideas from the
  app" suggests posts: classes with no promo post, challenge launches, and client spotlights.
- **Approval rules (owner only):** auto-approve or review, set per post type: class promos and
  reminders, challenge announcements, free Eventbrite listings, and general posts. **Client
  spotlights and paid Eventbrite events always need the owner.** Auto-approved posts still show
  on the owner's dashboard and can be pulled back until they go out.
- **Client content needs consent:** a client must opt in to being featured and share the photo
  and quote themselves. Admins never see progress photos, body metrics or other health data.
- **Instagram publishing:** through the Instagram Graph API (Content Publishing) for a
  professional (Business) Instagram account. Media must be hosted at a public URL (Supabase
  Storage signed URL), and publishing has two steps: create a media container, then publish it.
  This needs Meta app review for `instagram_business_content_publish` plus Meta Business
  Verification. Until that approval arrives, the fallback is a reminder at the scheduled time
  with copy-caption and download-image buttons. Story publishing support should be confirmed
  in a spike.
- **Create classes and events in the app, publish to Eventbrite:** the "New class or event"
  form (one-time event or weekly class) has toggles for the app's Classes tab (always on),
  Eventbrite, and auto-drafted Instagram posts (go-live post, a story 2 days before, and a
  last-call story if spots remain). Eventbrite API v3 calls, in order: create the event under
  the organization → add a ticket class (free, or paid with a cost) → upload the flyer as the
  banner → publish. Weekly classes become Eventbrite series dates 8 weeks ahead (series API
  to be confirmed in a spike). Bookings come back through Eventbrite webhooks
  (`order.placed`, `attendee.updated`), with a 15-minute poll as backup. Edits and
  cancellations in the app update Eventbrite. Calendly classes are still set up in Calendly.
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

- **Coaches (contractors):** coach onboarding (invite, MFA, signed contractor agreement,
  profile and bio); per-coach commission rules (percent or flat amount per product); a monthly
  payout report per coach for the accountant (paid outside the app at first); **coach
  offboarding** (owner only, 4 steps): move each client to a new coach (history, programs and
  chat always move; private notes and an intro message optional), handle open items (Calendly
  sessions, check-ins, unreviewed session results, draft programs, to-dos), end access now or
  at a set date (sign-in, devices, Google Calendar link), keep the coach's exercises and
  templates in the business library, and send a final payout report to the accountant.
- **Admin support tools — "Account help" (no health data):** look up a client or community
  member and see account status (active, restricted, invited, locked), billing status without
  amounts, last sign-in and device, app version, sign-in methods and email delivery (e.g. a
  bounced invite). Actions: send a sign-in link, unlock after failed sign-ins, resend an invite,
  resend Stripe's card-update link, send an app-update reminder or Apple Health setup guide.
  Removing a passkey for a lost phone, signing out everywhere and changing an email first
  require an **identity check using contact details already on file** (a code to the email on
  file, or a call to the phone on file). Two-step code resets for staff accounts are
  owner-only. Every action is written to the audit log.
- **Client offboarding & data retention (owner only):** end at the end of the billing period or
  today with a prorated Stripe refund; private reason; optional pause offer instead; unused class
  credits stay bookable until they expire; the client can stay on as a Community Member for free
  challenges; optional exit survey. The client keeps read-only access to
  their history for 90 days, get an automatic data export by email, and then their account
  is archived. Health data is deleted after the retention period the owner sets (default
  3 years for liability), sooner on request. Former clients can be re-activated.
- **Safety in onboarding:** emergency contact (shown to the coach in session mode); date of
  birth, and for clients under 18 a parent or guardian account with guardian consent and a
  guardian-signed waiver.
- **Package & cancellation rules:** per package: sessions included, expiry, late-cancel window
  (e.g. 12 h), whether a late cancel or no-show uses a credit, and an optional late fee
  charged through Stripe with owner approval.
- **Leads & consults (post-launch, Phase 11):** challenge members, Eventbrite attendees and website sign-ups who are
  not clients appear in a simple lead list (source, status: new / contacted / consult booked /
  client / not now) with a "book a consult" Calendly link; converting a lead creates the client
  invite. Challenge and promo-code tracking shows which challenges bring in clients.
- **Already designed (A1–A13):** client onboarding, coach-led session mode, client profile hub
  and private notes, workout history and PRs, swaps and form checks, announcements, membership
  pause, notification settings, units and account settings, business reports, email domain
  setup, session planner, session results. See `docs/phase-0/MVP_GAP_REVIEW.md`.

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
coach_tasks(coach, title, due_date, due_time?, all_day, client?, reminder_at?, done_at,
            visible_to_admins, google_event_id?)
google_calendar_links(coach, google_account, calendar_id, sync_token, watch_channel, expires_at)  -- tokens encrypted
social_posts(title, platforms[], scheduled_at, status: idea | draft | review | scheduled | posted,
             caption, media[], link_note, linked_class? | linked_challenge?, featured_client?,
             approved_by?, posted_url?)
marketing_consents(client, granted_at, revoked_at?, shared_media[], quote)  -- client-owned; revoking hides drafts
approval_rules(post_type: class | challenge | event_free | general, auto_approve)  -- owner-only RLS; spotlight & paid events fixed to review
studio_events(kind: once | weekly, name, starts_at, capacity, price_cents?, flyer, description,
              eventbrite_event_id?, eventbrite_series_id?, status: draft | review | live | cancelled)
meta_links(instagram_account_id, page_id, token_expires_at)  -- tokens encrypted
coach_profiles(status: invited | active | offboarded, agreement_signed_at, bio)
commission_rules(coach, product?, percent | flat_cents) → payout_reports(coach, month, lines)
client_handoffs(client, from_coach, to_coach, at)
emergency_contacts(client, name, phone, relation); guardians(client, guardian_profile, consent_at)
package_policies(package, sessions, expires_days, late_cancel_hours, late_cancel_uses_credit,
                 no_show_uses_credit, late_fee_cents?)
leads(name, email, source: challenge | eventbrite | website | instagram, status, notes, client?)  -- Phase 11
retention_settings(health_data_years, readonly_days); data_exports(client, requested_at, file)

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

MVP = Phases 0–9 (rebuilt in v4 so every feature designed so far is in the MVP). Each task
ships as its own reviewable pull request. **Beta checkpoint:** after Phase 7, 3–5 clients move
off Trainerize while Phases 8–9 are built, so the core is proven before full cutover.

### Phase 0 — Discovery & accounts
- P0-1 Business accounts under the LLC (D-U-N-S, Apple, Google Play, GitHub org, Supabase, Stripe, Cloudflare, domain email)
- P0-2 Trainerize export audit, including a **test import** of 2–3 clients before cancelling anything
- P0-3 Stripe/card migration assessment
- P0-4 Licence checks: exercise dataset, food sources, integrations
- P0-5 Wireframes for key flows
- P0-6 Brand kit (logo, colors, fonts)
- P0-7 Privacy policy, terms, waiver, guardian waiver, coach contractor agreement (attorney review)
- P0-8 Owner policies: package and cancellation rules, data retention, coach commission
  structure, whether minors are trained
- P0-9 Start long-lead approvals: Meta app review and Business Verification, Google OAuth
  verification (both need the live privacy policy)

### Phase 1 — Foundation & IAM
- P1-1 Monorepo, tooling, shared packages
- P1-2 Supabase environments (dev/staging/prod), migrations in git
- P1-3 Auth methods; mandatory staff MFA
- P1-4 Roles, coach assignment, invites; guardian accounts for minors
- P1-5 RLS on every table + pgTAP policy tests
- P1-6 Audit log
- P1-7 CI/CD (Actions, Cloudflare previews, EAS internal builds)
- P1-8 Sentry, PostHog, secrets management
- P1-9 App shells (client mobile, client web, staff dashboard)
- P1-10 Email domain setup: SPF, DKIM, DMARC for sign-in links and receipts (A11)
- P1-11 Developer handbook & runbooks

### Phase 2 — Clients & training
- P2-1 Client onboarding (A1): goals, PAR-Q, injuries, emergency contact, date of birth and
  guardian consent, e-signed waivers, baseline weight/measurements/photos, permissions
- P2-2 Exercise library (seed + uploads, business vs coach scope, muscle tags, muscle-map filter)
- P2-3 Workout builder
- P2-4 Program builder & template library (tags, duplicate, client-selectable switch, program heatmap)
- P2-5 Calendar assignment (month/week view, drag to start date, training days, overlap handling)
- P2-5b Client program browsing, self-start requests, coach approval queue
- P2-6 Workout player (offline sync, rest timer, PRs)
- P2-7 Workout history, PRs and "last time" weights for clients (A4)
- P2-8 Exercise swaps & form-check videos (A5)
- P2-9 Client profile hub with private coach notes (A3); adherence views
- P2-10 Units & account settings, account deletion, data export (A9)

### Phase 3 — In-person sessions & calendar
- P3-1 Session templates and session planner (A12), including couples sessions
- P3-2 Coach-led session mode on tablet (A2) with goals, emergency contact and inline logging
- P3-3 Session review and results shared with clients (A13)
- P3-4 Coach calendar & to-dos; admin view of coach calendars
- P3-5 Google Calendar sync (dedicated calendar, free/busy, watch channels)

### Phase 4 — Billing & packages
- P4-1 Products & prices from dashboard
- P4-2 Checkout: subscriptions, pay-in-full, add-ons, class packages
- P4-3 Webhooks + billing mirror
- P4-4 Entitlements, grace period, dunning emails
- P4-5 Customer Portal, invoices
- P4-6 Membership pause / freeze (A7)
- P4-7 Package & cancellation rules (expiry, late cancel, no-show, optional late fee)
- P4-8 Client offboarding: read-only period, automatic export, archive, retention clean-up job
- P4-9 PCI SAQ A

### Phase 5 — Nutrition, habits & progress
- P5-1 Nutrition modes
- P5-2 Food search, barcode, custom foods, cache
- P5-3 Meal plans, photos, water
- P5-4 Habit engine (ongoing + challenge-scoped), streaks
- P5-5 Check-in forms
- P5-6 Body metrics & progress photos: client logging, coach logging, Apple Health weight import,
  Today weight card, client Progress screen, coach Progress tab with trends, tracking settings

### Phase 6 — Communication, schedule & support
- P6-1 Coach chat; Support inbox
- P6-2 Announcements to all clients or groups (A6)
- P6-3 Push & email notifications; notification settings and quiet hours (A8)
- P6-4 Calendly + Eventbrite sync; Upcoming Classes; class flyer upload + thumbnails
- P6-5 Class package tracker (uses the P4-7 rules)
- P6-6 Review inbox & at-risk flags
- P6-7 Admin support tools: resend invite, reset MFA, account status (no health data)

### Phase 7 — Challenges & Apple Health
- P7-1 Challenge engine (types, metrics, scoring)
- P7-2 Community Member join flow (link/QR, Turnstile)
- P7-3 Teams, leaderboard, feed, reactions, badges
- P7-4 Templates, clone, bulk add, upgrade CTA
- P7-5 Apple HealthKit sync + manual entry
- P7-6 **Beta checkpoint:** import the beta clients, 3–5 clients move off Trainerize

### Phase 8 — Marketing, events, coaches & reports
- P8-1 Social media planner, approval rules, client "feature me" consent
- P8-2 Instagram Graph API publishing (reminder fallback until Meta approves)
- P8-3 New class/event form → Eventbrite publish (event, ticket class, banner, weekly series),
  booking webhooks, auto-drafted Instagram posts
- P8-4 Coach onboarding, contractor agreement e-sign, commission rules, monthly payout report
- P8-5 Coach offboarding and client handoff
- P8-6 Business reports (A10): active clients, adherence, churn, revenue per coach

### Phase 9 — Migration & launch
- P9-1 Import all clients, goals, history
- P9-2 Import exercise library & programs
- P9-3 Billing cutover
- P9-4 Security pass, backup restore drill
- P9-5 Store listings, privacy labels, review
- P9-6 Full cutover → cancel Trainerize

### Post-launch
- **Phase 10 — Wearables+:** Google Health Connect (Android clients' Garmin data; until then
  they enter steps by hand); direct Garmin and Oura APIs
- **Phase 11 — Leads, automations & polish:** leads list and "book a consult" link with
  promo-code and challenge conversion tracking (D7); welcome sequences and nudges (B1), habit and check-in
  template library (B2), in-app help/FAQ (B4), dark mode and large-text pass (B5);
  Stripe Connect payouts with 1099s if coach payouts outgrow the monthly report
