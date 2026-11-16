# Ember Prime — tiny-chess-01

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/Kojeck86/chess-entry-10-repo/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/Kojeck86/chess-entry-10-repo/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

A 4 KB Python chess engine for the [tiny-chess-01](https://make.fun/bounty/tiny-chess-01) bounty on make.fun: one `main.py`, at most 4,096 bytes, Python 3.13 standard library only, deterministic, fixed node budget, no clock.

## Design (v7 "Ember Prime")

- **Board**: 0x88, signed pieces, Zobrist hashing, incremental tapered eval.
- **Search**: principal variation search (PVS) with a persistent transposition table, null-move pruning (R=2, depth-3), late-move reductions (gentle: reduce 1, or 2 for very late moves at depth 4+), check extensions (+1 ply), futility pruning at depth 1–2, in-search repetition draws, two killers per ply, history heuristic.
- **Quiescence**: captures only, MVV-LVA ordering, delta pruning, check evasions. No SEE — SEE pruning in v6 did not help (v6 ≈ v5).
- **Eval**: incremental tapered material + piece-square tables (separate middlegame/endgame tables), bishop pair, king pawn shield, tempo. All eval terms are maintained incrementally on make/unmake, so `ev()` is O(1). (Pawn structure was tried during development and dropped — it cost ~280 packed bytes for ~15 Elo.)
- **Budget**: 40,000 nodes per move, iterative deepening 1–8. No wall clock anywhere.

## History

| Version | Practice (local, 3 trials) | Server |
| --- | --- | --- |
| v4 | — | 997 (rejected, floor 1,154) |
| v5 | ~1,018 | not submitted |
| v6 "Ember SEE" | ~1,003 | 1,099 (rejected, round 5) |
| v7 "Ember Prime" | 1,364 / 1,654 / 1,545 — mean 1,521 | _pending_ |

## Verification

- Perft: all 6 standard positions, depths 1–4 — OK.
- Eval consistency: incremental eval / hash / pawn counts match from-scratch recomputation over 30 random games — OK; make/unmake round-trip exact — OK.
- Self-play (6 games each, alternating colours, own env harness): vs v6 **5W 1D 0L (5.5/6)** — 5 checkmates (121, 154, 139, 34, 105 plies) + 1 draw (repetition, 104 plies). vs v5 **5W 1D 0L (5.5/6)** — 5 checkmates (63, 113, 56, 117, 92 plies) + 1 draw (repetition, 52 plies).
- Official practice trials (kit scorer, 3×6 games): trial-0 **1,364** (3W 1D 2L), trial-1 **1,654** (5W 0D 1L), trial-2 **1,545** (4W 1D 1L). Mean **1,521** — clears the 1,154 floor by 367. Overall 12W 2D 4L across 18 games. Max think time 4,099 ms (limit 10,000 ms) — no over-time losses.

## License

MIT — see `LICENSE`.
