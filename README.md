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

- 0x88 board, signed piece types, precomputed piece-square and Zobrist
  lookup tables for fast incremental evaluation and hashing.
- Negamax Principal Variation Search (PVS) with iterative deepening to a
  fixed node budget per move (200,000 nodes; deterministic, no clock,
  no unseeded randomness).
- Transposition table with ply-adjusted mate scores, quiescence over
  captures / promotions / check evasions with delta pruning, check
  extensions, null-move pruning (adaptive R = 2/3), Reverse Futility
  Pruning, Late Move Reductions, and 2-fold repetition detection.
- Move ordering: transposition-table / root PV move first, true MVV-LVA
  captures + promotions + en passant, two killer moves per ply, and
  history heuristic.
- Evaluation: tapered middlegame/endgame PeSTO piece values and
  piece-square tables + bishop-pair bonus + tempo bonus.

## Bug fixes & upgrades

- Fixed castling move generation, double-pawn-push check evasions while
  in check (passes all standard perft suites including `pos4`),
  transposition-table move ordering, mate score ply adjustment,
  killer-move slot overwrite, `Abort` board-state cleanup, and per-move
  total node budgeting.
- Upgraded evaluation from middlegame-only Michniewski PSTs to tapered
  PeSTO MG/EG evaluation, added PVS, LMR, Reverse Futility Pruning,
  delta pruning, and history heuristic, and optimized 0x88 move
  generation / attack checks.
- Head-to-head vs previous version: 6-0 (all wins by checkmate) while
  running faster per move. `main.py` fits in 4,017 bytes (<= 4,096 B).

## Practice result

Practice trials (6 games each, vs the house gauntlet depth-2 to
nodes-500k), node budget 200,000:

- trial-0: Elo 2112
- trial-1: Elo 1809
- trial-2: Elo 1654

Mean practice Elo 1858, above the 1154 final floor and above the house
baseline (sunfish-4K, ~1365). Slowest move 4.53 s (median ~2.1 s), under
the 10 s limit; no timeouts. Deterministic: fixed nodes, no clock.

## Reproduce

```sh
makefun compress engine.py --out main.py
```

then run a practice trial against the kit's local env (see the kit's
`RULES.txt`).
