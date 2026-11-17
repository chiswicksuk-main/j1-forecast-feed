# Handover audit — 2026-11-08

Scope: public feed `chiswicksuk-main/j1-forecast-feed`, live site
https://jleaguescout.soraretactics.com/football/market-intelligence, and whatever
could be inferred from published JSON. **Blocked:** private `chiswicksuk-main/j1-forecast`
(docs, workflow, publisher code) — not in this Cloud Agent environment / GitHub token scope.

Related feed fix (separate PR): restore Round 14 freeze scoring —
https://github.com/chiswicksuk-main/j1-forecast-feed/pull/1

## Verified against handover

| Expectation | Status | Evidence |
| --- | --- | --- |
| Feed schema `jls-forecast-feed/1` | OK | `latest.json` |
| Files: `latest.json`, `rounds/round-N.json`, `market-record.json`, `j2/latest.json` | OK | repo layout |
| Model label `blend-0.2.0`, `w_elo=0.4`, `elo-0.1.1`, 20k sims | OK | `model.version` / `season_simulations` |
| Adjustments separate from `results_only`; season odds results-only | OK | `adjustments_info.note`, match objects |
| ACL uses D-035 bands (not flat -10 only) | OK in feed metadata | `acl_bands` + per-match reasons; top-level `acl_elo: -10` is documented as full-strength reference |
| Player importance Star/Key/Squad/Fringe | Present in feed metadata | `player_importance` |
| Missing odds shown as unavailable, not fabricated | OK on site | Round 15 market columns are `—`; `market_odds.available: false` |
| Backtest vs published labelling | OK structurally | `forecast_source` / `by_round.source` |
| J2 separated, no injury layer | OK | `jls-j2-forecast-feed/1`, no adjustments |
| Site consumes feed (display layer) | OK | RSC payload embeds feed `created_utc` matching `main` |

## Genuine problems

### P0 — Published forecasts rewritten after kick-off / mid-round

`rounds/round-14.json` freeze (`issued_utc` 2026-11-07T00:28:31Z, commit `5211316…`) disagrees with `main` `season_history` / `market-record` for Sunday R14 matches (and several Saturday rows point at later issue commits).

Those later issues incorporate post-Saturday information while still labelled `forecast_source: "published"`. This violates the handover integrity rule.

**Mitigation in flight:** PR #1 restores freeze probs/RPS on the feed. **Still required in publisher:** lock scoring to the round snapshot; do not overwrite published history on later 2-hourly runs.

### P0 — Odds snapshot contains no upcoming fixture

On the 10:28 UTC feed of 8 Nov:

- `events_in_snapshot: 4`
- `matched_fixtures: 0`
- `unmatched_names: []`
- all upcoming J1 `market` fields null
- the site shows those markets as unavailable, which is the right display

This is not a failed name match. The four events are the Sunday Round 14 fixtures: Machida vs Hiroshima, Kobe vs Urawa, Gamba vs Cerezo, and JEF vs Mito. Every club name resolves, including `FC Machida Zelvia` and `Hiroshima Sanfrecce FC`. By 10:28 UTC those matches had kicked off, and the next open card is Round 15 on 20 Nov. The fetch policy does not call the API again when the next match is more than four days away, so this finished-round snapshot stays the latest file. An empty `unmatched_names` list means the names mapped.

Quota remains (`450`). Publisher PR #5 now lists these events on `events_not_on_card` and in `odds_note` when nothing attaches. Markets stay blank until a snapshot contains a fixture that is still upcoming.

### P1 — Generated `model.inputs` text is false

Feed still says: “Official J1/J2 results since 2003 only. No team news, injuries or betting odds.”

Reality: ACL + team-news Elo adjustments exist; version string includes markets lambda tilt; R14 freeze carried extensive team-news adjustments. This misleads readers and contradicts the adjustment layer design.

### P1 — Team-news / ACL layer not helping yet (monitor, don’t abandon)

`track_record.team_news_test` (9 matches): adjustments RPS 0.2393 vs results-only 0.2241 (difference +0.0151, SE 0.0184). On the R14 freeze, adjusted forecasts were also worse than `results_only` in aggregate. Keep the layer, keep measuring; do not fold into results-only ratings without a tested win.

