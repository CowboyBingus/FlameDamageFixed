Supports Helldivers 2 Steam build 25480438 / EXE 1.8.46015.0.

Offline: `tests/test_fix.lua` runs in LuaJIT and in the game's own lua51.dll against simulated game
memory (with a synthetic stand-in for the flame effect). It covers the effect lookup through both resource hash chains, the header/layout/value checks
and their refusals, the 16 fixed words (and nothing else) changing, the collision layer row (layer 11
without layer 20, both directions) and its refusals (filter type, changed flame row, layer in use,
write guard), the flame instance lookup, moving only the shared flame's particle systems (other
effects and filters untouched, one memory-protection check per heap region), a mission reload, the
Flame Sentry, and the per-frame API call budgets (idle, present, firing, burst starts, new instances).
`tests/test_constants.py` checks every embedded value against the real flame effects, extracted from a
local installation (not included in this repository).
`tests/test_fix_adapter.lua` exercises the real Windows adapter, including its write guards.

Live research (read-only external reads of a running game, 2026-09-27):
- Systems 0 and 3 of the shared flame had 0 live particles in every sample of every unfixed burst; with
  the spawn fix they held the Cremator's counts (20 and 25-28) and started 0.63-0.65 s and 0.42-0.46 s
  into the burst, the Cremator's 0.62-0.64 s and 0.41-0.44 s.
- The engine code: spawn curves are sampled at elapsed / lifetime (clamped); hknp copies a particle
  system's query filter info into every particle query; particle queries use the world's group filter.

In game with the development lab (recorded play, bugs, 2026-09-27):
- Hits per damage window on bugs: unfixed 4.7; spawn fix 14.9; the Cremator 14.6 (same session).
- 16 bursts alternating the fix with and without the self-hit layer: 222 self-hits and about 375 hull
  damage from the flame without it, 0 and 0 with it; hits per window on bugs 15.5 and 14.7 (4-8 m:
  18.6 and 17.5). The flame's particle systems stayed on the new layer for every burst.

In game with the release build (two recorded sessions with the frame probe, bugs, 2026-09-27):
- The addon logged all three fixes in both sessions.
- Flame systems 0 and 3 peaked at 20 and 27 live particles (0 without the fix); the flame's particle
  systems were on the new layer in 9215 of 9230 samples (the rest: the frames before a new flame
  instance is moved); 0 self-hits in 24.7 s of firing, with 2531 hits on enemies.
- Per-frame cost (probe, normal blocks): 0.0015 ms aboard the ship, 0.0078 ms in hosted missions;
  worst frame 1.05 ms at a burst start (0.3-1.3 ms across all blocks); 0 LuaJIT cache flushes.

Not yet verified in game: the Flame Sentry (it shares the flame and its code path is covered offline).
