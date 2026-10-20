# MVP Gap Review

Status: **approved — A1–A13 and D1–D6 are in the MVP; D7 moved to Phase 11** (owner, 2026-10-20; phases rebuilt in
`ARCHITECTURE_PLAN.md` v4). Views for A1–A13 are drafted on the wireframe canvas ("MVP additions" page; files `wireframes/A*.dc.html`).

Features the approved plan does not cover yet (or covers only implicitly) that a business
leaving Trainerize will need. Grouped by recommendation.

## A. Recommend adding to the MVP

| # | Feature | Why it matters for Markiewi Fitness |
|---|---|---|
| A1 | **Client onboarding flow** — welcome, goals, PAR-Q health screening, injuries, waiver e-sign, starting weight/measurements/photos, Apple Health and notification permissions | First impression; gathers the baseline that progress tracking compares against. Without it, coaches collect this by hand. |
| A2 | **Coach-led session mode** — the trainer runs a *session plan* (separate from the client's program) on a tablet during in-person sessions and logs it there | Most clients train in person. Their history should be in the app even when they don't log it themselves. |
| A3 | **Client profile hub for coaches** — one page with Overview, Program, Nutrition, Progress, Check-ins, Messages and **private coach notes** (injuries, preferences) | Everything about a client in one place. Notes are never shown to the client. |
| A4 | **Workout history & PRs for clients** — past sessions, personal records, "last time" weights | Clients expect to see their own history and lifts going up. |
| A5 | **Exercise swaps & form-check videos** — client requests a substitution or uploads a short clip for feedback in chat | Common online-coaching need; mostly reuses chat and storage. |
| A6 | **Announcements** — post updates (class changes, holiday hours, new challenge) to all clients or groups, shown on Today | Replaces "posting upcoming classes" announcements done in Trainerize today. |
| A7 | **Membership pause / freeze** — pause billing and programming for vacation or injury, then resume | Prevents cancellations and manual refunds. |
| A8 | **Notification preferences & quiet hours**, with no health details shown on the lock screen | Privacy, and keeps reminders from feeling spammy. |
| A9 | **Units & profile settings** — lb/kg, in/cm, time zone, account deletion, data export | Required by Apple (deletion) and privacy laws (export). Online clients are in other time zones. |
| A10 | **Basic business reports** — active clients, adherence, check-in completion, churn, revenue per coach | Answers "how is the business doing?" without a spreadsheet. |
| A11 | **Email deliverability setup** — SPF/DKIM/DMARC on the domain for receipts and magic links | Without it, sign-in links go to spam. Small, but blocks login if missed. |
| A12 | **Session planner** — week of Calendly sessions with plan status, session templates, couples sessions with per-person weights | Several sessions per client per week and couples clients need planning ahead of time |
| A13 | **Session results for clients** — each client sees their own results and shared coach notes after a session | Clients keep a record of in-person work; partners never see each other's numbers |

## D. Added after the second review (approved for the MVP)

| # | Feature | Phase task |
|---|---|---|
| D1 | Coach onboarding, contractor agreement, commission rules, monthly payout report | P0-7, P8-4 |
| D2 | Coach offboarding and client handoff (history, programs and notes kept) | P8-5 · mockup `M3CoachOffboard` |
| D3 | Admin support tools: resend invite, reset MFA, account status, all audited, no health data | P6-7 · mockup `M3Support` |
| D4 | Client offboarding and data retention: 90-day read-only access, automatic export, archive | P4-8 · mockup `M3ClientOffboard` |
| D5 | Emergency contact; guardian consent and waiver for clients under 18 | P1-4, P2-1 |
| D6 | Package and cancellation rules: expiry, late-cancel window, no-show, optional late fee | P0-8, P4-7 |
| D7 | Leads list and "book a consult" link; challenge and promo-code conversion tracking (was B3) | **Phase 11 (post-launch, owner decision)** |

Also added as process steps: a Trainerize **test import** before anything is cancelled (P0-2),
and starting Meta and Google app approvals early (P0-9). Android clients' Garmin data still
waits for Google Health Connect (Phase 10); they enter steps by hand until then.

## B. Recommend shortly after launch

| # | Feature | Notes |
|---|---|---|
| B1 | Automations (welcome sequence, "haven't logged in 5 days" nudges) | Phase 11 |
| B2 | Habit and check-in templates library | Speeds up setup as coaches are added |
| B3 | ~~Referral tracking / promo codes~~ | Now D7, scheduled for Phase 11 |
| B4 | In-app help / FAQ | Cuts admin support load |
| B5 | Dark mode and larger-text accessibility pass | Material supports both; it's mostly design QA |

## C. Leave out of the MVP

| Feature | Reason |
|---|---|
| In-app video calls | Online clients can use Zoom/FaceTime via a link |
| On-demand video library / paid content | Not sold today; would also trigger Apple in-app purchase rules |
| AI meal-photo estimation, AI coaching assistants | Nice later; adds cost and review overhead |
| Native Garmin/Oura APIs | Apple Health covers iPhone users (Phase 10) |
| Multi-location / franchise features | Not needed for one business |

## Decisions needed from the owner

1. Approve A1–A11 for the MVP? (Adds roughly 2–3 weeks.)
2. Coach-led session mode (A2): will trainers use a phone, an iPad, or both on the gym floor?
3. Should clients be able to pause their membership themselves, or only by asking staff?
4. Which reports matter most at launch?
