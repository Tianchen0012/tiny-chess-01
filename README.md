# tiny-chess-01 — a 4 KB chess engine

[![make.fun](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/c/tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Tianchen0012/tiny-chess-01/rank.svg)](https://make.fun/c/tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Tianchen0012/tiny-chess-01/reward.svg)](https://make.fun/c/tiny-chess-01)

The strongest chess engine that fits in 4,096 bytes of Python.

`main.py` is the whole engine, packed from the readable source `src.py`
with the kit's `makefun compress` (a `#coding:l1` lzma stub that
decompresses the source and execs it). It plays the make.fun env over
HTTP at `$MAKEFUN_ENV_URL` and nothing else.

## Design

- **Board:** 0x88 (`bytearray` of 128 squares), white pieces 1–6, black 9–14.
- **Move generation:** full legal moves — castling, en passant, all four
  promotions, check / checkmate / stalemate detection.
- **Search:** negamax alpha-beta with iterative deepening to a **fixed
  budget of 150,000 nodes per move** (never a clock), quiescence search
  on captures, promotions and check evasions, **check extensions**
  (+1 ply when in check), a transposition table, killer moves,
  persistent root ordering and MVV-LVA capture ordering.
- **Evaluation:** incremental material + Michniewski-style piece-square
  tables; kings use the middlegame table.
- **Determinism:** same positions give the same moves; the node budget
  is fixed, so trials replay move-for-move.

## Local practice results (150k nodes + check extensions)

| trial | W | D | L | Elo |
|-------|---|---|---|-----|
| 0 | 4 | 2 | 0 | 1654 |
| 1 | 3 | 2 | 1 | 1451 |
| 2 | 3 | 2 | 1 | 1451 |

Mean Elo **1519** across the three practice trials (house baseline
sunfish-4k averages ~1365 on the same trials). Average move ~3 s,
worst observed ~4.4 s, well inside the 10 s per-move limit; a trial
takes ~650–840 s of the 1,800 s wall budget.

## Build

```sh
node <kit>/cli/makefun.mjs compress src.py --out main.py
```

`main.py` must stay under 4,096 bytes (currently 3,072). The packed file
was verified by running a full practice trial on it: trial 0, Elo 1654
(4W 2D 0L), identical to the uncompressed engine. `src.py` lives
in this folder but is not committed. MIT licensed.
