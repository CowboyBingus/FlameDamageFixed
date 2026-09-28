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
