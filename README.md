# j1-forecast-feed

Public forecast feed for [J League Scout](https://jleaguescout.soraretactics.com/football/market-intelligence).

This repository is the **published data surface** only. Modelling, team-news adjustments, odds capture and publishing run in the private `chiswicksuk-main/j1-forecast` repo. The website consumes this feed server-side; it is a display layer, not a second forecasting engine.

## Live URLs

| File | URL |
| --- | --- |
| J1 latest | https://raw.githubusercontent.com/chiswicksuk-main/j1-forecast-feed/main/latest.json |
| J1 market record | https://raw.githubusercontent.com/chiswicksuk-main/j1-forecast-feed/main/market-record.json |
| J1 frozen round | https://raw.githubusercontent.com/chiswicksuk-main/j1-forecast-feed/main/rounds/round-N.json |
| J2 latest | https://raw.githubusercontent.com/chiswicksuk-main/j1-forecast-feed/main/j2/latest.json |

## Schemas

- J1 feed: `jls-forecast-feed/1`
- J1 round snapshot: `jls-round-snapshot/1`
- J1 market record: `jls-market-record/1`
- J2 feed: `jls-j2-forecast-feed/1`

## Integrity rules (owner)

- Historic predictions must never be rewritten with information that became available after kick-off.
- `rounds/round-N.json` is the frozen pre-match published forecast for that round.
- `season_history` rows with `forecast_source: "published"` must match the freeze.
- Backtests and published forecasts are labelled separately.
- Bookmaker odds stay separate from model probabilities; missing odds are unavailable, never fabricated.
- Temporary team-news / ACL adjustments are a separate layer versus `results_only`.
- J2 remains a simpler results-based model without invented injury/lineup coverage.

## Current model label

Published J1 model string (see `latest.json` → `model.version`):

`blend-0.2.0 w_elo=0.4 | markets v2 (lambda tilt) | elo-0.1.1 …`

- 40% Elo (`elo-0.1.1`) + 60% attack/defence Poisson
- Markets lambda tilt on match probabilities; season odds use results only
- 20,000 season simulations

Do not re-tune casually: any model change needs a version bump, out-of-sample comparison to the frozen baseline, tests and a documented decision.

## Docs

- [Handover audit (2026-11-08)](docs/HANDOVER_AUDIT.md) — public-feed findings and priorities. Private-repo docs (`docs/DECISIONS.md` onward) live in `j1-forecast` and were not readable from the feed-only agent environment at audit time.
