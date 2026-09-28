# v1.0

- The Lumberer's flame (shared with the Flame Sentry) spawns the two main flame parts that never started: their start-up curves assumed the Cremator's 64 s effect lifetime, but the shared flame's lifetime is 1e10 s. They now start with the Cremator's timing.
- The four main flame parts start at the Cremator's distances from the nozzle (1-4 m instead of 7-15 m).
- The flame no longer hits the Lumberer itself: it uses a copy of its collision layer without the Lumberer's hit-box layer.
- Checks the game.dll and EXE hashes for Steam build 25480438, the effect layout and every changed value, and the physics collision table before writing; writes only to committed private read-write memory.
- Requires Bingus Shared Loader v18 or newer.
