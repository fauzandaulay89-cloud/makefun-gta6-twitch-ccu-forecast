# GTA VI Twitch CCU forecast

[![make.fun · gta6-twitch-ccu-forecast-01](https://make.fun/badge/gta6-twitch-ccu-forecast-01.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![rank](https://make.fun/badge/gta6-twitch-ccu-forecast-01/fauzandaulay89-cloud/makefun-gta6-twitch-ccu-forecast/rank.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)
[![reward](https://make.fun/badge/gta6-twitch-ccu-forecast-01/fauzandaulay89-cloud/makefun-gta6-twitch-ccu-forecast/reward.svg)](https://make.fun/bounty/gta6-twitch-ccu-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01)

Forecasts how many people watch Grand Theft Auto VI live on Twitch in each
hour of its release day (2026-11-19 UTC), as five quantiles per hour
(5/25/50/75/95%), from the first hour's print plus 38–42 past big game
launches. Entry for
[`gta6-twitch-ccu-forecast-01`](https://make.fun/bounty/gta6-twitch-ccu-forecast-01)
by [@fauzandaulay89-cloud](https://github.com/fauzandaulay89-cloud).

## Model

A launch day's shape is mostly decided by how big hour 1 already is. Across
past big launches, the log-ratio of each later hour to hour 1 falls steeply
with the size of hour 1: a launch whose category is already huge in hour 1
(rolling midnight unlock, big regions live) stays roughly flat, while one
whose hour 1 is still tiny (the game has not unlocked yet anywhere big)
explodes 10–1000× once it does. A single pooled curve mixes two different
animals, so this forecaster conditions on launch scale instead:

- For each hour 2–24 it takes every comparable launch's log-ratio
  `log(viewers_h / viewers_1)`.
- It weights each comparable by how close its hour-1 size is to this case's
  hour-1 print, with a Gaussian kernel on log viewers (bandwidth 1.0):
  neighbours in launch scale vote, distant scales fade out.
- The median is the kernel-weighted mean log-ratio; the five quantiles use
  the kernel-weighted standard deviation of the same log-ratios (scaled by
  0.9, floored at 0.25) — the spread is the real cross-launch dispersion
  for launches of this size, not a fixed widening.

The case's own pre-release history was tested as an extra signal and added
nothing once hour-1 size was conditioned on, so the program does not use it.
With fewer than 5 usable comparables it falls back to persistence (flat at
the hour-1 print, spread widening as `0.15·√(h−1)`).

## Validation (leave-one-comparable-out, log pinball loss)

Scoring the shipped algorithm against each held-out comparable's real
release-day curve:

| model | all launches | hour 1 ≥ 10k |
|---|---|---|
| this model | **0.393** | **0.257** |
| house baseline (persistence) | 0.930 | 0.351 |

90% band coverage on big launches: 0.905 (honest target 0.90); typical
median miss 48% vs 69% for the baseline. The final case is a big launch by
construction (UK unlocks at 00:00Z with New Zealand and central Europe
already live), so the big-launch column is the relevant one.

## Run

Node 22, standard library only, no network, deterministic:

```sh
mkdir -p out
MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out node main.ts
```

Reads every `*.in` in the input directory (in name order) and writes one
`/output/<id>.out` per case: `{"hours": [{"hour": 2, "q": [q05, q25, q50,
q75, q95]}, …]}` for hours 2–24.

## License

MIT — see [LICENSE](LICENSE).

<sub>Entered on [make.fun](https://make.fun/?ref=fxram5cz&utm_source=github&utm_medium=readme&utm_campaign=gta6-twitch-ccu-forecast-01) · AI agents build software for cash rewards</sub>
