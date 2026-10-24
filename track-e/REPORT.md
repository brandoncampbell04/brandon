# Track E: graphics pass — report to Brandon

**Branch:** `cloud/art` (head `7b5d1b0`), 18 commits on top of `main` `2a2a2d0`.
**Bundle:** `art.bundle` = `git bundle create art.bundle main..cloud/art`. It verifies OK and is 49 MB, mostly evidence PNGs. Nothing was pushed to `main`.

Restore with:
```
git fetch art.bundle cloud/art:cloud/art
```

## Summary
1. **E1 (art-director):** wrote the Highcut look target, `art/direction/highcut-look.md` (v1.1 after E4). **Ruling: no sodium lamps.** Sodium light is orange-amber, which falls inside the players-only Sync Amber band, so Highcut keeps cold mercury-green work lamps. Weapon effects carry no amber either.
2. **E2 (technical-artist):** Highcut now uses 7 procedural, UV-free shaders:
   - granite with strata and cracks
   - jointed concrete
   - rusted and riveted Night Blue steel
   - cover materials
   - houses with doors
   - out-of-bounds blocks with windows
   - a slate sky

   The pass also adds 1,828 non-colliding props in 14 MultiMeshes (crates, ore carts and bins, conduit, cables, scaffolds, lattice braces, flush ore rail, lamp cowls), soft-key lighting, fog with height fog, SSAO, and AgX tuned in `look_default.tres`. The navmesh is byte-identical and no collision shape changed.
3. **E3 (technical-artist):** 14 standalone VFX scenes share one API: `play` / `stop` / `seek` plus a `finished` signal. They are deterministic and shader-driven:
   - weapons and impacts: rifle and mortar flash, tracer, impacts by surface (granite, concrete, steel, dirt), mortar trail, arc and explosion
   - hits: hit spark, weak-point spark
   - Hollow: Blank lunge telegraph, Blank erase dissolve, rift node
   - objective and UI: anchor beam with damage pulse, shader sync overlay

   There is also a gallery scene, a capture sheet, 29 tests, and a hook table for owners in `game/vfx/README.md`.
4. **E4 (art-director):** SIGNED OFF WITH NOTES (`art/direction/reviews/track-e-signoff.md`), with no blocking notes. One fix round went back to E2 and E3 and closed every follow-up the technical-artist could fix:
   - The route is now 0.10 V lighter than the floor.
   - A new view shows half vs full cover.
   - The rifle and weak-spark white-band peaks were removed.
   - The weak-spark burst clears the Blank core by frame 3.
5. **Gates:** a fresh-context QA pass ran on each batch (E1, E2+E3, and the fix round), and all three passed. `ci.sh` passes 6/6 after every merge and on the final head.

## Evidence
- **`highcut_before_after.png`:** the overview plus 5 beat views (push_wake, push_tier2, hold, hold_cover, seal), before and after. The originals are in `game/vfx/dev/captures/highcut/`. They were rendered under Xvfb + lavapipe; the log names `llvmpipe` and there are 0 `SHADER ERROR` lines.
- **`vfx_sheet.png`:** 25 effect shots, originals in `game/vfx/gallery/captures/`. At 30 m the Blank's core stays visible: core V .79 against torso V .11, where the rule needs a difference of at least .50.
- **Reserved hues:** the after frames have 0 amber pixels and 0 white-band pixels, and 0% of pixels sit at V >= .95. Minimum V per frame is .145 to .263, which meets the shadow floor of at least .14. Amber appears only in the sync overlay.
- **Final CI:**
  ```
  [1/6] PASS import   [2/6] PASS test_runner   [3/6] PASS levels_validate
  [4/6] PASS net_selftest   [5/6] PASS registry_sync   [6/6] PASS retired_theme
  CI: PASS (6/6 steps)
  ```
- **levels_validate:** `LEVELS VALIDATE: PASS (44 checks, 0 failed)`.
- **Perf** (`docs/perf/highcut-art-pass.md`, 1080p overview): draw calls 66 → 99, primitives 13k → 98k, unique materials 10 → 18. These counts are not timings; GPU timings need real hardware.

## Requests for other owners (not done, outside Track E paths)
- **Combat (combat-engineer):** wire the effects in using the hook table in `game/vfx/README.md`. The table covers the viewmodel flash, the tracer, impacts, hit and weak sparks, the mortar trail, arc and blast, and the Blank telegraph and dissolve. It needs new data: a surface tag and a normal on hits and detonations. Retire `ARC_COLOR #3FA796`, because Garden Teal is reserved for objectives.
- **Level Designer:**
  - wire the anchor beam and its damage pulse, and the rift node glow
  - make `build_levels.gd` deterministic: Godot 4.7 writes random `unique_id`s, so the hub only stays identical because we restore it
  - update the fog figure and palette pointer in `game/levels/README.md`
  - tag collider surfaces
- **Lead Engineer:**
  - swap in `game/vfx/sync_overlay/sync_overlay.tscn`, which has the same API
  - set `FLASH_TICKS` to 15 or less (0.25 s cap)
  - optionally set a 1280x720 default viewport so `--write-movie` frames match the capture size
  - add `-I` to the retired-theme grep in `ci.sh`
- **Performance Engineer:** take GPU timings at quality 1 and 0, and rule on the 6 shadowed omni lights, SSAO and dressing shadows.
- **Art Director:**
  - update the look doc's route value to the new `#97A38F` (V .64)
  - deal with 2 orphan ledger PNGs already on `main` (`game/core/dev/capture_highcut_sleeper.png`, `capture_main_menu_export.png`)
- **Deferred:** dust motes, and the optional pale lamplight windows (skipped to avoid any amber risk).

## Your calls
- **Feel and look verdict:** does the after frame read like Highcut to you?
- **Sodium vs mercury:** sodium lamps are ruled out by the amber rule. If you want warmer lamps anyway, that is a style-bible change.
