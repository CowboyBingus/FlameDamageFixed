# Flame Damage Fixed v1.2

Fixes the Lumberer's flamethrower arm, and the Flame Sentry that shares its flame, so it deals the damage its own data describes and stops burning the weapon that fires it:

- **Two flame parts spawn again.** Each part of a flame effect has a spawn rate multiplied by a start-up curve over the effect's lifetime. The Cremator's flame lives 64 s, so its two main parts start 0.4 and 0.6 s into a burst. The Lumberer's copy of that flame was given a lifetime of 10,000,000,000 s but kept the same curves, so those two parts never started. Together they land about three quarters of the Cremator's hits. They now start with the Cremator's timing. They hit but are not drawn, so the flame still looks like one stream.
- **The flame starts at the Cremator's distances.** The Lumberer's four main flame parts started 7-15 m ahead of the nozzle; they now start 1-4 m ahead, like the Cremator's, so bugs close to the Lumberer are hit too.
- **No more self-hits.** Fire splashing back off close targets hit the Lumberer's own hull, arm and cannon. From its first burst until it is gone, each Lumberer or Flame Sentry shares a private Havok collision group with its own flame, and members of that group skip each other. Everything else, from Chargers to teammates, collides with the flame exactly as in the base game: no collision layer is changed.

v1.0 stopped the self-hits by moving the flame to a copy of its collision layer without the Lumberer's hit-box layer. That layer is also the game's heavy-armour and vehicle layer, so v1.0's flame passed through Chargers, the Factory Strider, tank turrets, the Illuminate dropship and other armoured targets. v1.1 replaces that approach and fixes this.

Damage, fuel, heat and burning values are unchanged, and other flamethrowers (Cremator, FLAM-40, Torcher and the rest) are not affected.

