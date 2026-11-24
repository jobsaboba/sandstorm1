# 🌪️ Sandstorm 1%

Sandstorm's 2026 all-weather picks, re-settled as a **1% bankroll book at SP**.

- Starts at **£10.00**.
- **1% of the current bank** per horse, rounded to the penny.
- **Win** pays the SP; **place** pays **1/5 of the SP**.
- Only races with **8+ actual runners** are bet.
- All picks in one race share the **pre-race bank**; the bank moves after the race.
- Actual starting prices from **Sporting Life result pages** — not modelled or forecast odds.

Live app: **https://jobsaboba.github.io/sandstorm1/**

The selection rules are Sandstorm's locked v5 sheet, unchanged: UK & Irish
all-weather flat handicaps, course-&-distance winner, ran within 28 days,
today's mark below the last winning C+D mark, priced 3/1–33/1, shortest odds
first, at most three per race.

## Settlement model

This is a **single-stake win/place book**, not a half-stake each-way split:

| Result | Return |
| --- | --- |
| Win | `stake × SP` profit |
| Place | `stake × SP ÷ 5` profit |
| Unplaced | `−stake` |
| Non-runner / withdrawn | no bet |
| Abandoned / void | no bet |
| Under 8 runners | no bet |

Dead heats divide the stake by the number of horses sharing the finish; the
winning share pays full SP, the rest loses. A place dead heat only reduces the
return when it is for the **last** paid place.

> A true each-way split (0.5% win + 0.5% place) is a different, much harsher
> lens. This app deliberately uses one 1% stake per horse, matching “1% of the
> current bankroll per horse”.

## Data

Picks are read from Sandstorm's stored selection sheets:

```
~/wonderland/drafts-28/YYYY-MM-DD/selections.json
```

Result pages are cached as compact JSON in `pipeline/cache/race-<id>.json`.
The compact file holds the race stage, actual runner count, place terms, each
runner's status/finish position, and the result-page odds (`current_odds`,
treated as the SP).

The app data is static JSON:

```
data/YYYY-MM-DD.json   one day: races, picks, SP, stake, return, bank
data/latest.json       the current day
data/index.json        small bootstrap index
data/season.json       season summary, months, chart, records
```

Only files whose contents actually changed are rewritten, so a quiet refresh
does not create a commit.

## Commands

```bash
cd ~/projects/sandstorm1

# Full rebuild from the cached result pages
python3 pipeline/sandstorm1.py --rebuild

# Settle today's open races (refetch unresolved pages) + push
python3 pipeline/sandstorm1.py --settle --publish

# Morning: rebuild the day's pending picks + push
python3 pipeline/sandstorm1.py --publish

# Print the season summary without rebuilding
python3 pipeline/sandstorm1.py --summary
```

`--publish` commits and pushes `data/` to GitHub; the phone app pulls it on ⟳.
`--no-push` commits locally without pushing.

## Automation

- `~/.local/bin/sandstorm-morning.sh` runs after the Sandstorm 05:00 publish and
  pushes the new day as pending.
- `~/.local/bin/results-refresh.sh` runs every 30 minutes through the racing
  afternoon, settles finished races, and pushes the updated bank.

Logs go to `~/wonderland/cache/sandstorm-morning.log` and
`~/racing-scripts/cache/results-cron.log`.

## Stack

- Python 3 pipeline (`requests`, standard library otherwise).
- Static PWA: vanilla HTML/CSS/JS, manifest, service worker.
- GitHub Pages, project repo `jobsaboba/sandstorm1`.

## Notes

- The historical sheet was fetched after racing, so many stored pick odds already
  equal the SP. Live picks are scored in the morning and settle at the actual SP
  later.
- Non-finishers are losses.
- The bank is a backtest, not a guarantee.
