Supports Helldivers 2 Steam build 25480438 / EXE 1.8.46015.0.

# v1.1

Offline: `tests/test_fix.lua` runs in LuaJIT and in the game's own lua51.dll against simulated game memory,
with synthetic stand-ins for the flame effect and the physics world (units, bodies, collision filter, group
allocator, flame instances). It covers the effect lookup, layout and value checks and their refusals, the 18
fixed words and nothing else, the collision filter and flame-row checks, the weapon family walk (the seated
pilot's subtree excluded), the body scan spread over frames, grouping only the weapon's own flame-hittable
bodies (only bits 21-31 change; hit-boxes with game data in their subsystem bits are skipped), a private group
per weapon checked free in the allocator, the burst's flame instance joining the group, keeping the group
between bursts, a rebuilt hit-box, a destroyed vehicle (group released), the weapon leaving, a mission reload,
the Flame Sentry, refusals (no free group, unknown filter, family too large, unwritable memory), zero garbage on
every steady path, and the per-frame API call budgets. `tests/test_fix_adapter.lua` exercises the real Windows
adapter, including its write guards and bulk reads. `tests/test_constants.py` checks every embedded value
against the real flame effects, extracted from a local installation (not included in this repository).

Engine research (decompiled EXE and read-only reads of a running game): particle queries run the world's
group filter after their own body checks, and the engine registers every particle system with group 0, so a
flame has no built-in owner exclusion. Two bodies in the same non-zero group collide only if their subsystem
bits allow it, whatever their layers; the Lumberer's flame-hittable hit-boxes (hull, arm, cannon) all have
subsystem 0, so a shared group excludes exactly them. The game hands system groups out lowest first and only
ragdolls use them; in play it never used more than 154, and 2040-2047 stayed free.

In game with a development lab (recorded play, 2026-09-28, 22 bursts alternating the group fix with base-game
collision): 0 self-hits with the fix, 97 without (hull 80, cannon 14, arm 3); acid Chargers' layer-20
hit-boxes were hit either way (1933 and 1663 hits); only the Lumberer's 21 hit-boxes ever joined the group.

In game with release candidates (recorded play, 2026-09-28):
- Bugs: 0 self-hits; 2014 hits on Chargers' layer-20 hit-boxes; the two restored parts still spawned
  (at most 20 and 29 live particles) and landed the most hits; the flame's particle systems were in the group
  in 23169 of 23190 samples (the rest: the frames before a new flame instance joins). The flame looked like the
  base game's single stream.
- Release build, three Lumberers in one joined mission (about 20 minutes, with the frame probe): each
  Lumberer's 21 hit-boxes joined a group of their own at its first burst (2047, 2046, 2045) and stayed there
  through every burst until the mission ended. 4,822 hits on layer-20 hit-boxes (acid Chargers 3,082, Chargers
  1,120, Impalers 620). The only hits on the firing Lumberer came while the development probe was skipping every
  mod's update (85) or from a flame started in such a window (2). The seated pilot's hit-boxes were touched by
  flame 12 times near the nozzle (they cannot share the group: they carry game data in their subsystem bits);
  no damage or burning was seen in play over three Lumberers' worth of fuel.
- Per-frame cost (probe, normal blocks): 0.0080 ms in joined missions, 0.0018 ms aboard the ship, about
  3 bytes of garbage per frame. Worst frames: 1.69, 0.99 and 0.84 ms at the three Lumberers' first bursts;
  in 99% of seconds no frame of the addon exceeded 0.49 ms; one unexplained 1.49 ms frame. All mod hooks
  together cost -0.23 ms per frame in joined missions (90% interval -0.53 to +0.07 ms, 31 cycles; interim).
  No LuaJIT code cache flush (whole game at most 1.6 MB of machine code in 1,001 traces); the addon's own
  compiled code grew to 64 KB in 58 traces (v1.0: 9.3 KB in 9 traces).

Not yet verified in game: the Flame Sentry (it shares the flame and its code path is covered offline).

# v1.0

Offline: `tests/test_fix.lua` covered the effect lookup through both resource hash chains, the
header/layout/value checks and their refusals, the 16 fixed words, the collision layer row (layer 11 without
layer 20) and its refusals, the flame instance lookup, moving only the shared flame's particle systems, a
mission reload, the Flame Sentry, and the per-frame API call budgets.

Live research (read-only external reads of a running game, 2026-09-27):
- Systems 0 and 3 of the shared flame had 0 live particles in every sample of every unfixed burst; with
  the spawn fix they held the Cremator's counts (20 and 25-28) and started 0.63-0.65 s and 0.42-0.46 s
  into the burst, the Cremator's 0.62-0.64 s and 0.41-0.44 s.
- The engine code: spawn curves are sampled at elapsed / lifetime (clamped); hknp copies a particle
  system's query filter info into every particle query; particle queries use the world's group filter.

In game with the development lab (recorded play, bugs, 2026-09-27):
- Hits per damage window on bugs: unfixed 4.7; spawn fix 14.9; the Cremator 14.6 (same session).
- 16 bursts alternating the fix with and without the self-hit layer: 222 self-hits and about 375 hull
  damage from the flame without it, 0 and 0 with it; hits per window on bugs 15.5 and 14.7.

In game with the release build (two recorded sessions with the frame probe, bugs, 2026-09-27): the addon
logged all three fixes; 0 self-hits in 24.7 s of firing, with 2531 hits on enemies; 0.0015 ms per frame
aboard the ship, 0.0078 ms in hosted missions, worst frame 1.05 ms at a burst start.

Found after release: layer 20, which v1.0's layer copy left out, is the game's heavy-armour and vehicle
hit-box layer. 54 physics resources use it, and for 30 of them (every Charger, the Factory Strider, tank
turrets, the Illuminate dropship and more) it is the only layer the flame can hit, so v1.0's flame passed
through them (a recording had 0 hits on four Chargers in its path for 9.9 s). The v1.0 tests had checked the
new layer but never split hits by enemy type. v1.1 replaces the layer copy.
