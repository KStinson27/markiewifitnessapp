# Phase 0 — Owner Checklist

Tasks only the business owner can do. Accounts must be owned by **Markiewi Fitness LLC**
(not a personal account) and use a business-domain email. Never paste passwords or API
keys into chat, GitHub, or documents — when keys are needed, they go into the hosting
providers' secret settings.

Legend: ⏱ = long lead time, start first · 💲 = costs money

## A. Start now (long lead time)

- [ ] ⏱ **D-U-N-S Number** for Markiewi Fitness LLC — free at
      [dnb.com](https://www.dnb.com/duns-number/get-a-duns.html) (look up first; you may already have one).
      Use the LLC's **exact legal name and address** as registered with the State of Michigan —
      it must match the Apple enrollment character for character. Takes ~5 business days.
- [ ] ⏱ **Business website** live on your own domain with business info (address/service area,
      hours, contact). Apple rejects social pages and placeholder sites. A simple one-page site is fine.
- [ ] **Business email** on that domain (e.g., `owner@markiewifitness.com`, `support@…`).
      Google Workspace or Microsoft 365 (~$6–7/user/mo) or your domain host's email.
- [ ] **Trainerize Support request** — send the email in `TRAINERIZE_MIGRATION.md` §3.

## B. Accounts (after the business email exists)

| # | Account | Plan | Notes |
|---|---|---|---|
| 1 | 💲 Apple Developer Program — **Organization** | $99/yr | Needs D-U-N-S, website, domain email; Apple will call you |
| 2 | 💲 Google Play Console — **Organization** | $25 once | Also uses D-U-N-S |
| 3 | GitHub **organization** (e.g., `markiewi-fitness`) | Free | Transfer this repo into it; invite the developer |
| 4 | 💲 Supabase | Pro $25/mo (prod) | Free tier is fine for dev until launch |
| 5 | Cloudflare | Free | Move the domain's DNS here (optional but simplifies setup) |
| 6 | Stripe | Existing account | See section C |
| 7 | Expo | Free | Organization account |
| 8 | Resend, Sentry, PostHog | Free | Can be created during Phase 1 |
| 9 | 💲 Calendly | Standard or higher | Needed for booking webhooks |
| 10 | Eventbrite | Existing | Need organizer account access for an API token |

Turn on **two-factor authentication** for every one of these accounts. Add the
developer/Claude-operated service accounts as team members — don't share your login.

## C. Stripe & billing audit (30 minutes)

Log into the Stripe dashboard that Trainerize is connected to and record:

- [ ] Is the Stripe account in the LLC's name? (Settings → Business details)
- [ ] Number of **Customers** and how many have a saved card
- [ ] Active **Subscriptions** — list each client, price, billing day
- [ ] **Products/Prices** in use (monthly coaching, pay-in-full, add-ons, class packages)
- [ ] Any clients paid outside Stripe (cash, Venmo, etc.)
- [ ] Do you sell any **live online group** sessions? (See research §1 — sold on web only)
- [ ] Ask your accountant whether online training for clients in other states is
      sales-taxable there (decides whether to enable Stripe Tax).

## D. Information for the build

- [ ] **Brand kit** (P0-6): logo files (SVG/PNG, light + dark versions), brand colors (hex),
      fonts, any photos you want in the app. Drop them in a shared folder.
- [ ] **Exercise videos** you already have — list or folder; naming doesn't matter.
- [ ] **Intake questionnaire** you use today (copy/paste or screenshots).
- [ ] **Check-in form** questions you ask weekly.
- [ ] **Waiver** currently signed by clients, if any.
- [ ] **Package rules**: sessions per package, expiry, cancellation/no-show policy.
- [ ] **Failed payment policy**: how many days before access is paused?
- [ ] **Challenge examples** you've run (name, length, habits, scoring) — used as templates.
- [ ] Review the **wireframes** and leave comments.
- [ ] Have an attorney review the **legal drafts** in `legal/` (Michigan-licensed).

## E. Progress tracker

| Task | Owner | Status |
|---|---|---|
| P0-1 Accounts | Owner | ☐ |
| P0-2 Trainerize export audit | Owner (guided) | ☐ |
| P0-3 Stripe/card assessment | Owner (guided) | ☐ |
| P0-4 Licence & integration research | Claude | ✅ `RESEARCH_FINDINGS.md` |
| P0-5 Wireframes | Claude → Owner review | ✅ drafted, awaiting review |
| P0-6 Brand kit | Owner | ☐ |
| P0-7 Legal drafts | Claude → attorney review | ✅ drafted, awaiting attorney |