Install [the release ZIP](https://github.com/CowboyBingus/FlameDamageFixed/releases/latest) and [Bingus Shared Loader v18 or newer](https://github.com/CowboyBingus/BingusSharedLoader/releases/latest). Enable and deploy, then restart the game. Keep the shared loader as the winning Wwise startup replacement. Vanilla Plus Megapack also contains this addon as an option; use either the Megapack option or this package, not both.

Steam build **25480438** only: before writing, the addon checks the game.dll and EXE hashes, the flame effect's layout and every value it changes, the physics collision filter and the flame's collision row, and that each private group is free in the game's group allocator. It writes only to committed, private, read-write memory, and on a weapon's own hit-boxes it changes only the group bits (never their layer or the bits the game uses). On anything unexpected it logs the reason and leaves that part alone. If the game's update or a mod loaded before it raises an error, it takes its hit-boxes and flames out of their groups and pauses until the game has run cleanly for 60 frames.

## What it costs per frame

- No Lumberer or Flame Sentry nearby: nothing on most frames; every 0.5 s of game time a weapon scan of 3 memory reads, plus 3 for each flamethrower-type weapon in the mission.
- A Lumberer or Flame Sentry present: one memory read per weapon per frame. Every 0.5 s a weapon that has fired is checked with one bulk read and 7 memory reads (its collision group in the game's group allocator). When a weapon first appears, its hit-boxes are found with a few 64 KB bulk reads per frame over about 25 frames.
- A burst start: about 100 memory reads and one memory-protection check, when the burst's new flame joins the weapon's group. A weapon's first burst also puts its hit-boxes in the group (one more check and 21 writes for a Lumberer), and the first burst of a mission checks the flame effect (one more check).

Measured in live play with v1.2 (two joined missions): 0.008 ms per frame in missions and 0.003 ms per frame on the ship. v1.1 in recorded play (three Lumberers) cost about 3 bytes of garbage per frame; most burst starts took under 0.5 ms and a Lumberer's first burst 0.8 to 1.7 ms once. The game's shared LuaJIT code cache never flushed; v1.1's compiled code was 64 KB. v1.2's straight-line rarely-run code (burst starts, grouping and releases, the 0.5 s check) now stays interpreted, and the hit-box scan's loop stays compiled: about 35 KB of machine code instead of v1.1's 47 KB in the same offline test (unmeasured in game).

## For other mod authors: Havok system groups

Flame Damage Fixed puts each Lumberer's or Flame Sentry's own hit-boxes and flame into a private Havok system group (bits 21-31 of a body's or particle system's collision filter info), taken from the top of the game's 2048-group pool: 2047 first, then down to 2040. Members of one non-zero group skip the collision layer table and only their subsystem bits decide, so two mods sharing a group change what collides with what. The game hands groups out lowest first, to ragdolls (at most 144 were in use, highest group 154, in busy missions).

The mod never claims a group from the game's allocator and never writes it. It counts a group as its own only while the allocator still marks it free, none of its other weapons holds it, and no other live body carries it. It checks the allocator at every burst and every 0.5 s while a weapon is grouped, and the bodies whenever it scans a weapon's hit-boxes. When a group stops being its own alone, the weapon's hit-boxes and flames leave it at once and move to another free group at the next burst, or keep the game's own collision when none is left.

If your mod needs private system groups on build 25480438:

- Do not claim groups through the game's allocator (`exe+0x7E7F90`/`exe+0x7E7FD0` or its bitmap): the engine treats every claimed group as a ragdoll group, and putting other bodies in one crashed the game (`helldivers2.exe+0x16c768`, tested 2026-10-04).
- Take them from 2039 downward, never from 2040-2047, so you do not share one with this mod.
- Before using a group, check that the game's allocator still marks it free (`*(exe+0x27C5B88)`: +0 capacity 2048, +4 number of summary words, +8 bitmap: the summary words, then one bit per group, set = free) and that no live body carries it (body records of 160 bytes from `*(world+0x38)`; live when the flags at +68 have bit 0 or 1 set; filter info at +108).
- Change only bits 21-31 of filter infos you own, and clear them when you are done.

## Build and test

Clone [Bingus Shared Loader](https://github.com/CowboyBingus/BingusSharedLoader) beside this repository (or set `BINGUS_SHARED_LOADER` to its path). The tests need LuaJIT (`HD2_LUAJIT`, or `luajit` on `PATH`) and the installed game's `bin/lua51.dll` (set `HD2_LUA51_DLL` if the game is not in the default Steam folder).

- `python -B scripts/build.py` runs the tests in LuaJIT and in the game's own `lua51.dll` (`tests/game_lua.py`) and builds `releases/Flame-Damage-Fixed-v1.2.zip`. The addon is one plaintext entry assembled from the vendored Bingus Shared Runtime files (`src/bingus_runtime.lua`, `src/bingus_memory.lua`) and `src/flame_damage_fixed.lua`; the mod installs the runtime's update guard and checks the game build through its shared hash cache.
- `tests/test_fix.lua` drives the fix against simulated game memory, with synthetic stand-ins for the flame effect and the physics world, and pins every frame's API calls.
- `tests/test_guard_diff.lua` shows the shared runtime's update guard behaves the same as the hand-written one the mod used to carry, over many generated error sequences.
- `tests/test_ffi_names.lua` runs the real Windows adapter after another mod declared the same Windows functions first (SDK-style or wrong prototypes, `tests/hostile_vm.lua`) and before another mod declares them with its own prototypes.
- `tests/test_constants.py` checks the embedded values against the real flame effects (`scripts/patch_table.py`). This repository contains no game files: extract both effects from your own installation into `data/` with `hd2-resource-extract` from [Better Stratagem Bounce](https://github.com/CowboyBingus/BetterStratagemBounce), and the build runs this check too:

  ```
  hd2-resource-extract -type particles -name 0xe3d15622a42863c4 -out data/shared_flame_e3d15622a42863c4.particles
  hd2-resource-extract -type particles -name 0xa4f17daba8ecd8e5 -out data/cremator_flame_a4f17daba8ecd8e5.particles
  ```
- `tests/live_check.lua <root> <pid> <game.dll base> <exe base>` runs the fix's lookups and checks against a running game from another process, read-only.

[Changes](CHANGELOG.md) · [Release notes](docs/RELEASE_NOTES.md) · [Validation](docs/VALIDATION.md)

**AI disclosure:** Claude Opus 5.5 assisted with research, implementation, tests and documentation.

## License

Zero-Clause BSD (0BSD): use, copy, modify and distribute for any purpose, with no conditions. See `LICENSE`.
