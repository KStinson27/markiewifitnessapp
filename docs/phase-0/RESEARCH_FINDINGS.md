# Phase 0 — Research Findings

Verified 2026-10-20. Items marked **(re-verify)** come from secondary sources or
change often; confirm at the time of sign-up or implementation.

---

## 1. App Store payment rules (P0-4) — Stripe is allowed

Apple App Review Guidelines ([source](https://developer.apple.com/app-store/review/guidelines/)):

- **3.1.3(d) Person-to-Person Services:** "If your app enables the purchase of real-time
  person-to-person services between two individuals (for example tutoring students, medical
  consultations, real estate tours, or fitness training), you may use purchase methods other
  than in-app purchase… **One-to-few and one-to-many real-time services must use in-app purchase.**"
- **3.1.3(e) Goods and Services Outside of the App:** services "consumed outside of the app"
  *must* use purchase methods other than in-app purchase.
- **US storefront:** 3.1.1(a) and 3.1.3 state that buttons, external links and other calls to
  action toward non-IAP purchasing are permitted in **United States storefront** apps.

**What this means for Markiewi Fitness**

| Product | Rule | Payment method |
|---|---|---|
| 1:1 coaching (monthly / pay-in-full) | 3.1.3(d) | Stripe ✅ |
| In-person small-group class packages | 3.1.3(e) | Stripe ✅ |
| Add-ons (in-person or 1:1) | 3.1.3(d)/(e) | Stripe ✅ |
| *Live online group* sessions bought **inside** the app | 3.1.3(d) one-to-few → IAP | Avoid: sell on the web portal and link out (allowed in US) |
| Free challenges | No purchase | n/a |

**Decision:** all checkout happens on the web portal (Stripe Checkout); the mobile app shows
status and links out. This is compliant for every product above and keeps a single billing path.

Other guidelines that shape the build:
- **5.1.1(v):** account deletion must be available inside the app.
- **4.8:** if Google sign-in is offered, an equivalent privacy-preserving option (Sign in
  with Apple) must also be offered.
- **5.1.2(vi), 5.1.3(i–ii):** HealthKit data may not be used for marketing/advertising/data
  mining; must disclose what health data is collected; no personal health info in iCloud.
- **2.5.1:** HealthKit must be used for health/fitness purposes and integrate with the Health app.

## 2. Apple Developer organization enrollment (P0-1)

Requirements for enrolling **Markiewi Fitness LLC** as an organization
([Apple enrollment](https://developer.apple.com/programs/enroll/); vendor summaries
[Everfit](https://help.everfit.io/en/articles/5607471-white-label-solution-enrolling-in-apple-developer-program),
[Glofox](https://support.glofox.com/hc/en-us/articles/46380636064916)):

1. **Legal entity** — LLC ✅ (DBAs/trade names are not accepted).
2. **D-U-N-S Number** — free from Dun & Bradstreet; up to ~5 business days to issue, then up
   to 2 business days to reach Apple. Legal name and address must **exactly match** the
   Apple enrollment. Check for an existing number first.
3. **Public, functional website** on a domain associated with the business, showing business
   information (e.g., location, hours). Social-media pages or registrar placeholder pages are rejected.
4. **Work email on the business domain** (e.g., `name@markiewifitness.com`).
5. **Signing authority** — the enrolling person must be able to bind the LLC.
6. **Verification phone call** from Apple.
7. **Fee:** $99/year **(re-verify)**.

Google Play: organization account, $25 one-time, also requires a D-U-N-S number for organizations.

## 3. Trainerize data export (P0-2) — expect limited export

- No official bulk export of workout history or programs was found. Trainers have requested
  bulk export of workout/program progress for years on Trainerize's idea forum
  ([1](https://ideas.abcfitness.com/forums/167887-coach-trainer-abc-trainerize/suggestions/45402460-ability-to-mass-export-workout-exercise-progress-a),
  [2](https://ideas.abcfitness.com/forums/167887-coach-trainer-abc-trainerize/suggestions/18722533-graduate-a-client-export-their-data),
  [3](https://ideas.abcfitness.com/forums/167887-coach-trainer-abc-trainerize/suggestions/5437554-export-names-last-workouts-logged)).
- A third-party migration guide reports the client CSV export contains names and contact details
  only, and recommends checking your own account's export options
  ([CoachWay](https://coachway.io/articles/how-to-switch-from-trainerize/)) **(re-verify)**.
- **Plan:** do an in-account audit (see `TRAINERIZE_MIGRATION.md`), ask Trainerize Support in
  writing for a full export, and fall back to structured spreadsheet templates
  (`templates/`) for programs, exercises and goals. With ~20 clients this is manageable.

## 4. Existing payments (P0-3) — cards may already be in your Stripe account

- Trainerize retired its built-in "Trainerize Pay" gateway in 2022 and moved trainers to
  **directly connected Stripe accounts** that the trainer controls; disputes are handled in
  the trainer's own Stripe dashboard
  ([Trainerize blog](https://www.trainerize.com/blog/trainerize-update-connecting-stripe-for-a-more-powerful-payments-experience/)).
- If so, your clients' Stripe **Customers and saved payment methods already live in your
  Stripe account**, and the new app can reuse them — no card re-entry.
- Caveat: subscriptions created by Trainerize may be configured in ways specific to Trainerize.
  Plan is to create new subscriptions in our system against the **same customers/cards** and
  cancel the Trainerize ones at cutover (Phase 9), timed to avoid double-billing.
- If any cards are *not* in your account, Stripe supports account-to-account card data
  migration on request ([Stripe guide](https://stripe.com/guides/five-steps-to-accelerate-your-data-migration-to-stripe));
  fallback is an emailed "update card" link.

## 5. Exercise starter library (P0-4)

**free-exercise-db** ([github.com/yuhonas/free-exercise-db](https://github.com/yuhonas/free-exercise-db))
- **The Unlicense (public domain)** — commercial use, modification, no attribution required.
- 800+ exercises with JPG images (start/end positions).
- Fields: `id, name, force, level, mechanic, equipment, primaryMuscles, secondaryMuscles,
  instructions, category, images` (`force`, `mechanic`, `equipment` sometimes null).
- **Decision:** seed the library from this dataset; owner videos replace images over time.
  Images are stills, not videos — the owner's own clips remain the premium content.

## 6. Food data (P0-4)

| Source | Licence / terms | Use |
|---|---|---|
| USDA FoodData Central | US government work, public domain (CC0); free API key | Generic/whole foods, nutrient detail |
| Open Food Facts | **ODbL** database; images CC BY-SA. Commercial use allowed with **attribution + link to licence**, share-alike on derived *databases*, custom User-Agent required, rate limits ([support FAQ](https://support.openfoodfacts.org/help/en-gb/12-api-data-reuse/94-are-there-conditions-to-use-the-api), [docs](https://openfoodfacts.github.io/documentation/docs/Product-Opener/api/tutorials/license-be-on-the-legal-side/)) | Barcode lookup, branded foods |
| FatSecret Platform (free tier) | Attribution required **(re-verify terms)** | Fallback if coverage gaps appear |

**Design consequences:**
- Keep Open Food Facts–derived rows in their own cache table (not merged into proprietary
  tables) so share-alike obligations stay contained; show "Data from Open Food Facts (ODbL)"
  attribution on food detail screens and in the app's About/Licences page.
- Send a custom User-Agent (`MarkiewiFitness/1.0 (support@<domain>)`); cache aggressively.

## 7. Scheduling integrations (P0-4)

- **Calendly:** API available to all users; **webhooks require Standard plan or higher**
  ([Calendly help](https://calendly.com/help/calendly-api-overview)). Owner must confirm plan.
- **Eventbrite:** REST API v3 with private token (single account) or OAuth. Organization
  events list and per-event attendee list (with check-in status); webhooks for orders and
  check-ins ([integration overview](https://docs.nexla.com/user-guides/connectors/eventbrite_api))
  **(re-verify against official Eventbrite docs during Phase 8)**.
- Matching bookings to clients is by email address; unmatched bookings appear in an admin
  review queue.

## 8. Wearables (P0-4)

**Garmin → Apple Health** (one-way). Data types: active energy, flights climbed, heart rate,
resting energy, sleep, steps, walking/running distance, weight, body fat/BMI, workouts (no GPS
tracks; only high/low HR for activities)
([summary](https://the5krunner.com/2025/06/20/garmin-to-add-health-connect-support-to-apple/),
[Apple Community](https://discussions-kr-prz.apple.com/thread/251525307)).
Caveat: Garmin may only push data when Garmin Connect is open; delays are reported.

**Oura → Apple Health.** Active energy, heart rate, height, mindful minutes, respiratory rate,
sleep (workouts/weight per secondary sources); Readiness/Sleep scores stay in Oura
([Oura support](https://support.ouraring.com/hc/en-us/articles/360025438734)).

**Conclusion:** Apple HealthKit in the MVP covers Apple Watch, Garmin and Oura users on
iPhone for steps, workouts, sleep, heart rate and weight — sufficient for challenges.
Onboarding includes "enable Garmin/Oura → Apple Health" instructions and a reminder that
Garmin users should open Garmin Connect to sync before a challenge deadline.
Direct Garmin Health API (requires business application/approval) and Oura API are Phase 10.

## 9. Hosting & budget (re-verify at sign-up)

- **Supabase Pro** ($25/mo): includes generous MAU, database, storage and egress quotas plus
  daily backups; free projects pause after inactivity, so production must be on Pro.
- **Cloudflare Pages/Workers** free tier permits commercial use; Vercel's Hobby tier does not.
- Expo EAS free tier is sufficient for low build volume.
