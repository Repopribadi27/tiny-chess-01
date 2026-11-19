# Tiny Chess — a 4 KB Python chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/repopribadi27/tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/repopribadi27/tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A chess engine that fits in 4,096 bytes of Python for the make.fun
tiny-chess-01 gauntlet: 8 games per trial against house engines (alpha-beta
depths 2–4 and 100k–2M nodes), rated by Elo anchored to their calibrated
ratings. Written from scratch — no copied engine code.

## Design

- **Board:** 0x88, incremental Zobrist hashing, make/unmake with full state
  restore (castling rights, en passant, halfmove clock, non-pawn material).
- **Search:** negamax alpha-beta with iterative deepening to a fixed
  40,000-node budget per move (never a clock, so replays are byte-identical),
  quiescence search (captures + promotions, all evasions in check),
  null-move pruning (R=3, only when non-pawn material > 2000 and not in
  check), transposition table with mate-distance adjustment, killer +
  history move ordering, MVV-LVA captures, repetition detection.
- **Evaluation:** material (100/320/330/500/900) + piece-square tables
  generated from compact formulas (pawn advance, centre bonuses, rook 7th
  rank, king middlegame/endgame blend) + bishop pair (+30).
- **Determinism:** every move derives only from the position, the game
  history, and the trial seed. No randomness at play time, no network calls
  except the env API, no wall-clock use in search.

## Packing

`main.py` (3,802 bytes) is the engine minified (8,847 bytes) and
lzma-compressed, stored as raw latin-1 characters in a string literal.
Readable source, tests (perft vs python-chess: start 20/400/8902/197281,
kiwipete 48/2039/97862) and packer live outside this repo to keep the entry
tree under the 8 KB cap.

## Measured strength (local env server, practice trials, 6 games)

| Trial | Settings | Result | Elo |
|-------|----------|--------|-----|
| 0 | 30k nodes, no null-move, TT cleared/move | 2W 0D 4L | ~1,100 |
| 0b | 100k nodes, TT persisted | 0W 0D 5L (all by time) | — |
| 0c | 50k nodes + null-move, TT persisted | 3W 1D 2L | ~1,365 |
| 1 | 50k nodes + null-move (packed main.py) | 3W 0D 3L | 1,279 |
| 2 | 40k nodes + null-move (packed main.py) | 3W 2D 1L | 1,451 |

Trials 1–2 are scorer-computed; trial 2 beat the nodes-500k house engine.
Mean of the two scorer trials is 1,365 — level with the house baseline
(sunfish 4k, ~1,365 Elo on 6-game practice trials), so the entry is
competitive with the baseline but not yet clearly above it. Max think time
9.0 s per move at 40k (limit 10 s, mean 2.4 s); the 100k budget lost every
game on time when the CPU throttled to ~9–13k nodes/s, which is why the
budget is fixed at 40k: depth 4 in the middlegame completes at ~25k nodes,
so 40k loses almost nothing versus 50k.

## Files

- `main.py` — the entry (3,802 bytes)
