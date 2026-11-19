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
  check), late-move reductions with null-window re-search, transposition
  table with mate-distance adjustment, killer + history ordering, MVV-LVA
  captures, repetition detection.
- **Evaluation:** material (100/320/330/500/900) + piece-square tables
  from compact formulas (pawn advance, centre bonuses, rook 7th rank, king
  middlegame/endgame blend) + bishop pair (+30) + passed-pawn bonuses
  (up to +110 on the 7th rank).
- **Determinism:** every move derives only from the position, the game
  history, and the trial seed. No randomness at play time, no network calls
  except the env API, no wall-clock use in search.

## Packing

`main.py` (3,960 bytes) is the engine minified (9,226 bytes) and
lzma-compressed, stored as raw latin-1 characters in a string literal.
Readable source, tests (perft vs python-chess: start 20/400/8902/197281,
kiwipete 48/2039/97862) and packer live outside this repo to keep the entry
tree under the 8 KB cap.

## Measured strength

Practice trials on the local env server (6 games each, scorer-computed):
trial 1 (50k nodes) 3W 0D 3L, Elo 1,279; trial 2 (40k nodes) 3W 2D 1L,
Elo 1,451 — mean 1,365, level with the house baseline (sunfish 4k).
The v2 engine (LMR + passed pawns) was validated by 20-game self-play
against v1 before submission; see the build log for the numbers. Max think
9.0 s per move at 40k nodes (limit 10 s); the 100k budget lost every game
on time when the CPU throttled, which is why the budget is fixed at 40k.

## Files

- `main.py` — the entry (3,960 bytes)
