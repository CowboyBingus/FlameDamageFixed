# Flame Damage Fixed v1.1

Fixes the Lumberer's flamethrower arm, and the Flame Sentry that shares its flame, so it deals the damage its own data describes and stops burning the weapon that fires it:

- **Two flame parts spawn again.** Each part of a flame effect has a spawn rate multiplied by a start-up curve over the effect's lifetime. The Cremator's flame lives 64 s, so its two main parts start 0.4 and 0.6 s into a burst. The Lumberer's copy of that flame was given a lifetime of 10,000,000,000 s but kept the same curves, so those two parts never started. Together they land about three quarters of the Cremator's hits. They now start with the Cremator's timing. They hit but are not drawn, so the flame still looks like one stream.
- **The flame starts at the Cremator's distances.** The Lumberer's four main flame parts started 7-15 m ahead of the nozzle; they now start 1-4 m ahead, like the Cremator's, so bugs close to the Lumberer are hit too.
- **No more self-hits.** Fire splashing back off close targets hit the Lumberer's own hull, arm and cannon. From its first burst until it is gone, each Lumberer or Flame Sentry shares a private Havok collision group with its own flame, and members of that group skip each other. Everything else, from Chargers to teammates, collides with the flame exactly as in the base game: no collision layer is changed.

v1.0 stopped the self-hits by moving the flame to a copy of its collision layer without the Lumberer's hit-box layer. That layer is also the game's heavy-armour and vehicle layer, so v1.0's flame passed through Chargers, the Factory Strider, tank turrets, the Illuminate dropship and other armoured targets. v1.1 replaces that approach and fixes this.

Damage, fuel, heat and burning values are unchanged, and other flamethrowers (Cremator, FLAM-40, Torcher and the rest) are not affected.

Install [the release ZIP](https://github.com/CowboyBingus/FlameDamageFixed/releases/latest) and [Bingus Shared Loader v18 or newer](https://github.com/CowboyBingus/BingusSharedLoader/releases/latest). Enable and deploy, then restart the game. Keep the shared loader as the winning Wwise startup replacement. Vanilla Plus Megapack v34 also contains this addon as an option; use either the Megapack option or this package, not both.

Steam build **25480438** only: before writing, the addon checks the game.dll and EXE hashes, the flame effect's layout and every value it changes, the physics collision filter and the flame's collision row, and that each private group is free in the game's group allocator. It writes only to committed, private, read-write memory, and on a weapon's own hit-boxes it changes only the group bits (never their layer or the bits the game uses). On anything unexpected it logs the reason and leaves that part alone.

## What it costs per frame

- No Lumberer or Flame Sentry nearby: nothing on most frames; every 0.5 s of game time a weapon scan of 3 memory reads, plus 3 for each flamethrower-type weapon in the mission.
- A Lumberer or Flame Sentry present: one memory read per weapon per frame. Every 0.5 s a weapon that has fired is checked with one bulk read. When a weapon first appears, its hit-boxes are found with a few 64 KB bulk reads per frame over about 25 frames.
- A burst start: about 100 memory reads and one memory-protection check, when the burst's new flame joins the weapon's group. A weapon's first burst also puts its hit-boxes in the group (one more check and 21 writes for a Lumberer), and the first burst of a mission checks the flame effect (one more check).

Measured in recorded play (joined missions with three Lumberers): 0.008 ms per frame on average in missions and 0.002 ms per frame aboard the ship, about 3 bytes of garbage per frame. Most burst starts take under 0.5 ms; a Lumberer's first burst takes 0.8 to 1.7 ms once. The game's shared LuaJIT code cache never flushed; the addon's compiled code is 64 KB.

## Build and test

Clone [Bingus Shared Loader](https://github.com/CowboyBingus/BingusSharedLoader) beside this repository (or set `BINGUS_SHARED_LOADER` to its path). The tests need LuaJIT (`HD2_LUAJIT`, or `luajit` on `PATH`) and the installed game's `bin/lua51.dll` (set `HD2_LUA51_DLL` if the game is not in the default Steam folder).

- `python -B scripts/build.py` runs the tests in LuaJIT and in the game's own `lua51.dll` (`tests/game_lua.py`) and builds `releases/Flame-Damage-Fixed-v1.1.zip`.
- `tests/test_fix.lua` drives the fix against simulated game memory, with synthetic stand-ins for the flame effect and the physics world, and pins every frame's API calls.
- `tests/test_constants.py` checks the embedded values against the real flame effects (`scripts/patch_table.py`). This repository contains no game files: extract both effects from your own installation into `data/` with `hd2-resource-extract` from [Better Stratagem Bounce](https://github.com/CowboyBingus/BetterStratagemBounce), and the build runs this check too:

  ```
  hd2-resource-extract -type particles -name 0xe3d15622a42863c4 -out data/shared_flame_e3d15622a42863c4.particles
  hd2-resource-extract -type particles -name 0xa4f17daba8ecd8e5 -out data/cremator_flame_a4f17daba8ecd8e5.particles
  ```
- `tests/live_check.lua <root> <pid> <game.dll base> <exe base>` runs the fix's lookups and checks against a running game from another process, read-only.

[Changes](CHANGELOG.md) · [Release notes](docs/RELEASE_NOTES.md) · [Validation](docs/VALIDATION.md)

**AI disclosure:** Claude Opus 5.5 assisted with research, implementation, tests and documentation.
