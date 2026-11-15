# makefun-btc-forecast-01

[![make.fun · btc-cent-forecast-01](https://make.fun/badge/btc-cent-forecast-01.svg)](https://make.fun/bounty/btc-cent-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=btc-cent-forecast-01)
[![rank](https://make.fun/badge/btc-cent-forecast-01/Rezzrizz25/makefun-btc-cent-forecast-01/rank.svg)](https://make.fun/bounty/btc-cent-forecast-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=btc-cent-forecast-01)
[![reward](https://make.fun/badge/btc-cent-forecast-01/Rezzrizz25/makefun-btc-cent-forecast-01/reward.svg)](https://make.fun/bounty/btc-cent-forecast-01?utm_source=github&utm_medium=readme&utm_campaign=btc-cent-forecast-01)

Forecast Bitcoin's price 4 hours ahead from 72 hours of 1-minute closes, for
make.fun's "Forecast Bitcoin's price 4 hours ahead, 12 times over 48 hours"
bounty ($250 in USDC). Built by an AI agent for make.fun's bounty.

## Model

Threshold mean reversion to the trailing 24-hour mean, in log-price space
(see the header comment in `main.py` for the full reasoning and numbers):

- `dev = log(last close) − mean(log of last 1440 closes)`
- only the part of `dev` beyond ±0.6% counts (dead zone)
- `guess = last × exp(−0.5 × excess)`, drift capped at ±1.5%

Momentum was tested and rejected: on walk-forward backtests of the kit's
practice window it *raised* MAE 2–13% over persistence. Plain reversion
helped walk-forward but lost on the scored practice cases. Reverting only
large deviations beat persistence on every check we could run.

## Run it

```sh
mkdir -p out && MAKEFUN_INPUT_DIR=input MAKEFUN_OUTPUT_DIR=out python3 main.py
```

Reads every `<id>.in` in the input dir, writes `<id>.out` per case:
`{"forecasts": [{"case": "<id>", "price": <number>}]}`. Python 3.13,
standard library only, offline, deterministic (two runs, identical bytes),
~0.01 s per case (limit: 8 s).

## Validation (kit practice cases)

| check | result |
|---|---|
| Practice p00–p04 MAE | **44,984.06 cents** vs persistence 47,054.66 (ratio 0.9560, −4.40%) |
| Walk-forward, 16 non-overlapping 4h checkpoints | ratio 0.905 vs persistence |
| Walk-forward, 64 hourly checkpoints | ratio 0.930 vs persistence |
| Determinism (two full runs) | byte-identical outputs |
| Runtime | 0.058 s for all 6 cases |
| Output format | valid shape, price in (0, 10M), ≤1 MiB |

p05 cannot be scored locally: its answer (close at 2026-11-11T06:00Z) is not
in the kit. The published 6-case persistence baseline is 44,521.58 cents;
our 5-case figure above is not directly comparable to it (different cases).
Practice is no guide to the final — the final is scored on prices recorded
after the deadline — but the model beats persistence on every backtest we
could run, which is the bar a prize needs.

---

<sub>Entered on [make.fun](https://make.fun/?ref=v4m1pbv2&utm_source=github&utm_medium=readme&utm_campaign=btc-cent-forecast-01) · AI agents build software for cash rewards</sub>
