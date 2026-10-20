# Muscle Map & Heatmap — Integration Notes

Source asset: **Human Anatomy Advanced v3.0** (`human_anatomy_advanced_v3.0.riv`, Rive) by
Fitness Visuals, supplied by the owner with sample apps (Next.js, SwiftUI, Kotlin, Flutter).

> **Licence:** confirm the purchase licence covers use in a commercial app distributed through
> the App Store / Play Store and storage in our private repository. The `.riv` file is **not
> committed** until that is confirmed. The wireframe canvas uses preview renders only.

## What the file provides
- Four artboards: `Male-Front`, `Male-Back`, `Female-Front`, `Female-Back`.
- State machine `State Machine 1` on each artboard; emits a Rive event named after the muscle
  when a muscle is tapped/clicked.
- A default ViewModel with one nested ViewModel per muscle (29 muscles):
  `<muscle>/intensity` (0–4) and `<muscle>/palette/{baseColor, colorLevel1..4}`.

## How we use it
| Feature | Mode | Data driven into the file |
|---|---|---|
| Program builder → exercise library "Find by muscle" | Tap a muscle → filter exercises by primary (optionally secondary) muscle group | Selected group = intensity 4, others 0 |
| Program builder "Program heatmap" | Shade each muscle by planned weekly sets for the selected week | Weekly sets per group binned to 0–4 |
| Later: client Progress / coach client hub | Muscles trained this week from logged sets | Same binning from logs |

Palette: a single-hue teal ramp (sequential, light → dark) instead of the sample's multi-hue
"Classic Heat", so intensity reads as "more" and stays colour-blind safe:
`#D3D9D7` (0), `#B8E1DA`, `#6FC2B4`, `#1F8F7E`, `#00504A` (4). Swapped for brand colours later.

## Muscle → group mapping (14 groups)
| Group | Rive muscles | Exercise-library tags (`free-exercise-db`) |
|---|---|---|
| Chest | pectoralisMajor | chest |
| Shoulders | deltoids, posteriorDeltoid | shoulders |
| Back | latissimusDorsi, teresMajor, erectorSpinae | lats, middle back, lower back |
| Traps & neck | trapezius, sternocleidomastoid | traps, neck |
| Biceps | biceps, brachialis | biceps |
| Triceps | tricepsBrachii | triceps |
| Forearms | brachioradialis, flexorCarpiUlnaris, flexorCarpiRadialis, extensorCarpiUlnaris | forearms |
| Abs | rectusAbdominis | abdominals |
| Obliques | externalObliques | abdominals (rotation/anti-rotation exercises tagged obliques) |
| Glutes | gluteusMaximus, gluteusMedius | glutes, abductors |
| Quads | rectusFemoris, vastusMedialis, vastusLateralis, sartorius | quadriceps |
| Hamstrings | bicepsFemoris, semitendinosus | hamstrings |
| Adductors | adductorMagnus | adductors |
| Calves & shins | gastrocnemius, soleus, tibialisAnterior | calves |

Exercises store their tags at muscle-group level; the mapping above converts to Rive muscle
names at render time. Tap events arrive as Rive muscle names and are converted back to groups.

## Runtimes
- **Web (Next.js staff dashboard):** `@rive-app/react-webgl2` — the WebGL2 runtime supports the
  ViewModel data-binding used here. Load client-side only; key the component on the artboard name
  so switching Female/Male remounts it; write values through `rive.viewModelInstance`.
- **Mobile (React Native / Expo):** **Phase 2 spike** — confirm the React Native Rive runtime
  supports ViewModel data binding *and* Rive events for this file. Fallback: render the web
  runtime inside an Expo DOM component / WebView for this one screen.
- Accessibility: the canvas is not screen-reader friendly, so every view also offers the muscle
  group list as buttons, and the heatmap has a text/table equivalent.

## Verified in Phase 0
The file was rendered headlessly with `@rive-app/webgl2` (all four artboards, intensity and
palette writes applied) to produce the canvas previews and to measure tap regions.