### P1 — Historical model-vs-market lag

`model_v_market` covers rounds 1–13 only (closing odds via football-data). Live `market-record.json` holds R14 Odds API comparisons. Closing-odds path for published R14 is not in `model_v_market` yet.

### P2 — Stale / confusing operational fields

- `status.team_news_sync.round: 14` after R14 is complete and R15–19 are the live horizon.
- `odds_fetched_utc` (05:57Z) older than `model_run_utc` (10:28Z) on the same publish — clarify whether odds refresh every run.
- R16 still contains postponed fixtures through **2027-04-14**; legitimate if official, but consumers need clear postponed labelling.

### P2 — Public feed README was empty

Replaced by this audit + an architecture README. Private docs (`DECISIONS.md` D-034+, `HANDOFF.md`, `HOSTING.md`, `TEAM_NEWS.md`, ChatGPT MI instructions, `forecast.yml`) **not audited** until `j1-forecast` is attached.

### P2 — Cannot verify schedule / “republish only when inputs change”

Handover requires 2-hourly at minute 17 (not daily) and republish-on-change. Workflow file is private. Feed commit timestamps on 2026-11-07/08 show multiple publishes through the day (including :28, :40, :58), consistent with a frequent schedule, but change-detection quality is unverified.

## Freshness check (2026-11-08 ~11:50 UTC)

| Stage | Timestamp | Notes |
| --- | --- | --- |
| Publisher run (linked) | Actions run `42471431765` | Private UI 404 to this agent |
| Feed `created_utc` | 2026-11-08T10:28:27Z | ~1.4h old at audit |
| `data_through_jst` | 2026-11-08 17:00 | Aligns with completed R14 |
| Site embed | Same `created_utc` / model string as `main` | Cache serving current main feed |
| Odds snapshot | 2026-11-08T05:57:40Z | Four finished Round 14 fixtures; club names resolved |

## Priority recommendations

1. **Publisher freeze lock (P0)** — Score and store published rows only from `rounds/round-N.json`. Later runs may update *upcoming* rounds and season sims, never past kick-off published probabilities. Add a CI/publish assertion: history published probs == freeze.
2. **Odds when the snapshot is only finished matches (diagnosed)** — Do not treat an empty `unmatched_names` list as a matcher failure. Name the events that are not on the upcoming card. Keep the site on unavailable until a snapshot actually contains Round 15 prices.
3. **Honest `model.inputs` (P1)** — Generate copy that matches the adjustment + markets-tilt architecture; keep results-only wording only on `results_only` / season odds.
4. **Attach `j1-forecast` to the agent environment (P1)** — Audit `forecast.yml`, D-034+, team-news sync, snapshot retention, and republish-on-change gating against code.
5. **Adjustment calibration dashboard (P1)** — Track team-news and ACL deltas vs results-only continuously (extend `team_news_test`); no merge into baseline without out-of-sample win.
6. **Close the R14 closing-odds gap (P2)** — Extend `model_v_market` when football-data catches up; keep Odds API `market-record` distinct.
7. **Operational clarity (P2)** — Refresh odds on publish or document why not; advance `team_news_sync` target to the next unsettled round; mark postponed R16 dates explicitly.
8. **No model rewrite without evidence (standing rule)** — `blend-0.2.0` stays until a versioned backtest beats the frozen baseline.

## What was changed in this PR

- Replaced empty root `README.md` with accurate public-feed architecture notes.
- Added this audit document.

## What remains unverified

- `.github/workflows/forecast.yml` schedule and republish gating
- Snapshot archive retention for Odds API
- Google Sheets sync details (Flashscore IDs, D-042 off, importance overwrite guards)
- Whether paid El Golazo/Targma text can leak into public adjustment reasons
- Website last-good-feed fallback behaviour under feed outage
- Any outdated private documentation vs code

## Tests run

- Compared `main` feed JSON to `rounds/round-14.json` and `market-record.json` (integrity drift confirmed).
- Fetched live site HTML/RSC payload and confirmed it mirrors `main` feed timestamps and Round 15 probabilities, with markets shown unavailable.
- Confirmed J2 feed has no adjustment layer.
- Could not clone or read `chiswicksuk-main/j1-forecast` (GitHub 404 for this token).
