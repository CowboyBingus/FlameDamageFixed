# Flame Damage Fixed v1.0

Fixes the Lumberer's flamethrower arm, and the Flame Sentry that shares its flame, so it deals the damage its own data describes and stops burning the Lumberer:

- **Two flame parts spawn again.** Each part of a flame effect has a spawn rate multiplied by a start-up curve over the effect's lifetime. The Cremator's flame lives 64 s, so its two main parts start 0.4 and 0.6 s into a burst. The Lumberer's copy of that flame was given a lifetime of 10,000,000,000 s but kept the same curves, so those two parts never started. Together they land about three quarters of the Cremator's hits. They now start with the Cremator's timing.
- **The flame starts at the Cremator's distances.** The Lumberer's four main flame parts started 7-15 m ahead of the nozzle; they now start 1-4 m ahead, like the Cremator's, so bugs close to the Lumberer are hit too.
- **No more self-hits.** Fire splashing back off close targets hit the Lumberer's own hull, arm and cannon. The flame now uses a copy of its physics collision layer that leaves out the Lumberer's hit-boxes (a layer nothing else uses), so it passes through the Lumberer and hits everything else as before.

Damage, fuel, heat and burning values are unchanged, and other flamethrowers (Cremator, FLAM-40, Torcher and the rest) keep their normal collision layer.

Install [the release ZIP](https://github.com/CowboyBingus/FlameDamageFixed/releases/latest) and [Bingus Shared Loader v18 or newer](https://github.com/CowboyBingus/BingusSharedLoader/releases/latest). Enable and deploy, then restart the game. Keep the shared loader as the winning Wwise startup replacement. Vanilla Plus Megapack v33 also contains this addon as an option; use either the Megapack option or this package, not both.

Steam build **25480438** only: the addon checks the game.dll and EXE hashes, the flame effect's layout and every value it changes, and the physics collision table, before writing. It writes only to committed, private, read-write memory. On anything unexpected it logs the reason and leaves that part alone.

## What it costs per frame

- No Lumberer or Flame Sentry nearby: nothing on most frames; every 0.5 s of game time a weapon scan of 3 memory reads, plus 3 for each flamethrower-type weapon in the mission.
- A Lumberer or Flame Sentry present: one memory read per weapon per frame; a few more while one is firing and for 1 s after.
- The start of a burst: about 200 memory reads, and one memory-protection check for each new flame instance (normally one per burst). The first burst of a mission adds one more check, and the first burst after starting the game one more.

Measured in recorded play: 0.0015 ms per frame aboard the ship and 0.008 ms per frame on average in missions with the Lumberer firing. The frame that starts a burst takes 0.3 to 1.3 ms once: that is the memory-protection check before the new flame moves to its own collision layer. The game's shared LuaJIT code cache never flushed.

## Build and test

Clone [Bingus Shared Loader](https://github.com/CowboyBingus/BingusSharedLoader) beside this repository (or set `BINGUS_SHARED_LOADER` to its path). The tests need LuaJIT (`HD2_LUAJIT`, or `luajit` on `PATH`) and the installed game's `bin/lua51.dll` (set `HD2_LUA51_DLL` if the game is not in the default Steam folder).

- `python -B scripts/build.py` runs the tests in LuaJIT and in the game's own `lua51.dll` (`tests/game_lua.py`) and builds `releases/Flame-Damage-Fixed-v1.0.zip`.
- `tests/test_fix.lua` drives the fix against simulated game memory, with a synthetic stand-in for the flame effect, and pins every frame's API calls.
- `tests/test_constants.py` checks the embedded values against the real flame effects (`scripts/patch_table.py`). This repository contains no game files: extract both effects from your own installation into `data/` with `hd2-resource-extract` from [Better Stratagem Bounce](https://github.com/CowboyBingus/BetterStratagemBounce), and the build runs this check too:

  ```
  hd2-resource-extract -type particles -name 0xe3d15622a42863c4 -out data/shared_flame_e3d15622a42863c4.particles
  hd2-resource-extract -type particles -name 0xa4f17daba8ecd8e5 -out data/cremator_flame_a4f17daba8ecd8e5.particles
  ```
- `tests/live_check.lua <root> <pid> <game.dll base> <exe base>` runs the fix's lookups and checks against a running game from another process, read-only.

[Changes](CHANGELOG.md) · [Release notes](docs/RELEASE_NOTES.md) · [Validation](docs/VALIDATION.md)

**AI disclosure:** Claude Opus 5.5 assisted with research, implementation, tests and documentation.
