# Token Radar

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/baijorl21/token-launch-radar/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/baijorl21/token-launch-radar/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar for make.fun bounty `token-radar-01`: it reads the sealed DefiLlama snapshot,
ranks every protocol that records no token by an implied value derived from the snapshot's own
figures, and writes a static dashboard with a farm plan for each protocol.

## What it does

- `main.ts` (Node 22, standard library only, no network) reads `/input/snapshot.json`, scores the
  protocols with the model in `lib/model.ts`, and writes `/output/board.json`, `/output/data.js`,
  a copy of `dashboard/` and `screenshots/`, and `/output/dossier.json` (via `buildDossier` from
  `vendor/makefun-board.ts`, copied byte for byte from the official kit).
- The board covers the snapshot's whole watchlist and every other rankable pre-token protocol, so
  protocols below the watchlist floor are ranked before they cross it.
- `dashboard/` is a static page (board, filters, per-protocol detail view, live-token view) that
  renders from `data.js` with no network and recomputes each row's implied value in the page.

## The model (radar-2)

```
implied = round(tvl_usd x category_multiple x growth_multiplier x maturity_multiplier x breadth_multiplier)
```

- `category_multiple`: median market-cap-to-TVL ratio of live-token peers in the same DefiLlama
  category (at least 3 peers, else the global median), clamped to 0.05–1.50x.
- `growth_multiplier`: `1 + clamp(change_7d, -50, 100) / 100 x 0.4`.
- `maturity_multiplier`: `0.55 + min(age_days, 1095) / 1095 x 0.45` (unknown age counts as 365).
- `breadth_multiplier`: `1 + min(chains - 1, 4) x 0.03`.

Implied value is an estimate from snapshot figures. It is not a price, not a prediction that a token
will launch, and farming a protocol does not guarantee an airdrop.

## Run locally

```sh
mkdir -p out
MAKEFUN_OUTPUT_DIR=out node main.ts --input <path-to-snapshot.json>
```

Then open `out/dashboard/index.html` in a browser (it loads `../data.js`).

## Layout

- `main.ts` — entry program
- `lib/model.ts` — scoring model, board builder, farm plans
- `dashboard/` — `index.html`, `app.css`, `app.js`
- `screenshots/` — board, detail and filtered views of the dashboard
- `vendor/makefun-board.ts` — make.fun's dossier library from the official kit
- `LICENSE` — MIT
