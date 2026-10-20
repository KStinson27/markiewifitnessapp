# Trainerize Migration — Export Audit & Plan

Goal: move **client history/goals, programs, and the personal exercise library** (plus
billing) out of Trainerize with no data loss and no double-billing. Keep the Trainerize
subscription active until the Phase 7 cutover.

## 1. In-account export audit (owner, ~45 minutes)

For each item, record in the table: *Can export? Format? Where in the menu? Notes.*

| Data | Where to look | Can export? | Format | Notes |
|---|---|---|---|---|
| Client list (name, email, phone) | Clients → list → export/download | | | |
| Client goals, notes, tags | Client profile | | | |
| Body stats / measurements history | Client → Progress | | | |
| Progress photos | Client → Progress → Photos | | | |
| Workout history (sets/reps/weights) | Client → Workouts / Calendar | | | |
| Programs & workout templates | Training → Master programs / Workout library | | | |
| Personal exercise library (incl. videos) | Training → Exercises → custom | | | |
| Habits & food log history | Client → Nutrition / Habits | | | |
| Messages | Messages | | | |
| Payment plans / packages | Payments | | | |

Also try: PDF/print of a program, "share program" features, and any *Settings → Data / Account*
export option. Take screenshots of what's available — that's enough for me to plan imports.

## 2. Migration strategy by data type

| Data | Primary path | Fallback |
|---|---|---|
| Clients + contact info | Trainerize CSV → import | `templates/clients.csv` |
| Goals, notes, injuries | Copy into `templates/clients.csv` | Re-enter during onboarding call |
| Body stats history | Export if available | Starting values + Apple Health weight history |
| Progress photos | Download per client | Client re-uploads selected photos |
| Workout history | Export if available | Import **PRs and current working weights** only (`templates/client_prs.csv`) — the part that matters for programming |
| Programs | Export/PDF → `templates/programs.csv` | Rebuild in new program builder (Phase 2) |
| Exercise library + videos | Download custom videos → `templates/exercises.csv` | Re-record or reuse originals from your phone |
| Messages | Not migrated | Clients keep Trainerize access until cutover |
| Billing | Reuse existing Stripe customers & cards (research §4) | "Update card" email link |

Imports are run by an import tool in Phase 7 (P7-1/P7-2) that validates every row,
reports errors, and can be re-run safely.

## 3. Email to Trainerize Support

> **Subject:** Full data export request — account [your account email]
>
> Hello,
>
> I'm requesting a complete export of my business's data from Trainerize, including for all
> current and past clients:
>
> 1. Client profiles (contact details, goals, notes, tags)
> 2. Body stats / measurement history and progress photos
> 3. Logged workout history (exercise, sets, reps, weight, date)
> 4. All programs, workout templates and master programs I created
> 5. My custom exercise library, including uploaded videos
> 6. Habit and nutrition log history
>
> Please provide it in a machine-readable format (CSV, JSON or Excel) and let me know the
> expected timeline and any fees.
>
> Also, please confirm that my clients' payment methods and customers are stored in **my
> connected Stripe account** (not a Trainerize-held account), and let me know how to cleanly
> disconnect Stripe from Trainerize without cancelling clients' saved payment methods.
>
> Thank you,
> [Name], Markiewi Fitness LLC

## 4. Cutover sequence (Phase 7, for reference)

1. Import clients, PRs, programs, exercises into the new app (staging → production).
2. Beta: 3–5 clients use both apps for ~2 weeks.
3. For each client, on their next billing date: create the new subscription on the same Stripe
   customer/card **and** cancel the Trainerize-created subscription in the same step.
4. Invite all remaining clients; send onboarding guide (incl. Garmin/Oura → Apple Health steps).
5. Once every subscription has moved, disconnect Stripe from Trainerize and cancel Trainerize.
