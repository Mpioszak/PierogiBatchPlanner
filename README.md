# Pierogi Batch Planner

A single-page production worksheet for pierogi. Enter how many have to come off the table and it works backward to flour, water, filling, sheet pans, freezer bags, crew hours and a running order — from a dozen for dinner up to a parish-hall morning.

**Live:** https://mpioszak.github.io/PierogiBatchPlanner/

---

## Why it exists

Pierogi recipes are written for one batch and don't survive multiplication. Scaling by eye fails on the parts that actually constrain a big session:

- **Filling outweighs dough.** Roughly 1 : 1.2 by weight, so a batch needs more filling than most people plan for, and it has to be made and fully cooled the day before.
- **Flour isn't the dough weight.** Dough weight includes water, salt and fat, so you have to work back through hydration to get the flour figure.
- **The real limits are pans, freezer space and hands.** A 400-piece batch needs eight full sheet pans in a single layer before anything can be bagged.

Everything here is per-piece, so the numbers scale cleanly in either direction.

## What it does

- **Three sizes** — 6.5 cm / 2½ in, 7.5 cm / 3 in, 9 cm / 3½ in — each with its own dough and filling weight per piece, and its own pan, bag and pot capacity.
- **Five fillings** with full ingredient breakdowns that scale to the batch, including trim notes (what to buy unpeeled, what onion cooks down to, what dried mushrooms rehydrate to).
- **Dough table** derived from hydration and a trim allowance, with bench flour called out separately so the shopping figure is right.
- **Line planning** — output per hour for the crew size, elapsed time, and the same job done by one pair of hands for comparison.
- **Freezer and pot** — sheet pans, bags, pot load, boiling throughput, and litres of water if serving fresh rather than freezing.
- **Running order** — a five-step T−countdown from the day before through bagging, with the batch's own numbers written into each step.
- **Batch presets** for 12, 60, 144, 400 and 1200 pieces.

## The math

**Dough**

```
dough needed = pieces × dough per piece × (1 + trim%)
multiplier   = 1 + hydration% + salt% + fat%
flour        = dough needed ÷ multiplier
water        = flour × hydration%
salt         = flour × 1.4%
fat          = flour × 3%
bench flour  = flour × 4%
```

Salt and fat are fixed baker's percentages in the code and aren't exposed as inputs. Bench flour sits outside the dough total — it's added to the shopping figure, not to what comes off the bench.

**Filling**

```
filling needed = pieces × filling per piece
scale          = filling needed ÷ 1000
```

Each filling's rows are written per **1000 g of finished filling** and multiplied by that scale.

**Line**

```
rate       = pierogi per person-hour, by crew size
output     = rate × crew
line hours = pieces ÷ output
```

| Crew | Per person-hour |
| --- | --- |
| 1 | 45 |
| 2 | 55 |
| 3–5 | 65 |
| 6–9 | 70 |
| 10+ | 75 |

The rate climbs with crew size because the work splits into stations — rolling, filling, sealing, racking — then flattens out. Assembly only; it assumes filling made the day before and dough already resting.

**Freezer and pot**

```
sheet pans   = ceil(pieces ÷ per pan)
freezer bags = ceil(pieces ÷ per bag)
throughput   = pot load × 10 per hour
water        = ceil(pieces ÷ pot load) × 5 L
```

### Sizes

| Key | Size | Dough | Filling | Per pan | Per bag | Per pot |
| --- | --- | --- | --- | --- | --- | --- |
| `small` | Małe — 6.5 cm / 2½ in | 14 g | 16 g | 64 | 32 | 40 |
| `standard` | Standard — 7.5 cm / 3 in | 20 g | 24 g | 48 | 24 | 30 |
| `large` | Duże — 9 cm / 3½ in | 28 g | 34 g | 35 | 18 | 22 |

### Fillings

| Key | Filling |
| --- | --- |
| `ruskie` | Ruskie — potato & twaróg |
| `kapusta` | Kapusta i grzyby — kraut & mushroom |
| `mieso` | Mięso — braised meat |
| `serowe` | Serowe na słodko — sweet cheese |
| `jagodowe` | Jagodowe — blueberry |

### Inputs

| Control | Default | Range |
| --- | --- | --- |
| Pierogi to make | 400 | 1 and up (step 12) |
| Crew on the line | 6 | 1–24 |
| Hydration % | 54 | 40–70 |
| Trim allowance % | 8 | 0–25 |

## Running it

One file, no build step, no package manager:

```
git clone https://github.com/Mpioszak/PierogiBatchPlanner.git
cd PierogiBatchPlanner
open index.html          # or: python3 -m http.server
```

Deployed through GitHub Pages from `main`.

### Structure

```
index.html    # markup, styles and logic — the entire application
```

- Vanilla JavaScript, no framework or bundler.
- The only external request is a Google Fonts stylesheet (Archivo, Source Sans 3, Source Code Pro). System-font fallbacks are declared, so it renders fine offline.
- Light and dark themes via `prefers-color-scheme`, with a `data-theme` attribute override.
- Prep-sheet layout: batch controls across the top, then four stations in a two-up grid, stacking below 820px.

### Changing the data

Sizes live in `SIZES`, fillings in `FILLINGS`, near the top of the script.

| `SIZES` field | Meaning |
| --- | --- |
| `label` | Name shown in the dropdown |
| `dough`, `fill` | Grams per piece |
| `perPan`, `perBag`, `perPot` | Capacity figures |

| `FILLINGS` field | Meaning |
| --- | --- |
| `label` | Name shown in the dropdown |
| `pl` | Polish name shown in the station header |
| `rows` | `[ingredient, grams per 1000 g, note]` |
| `note` | Practical note under the table |

Filling rows should add up to roughly 1000 g so the scaling stays honest. Adding to either list needs no other changes.

## Known quirks

- **"Same job, one pair of hands"** divides by the rate for the *selected* crew size rather than the solo rate of 45/hr, so at a crew of 6 it reports a faster solo time than one person would actually manage. If you want the true single-handed figure, divide pieces by 45.
- `fillKg` is computed in `render()` and never used — there's a `void fillKg;` holding it in place.

## Accuracy

Dough and filling proportions follow standard Polish home and parish-hall practice. Output rates are assembly only and assume prep is done; a drilled parish line beats them, a first-time kitchen won't come close. Weigh the dough for the first dozen to calibrate, then work by eye.

## License

Not yet specified. Add a `LICENSE` file if you want to set terms for reuse.
