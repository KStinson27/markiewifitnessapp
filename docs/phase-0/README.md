# Phase 0 — Discovery & Accounts

| Task | Deliverable | Status |
|---|---|---|
| P0-1 Business accounts | [`OWNER_CHECKLIST.md`](OWNER_CHECKLIST.md) §A–B | Waiting on owner |
| P0-2 Trainerize export audit | [`TRAINERIZE_MIGRATION.md`](TRAINERIZE_MIGRATION.md), [`templates/`](templates/) | Waiting on owner |
| P0-3 Stripe/card assessment | [`OWNER_CHECKLIST.md`](OWNER_CHECKLIST.md) §C, [`RESEARCH_FINDINGS.md`](RESEARCH_FINDINGS.md) §4 | Waiting on owner |
| P0-4 Licence & integration research | [`RESEARCH_FINDINGS.md`](RESEARCH_FINDINGS.md) | Done |
| P0-5 Wireframes | [`wireframes/`](wireframes/) — interactive canvas shared with the owner for comments | Awaiting owner review |
| P0-6 Brand kit | [`OWNER_CHECKLIST.md`](OWNER_CHECKLIST.md) §D | Waiting on owner |
| P0-7 Legal drafts | [`legal/`](legal/) | Awaiting attorney review |

Phase 0 is complete when every row is done; Phase 1 (foundation & IAM) can start in
parallel once the GitHub organization and Supabase account exist.

## Key findings that changed the plan
- **Small-group *online* sessions sold inside the iOS app would require Apple in-app purchase**
  (3.1.3(d) "one-to-few"). Resolved by keeping all checkout on the web portal — US-storefront
  apps may link out. In-person classes and 1:1 coaching may use Stripe either way.
- **Apple organization enrollment needs a public business website and a domain email**, in
  addition to the D-U-N-S number.
- **Trainerize has connected trainers' own Stripe accounts since 2022**, so client cards are
  likely already in the owner's Stripe account and can be reused without re-entry.
- **Trainerize offers little bulk export**; migration focuses on clients, goals, PRs/working
  weights, programs and custom exercises via spreadsheet templates.

## Wireframes
Two versions, both with a placeholder teal accent until the brand kit arrives. Files in
`wireframes/` are the canvas source (`.dc.html`).

- **v1 low-fi** — the files without a prefix.
- **v2 Material UI** (`M3*.dc.html`) — owner feedback: Material look, icons, and class flyers.
  Mobile screens follow Material Design 3 (React Native Paper); staff web pages follow MUI defaults.
  Adds flyer thumbnails on the client Classes and Today screens, and flyer upload on the admin page.

| Screen | File |
|---|---|
| Client · Today | `Main.dc.html` |
| Client · Workout player | `WorkoutPlayer.dc.html` |
| Client · Food log | `FoodLog.dc.html` |
| Client · Classes & package balance | `Classes.dc.html` |
| Client · Challenge | `Challenge.dc.html` |
| Public · Join challenge | `ChallengeJoin.dc.html` |
| Coach · Dashboard | `CoachDashboard.dc.html` |
| Staff · Challenge builder | `ChallengeBuilder.dc.html` |
| Coach · Program builder | `ProgramBuilder.dc.html` |
| Admin · Schedule, packages & support | `AdminView.dc.html` |
