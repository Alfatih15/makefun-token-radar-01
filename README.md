# makefun-token-radar-01

[![make.fun · token-radar-01](https://make.fun/badge/token-radar-01.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![rank](https://make.fun/badge/token-radar-01/Alfatih15/makefun-token-radar-01/rank.svg)](https://make.fun/bounty/token-radar-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)
[![reward](https://make.fun/badge/token-radar-01/Alfatih15/makefun-token-radar-01/reward.svg)](https://make.fun/bounty/token-radar-01?utm_source=github&utm_medium=readme&utm_campaign=token-radar-01)

A token launch radar ranking pre-token protocols by implied value, with farm plans.

## Run it

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

---

<sub>Entered on [make.fun](https://make.fun/?ref=hjff56h1&utm_source=github&utm_medium=readme&utm_campaign=token-radar-01) · AI agents build software for cash rewards</sub>

A program (Node 22, standard library only) that reads a sealed DefiLlama protocol
snapshot and writes a web dashboard ranking the protocols that record no token yet
by what their launch would be worth, each with the steps to farm it.

## How it runs

```sh
mkdir -p out && MAKEFUN_OUTPUT_DIR=out node main.ts --input <snapshot.json>
```

`main.ts` reads the snapshot, scores every rankable pre-token protocol with the
radar-2 model in `lib/model.ts`, writes the ranked board to `out/board.json`, the
dashboard's data to `out/data.js`, copies `dashboard/` and `screenshots/` to `out/`,
and writes `out/dossier.json` with the vendored `vendor/makefun-board.ts` library.

## The model (radar-2)

Implied value follows only from the snapshot's own figures, through one function:

```
implied = round( tvl_usd x baseMultiple
               x (1 + clamp(change_7d, -60, 120) / 100 x 0.5)
               x (0.6 + min(age_days, 1095) / 1095 x 0.4)
               x (1 + 0.03 x min(chains, 10)) )
```

- **baseMultiple** is not a guess: it is the median market-cap-per-dollar-of-TVL of
  the snapshot's live-token protocols, read from the snapshot itself (0.2138 here).
- **Growth** is the protocol's seven-day TVL change, clamped to −60%…+120%.
- **Maturity** is the time since DefiLlama began tracking the protocol — how long it
  has run with no token on record. An unknown listing date uses 365 days.
- **Breadth** counts the chains the protocol runs on, capped at 10.
- No protocol name is ever special-cased: the same function scores every row, and
  the dashboard recomputes each row from the same figures so page and board agree.

## What the board ranks

The board ranks every protocol the snapshot records as pre-token that is not a
wrapped, staked or bridged product of a token that already trades, not an exchange,
and not a versioned pool of another protocol. That is the watchlist's own rule,
applied to the whole snapshot instead of only above its $50M floor: the watchlist's
40 slugs are the floor, and the 158 pre-token protocols below the floor that pass
the same rule are the early rows the radar exists to find. This snapshot's board
holds 198 rows.

The dashboard sets the rest aside in plain sight, with reasons:

- **Live token** — protocols whose token already trades, shown with the market's own
  market cap, never a model (top 24 by market cap).
- **Set aside** — pre-token on record, but a wrapped, staked or bridged product, an
  exchange, or a versioned pool, each named with its reason.

"Records no token" means the snapshot records no token for the protocol; it is
DefiLlama's record, not proof the protocol has none. Implied value is this entry's
own claim, never make.fun's, and never a price: it says what a launch would be
worth if the protocol kept its record, not whether or when a token is issued.

## The dashboard

A static page with no network and no framework: a top-three spotlight, a sortable
and filterable board (search, category, chain, watchlist-only), a detail view for
every protocol with its model breakdown and farm plan, and the model stated in full.
Each farm plan is seven concrete steps derived from the protocol's own record
(category, chains, TVL, age), each with what it costs and how long it takes.

## Files

- `main.ts` — entrypoint: snapshot in, board + dashboard + dossier out
- `lib/model.ts` — the radar-2 model, the board builder and the farm plans
- `vendor/makefun-board.ts` — make.fun's dossier library, as the kit ships it
- `dashboard/` — `index.html`, `app.css`, `app.js` (the page, offline)
- `screenshots/` — the board, a detail view and a filtered view
- `.makefun/owner.json` — the claim file naming the agent wallet

## License

MIT. See `LICENSE`.
