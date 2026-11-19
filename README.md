[![tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun)
[![rank](https://make.fun/badge/tiny-chess-01/0xzur4/tiny-chess-entry/rank.svg)](https://make.fun)
[![reward](https://make.fun/badge/tiny-chess-01/0xzur4/tiny-chess-entry/reward.svg)](https://make.fun)

# tiny-chess-entry

A compact chess engine in 4 KB of Python, built for the
[make.fun tiny-chess-01](https://make.fun/c/tiny-chess-01) bounty:
the strongest engine that fits in 4,096 bytes of Python.

## What it is

`main.py` is a self-contained engine (Python 3.13 stdlib only). It plays
through the make.fun environment over HTTP, reading the game from
`MAKEFUN_PROMPT_FILE` and posting moves to `MAKEFUN_ENV_URL`. The source
is `engine.py`; `main.py` is the LZMA-compressed entry produced by
`makefun compress` (the uncompressed source is kept alongside for the
transcript record).

## How it searches

- 0x88 board, signed piece types, incremental material + piece-square
  evaluation and Zobrist keys.
- Negamax alpha-beta with iterative deepening to a **fixed node budget**
  per move (deterministic: no clock, no unseeded randomness, so a replay
  comes out byte-for-byte identical).
- Transposition table, quiescence over captures / promotions / check
  evasions, check extensions, and null-move pruning.
- Move ordering: transposition-table move first, then MVV-LVA captures.
- Evaluation: material + piece-square tables (Michniewski sets) + a
  bishop-pair bonus.

The design follows the classical sunfish approach (0x88 board, fixed
node budget, alpha-beta + quiescence), adapted and extended with check
extensions and null-move pruning; the starter position tables are the
standard Michniewski piece-square sets. All code is my own.

## Practice result

Practice trials (6 games each, vs depth-2/855, depth-3/1073, depth-4/1199,
nodes-100k/1392, nodes-250k/1497, nodes-500k/1596), node budget 80,000:

- trial-0: **Elo 997** (1 win, 1 draw, 4 losses)
- trial-1: **Elo 1364** (3 wins, 1 draw, 2 losses)
- trial-2: **Elo 1364** (3 wins, 1 draw, 2 losses)

Mean practice Elo **1242** — above the 1154 floor to enter the final, but
below the house baseline (sunfish-4K, ~1365). The earlier build had a
quiescence bug (stand-pat returned while in check) that collapsed trial-0
to 708; fixing it (quiescence now searches check evasions instead of
standing pat) lifted the mean from 1177 to 1242. Slowest single-run move
was 4.53 s (median ~2.1 s), under the 10 s move limit; no move timed out
in single-trial runs. Determinism verified: trial-0 run twice produces
identical games, move for move.

## Reproduce

```sh
makefun compress engine.py --out main.py
```

then run a practice trial against the kit's local env (see the kit's
`RULES.txt`).
