# v1.2

- Windows functions are declared under private names, so another mod that declares them with other prototypes, before or after this one, can no longer stop this mod from starting or change how that mod's calls work.
- A Lumberer or Flame Sentry that disappears while its hit-boxes are being looked up again now leaves its private collision group; before, its hit-boxes could stay in it.
- A weapon's private collision group is checked at every burst and every 0.5 s. If the game no longer marks it free or other bodies carry it, the weapon moves to another free group, or keeps the game's own collision when none is left.
- When the game's update or a mod loaded before this one raises an error, the mod takes its hit-boxes and flames out of their groups and pauses until 60 clean frames; before, it kept running.
- Eight errors below it, or eight of its own, in one burst stop the mod for the session and take its hit-boxes and flames out of their groups. Before, its own errors stopped it after 8 per session.
- The log gets one line per error burst, and the first failure is kept in the mod's status at shutdown.
- The update guard comes from Bingus Shared Runtime with the same policy, and the game build is checked through its session-wide hash cache, so the game files are hashed once for every mod.
- Rarely-run code (burst starts, grouping, releases and the 0.5 s check) stays interpreted while the hit-box scan stays compiled, using about 35 KB of the shared LuaJIT code cache instead of 47 KB in the same offline test.
- README.md describes the private-group convention for other mod authors.
- Licensed under the Zero-Clause BSD license (0BSD).

# v1.1

- Fixes v1.0's flame passing through Chargers, the Factory Strider, tank turrets, the Illuminate dropship and other armoured targets. v1.0 moved the flame to a copy of its collision layer without layer 20, which is not only the Lumberer's hit-box layer but the game's heavy-armour and vehicle layer.
- New self-hit fix, with no collision layer changed: from its first burst until it is gone, each Lumberer or Flame Sentry and its own flame share a private Havok system group (2047 down to 2040, checked free in the game's allocator), whose members skip each other. Only the group bits of the weapon's own hit-boxes change; every other body keeps its base-game collision with the flame.
- The two restored flame parts are not drawn (visualizer count 1 -> 0, like the game's own invisible carrier part): they still hit, but the flame no longer shows a second short, wide cone near the nozzle.
- Lower per-frame garbage (about 3 bytes per frame in missions) and fewer reads while firing.
- Requires Bingus Shared Loader v18 or newer.

# v1.0

- The Lumberer's flame (shared with the Flame Sentry) spawns the two main flame parts that never started: their start-up curves assumed the Cremator's 64 s effect lifetime, but the shared flame's lifetime is 1e10 s. They now start with the Cremator's timing.
- The four main flame parts start at the Cremator's distances from the nozzle (1-4 m instead of 7-15 m).
- The flame no longer hits the Lumberer itself: it uses a copy of its collision layer without the Lumberer's hit-box layer.
- Checks the game.dll and EXE hashes for Steam build 25480438, the effect layout and every changed value, and the physics collision table before writing; writes only to committed private read-write memory.
- Requires Bingus Shared Loader v18 or newer.
