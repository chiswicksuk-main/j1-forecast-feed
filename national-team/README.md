# Samurai Blue research handoff

This directory is an editorial research inbox for J League Scout, separate from the J1/J2 forecast feeds. It has **no impact** on `latest.json`, J2 feeds, forecasts, or model automation.

## Read/write endpoints

- Latest working research: `national-team/latest.md`
- Raw read URL: https://raw.githubusercontent.com/chiswicksuk-main/j1-forecast-feed/main/national-team/latest.md
- Editor view: https://github.com/chiswicksuk-main/j1-forecast-feed/blob/main/national-team/latest.md
- Site hub: https://jleaguescout.soraretactics.com/football/samurai-blue
- Brazil preview: https://jleaguescout.soraretactics.com/football/samurai-blue/japan-v-brazil-2026

Grok should update the *same* `latest.md` file using authenticated GitHub access (git commit/push, or GitHub Contents API with the current file SHA). A local path such as `/workspace/japan-nt-20261113/sweep1.md` is not accessible to ChatGPT or this public URL. Never store API tokens, paid-source copy, private player data, or credentials here.

## Scope and deadline

Research the senior Japan national team versus Brazil in Singapore on **Saturday 14 November 2026 at 10:15 UK / 19:15 JST**, and the Japan U21s versus Spain on **Friday 13 November at 18:00 UK / Saturday 14 November at 03:00 JST**. Stop pre-match sweeps after the last of these kick-offs unless explicitly instructed otherwise. Do not silently expand to Paraguay or Egypt post-14-November coverage.

## Research standards

1. Japan FA/JFA, CBF, Spanish federation, official match sheets and official team media first. Then Soccer King, Nikkan, Hochi, Sponichi, Kyodo, Gekisaka, Globo and credible reporters. Search Japanese first where relevant. Prioritise fresh reporting dated 11–14 November.
2. Cite **direct article URLs**, not search pages. Record publication time in the original time zone if available. Each reported claim gets status `OFFICIAL`, `REPORTED`, `PREDICTION`, or `UNVERIFIED` and confidence `A/B/C`. Explain the grading, and separate fact from interpretation.
3. Preserve the confirmed JFA U21 squad change dated 8 November: **Shin Miidera OUT (injury), Sena Ishibashi IN (Shonan)**. Do not include Miidera among available players or predicted lineups. Check for further changes.
4. Differentiate **press-predicted XIs**, **Matsu's own XI**, and **official XIs**. Never upgrade predictions to confirmed team sheets. If sources disagree on a back three versus back four or Ogawa versus Maeda, report both and why.
5. Keep unchanged information short. Lead with material changes since the previous sweep, a clear status, exact source URL and suggested page-level edit. Report uncertainty or missing checks. Never fabricate press quotes, squad lists, minutes or match events.
6. This is a research inbox, **not an automatic public publishing feed**. An editor or a verified site-write integration must review and apply changes. Preserve live page design, canonical URLs, and SEO metadata.

## Suggested file layout

`latest.md` should carry:
- `Updated at` in UTC and JST, `Sweep ID`, `Cutoff`.
- `Executive changes since previous sweep`.
- `Confirmed squad & fitness` (Japan, Brazil, U21, Spain).
- `Predicted XI` with formation, individual names, source and confidence.
- `J1 / Sorare availability risk` only where relevant; keep it out of the casual-audience public article unless warranted.
- `Tactical observations / original angles`.
- `Site changes to make` with **exact URL and existing text/section**.
- `Unresolved checks` and `Sources` (direct URL, timestamp, reliability).
- `Publish state`: inbox, reviewed, published, or superseded.

The editors should compare `latest.md` against the live pages, not overwrite the site with raw notes.
