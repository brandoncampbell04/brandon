# Track F (characters): Producer report

**To:** Brandon · **From:** Producer, Track F · **Date:** 2026-10-24
**Branch:** `cloud/characters`, built from `main` at `2a2a2d0`. Head is `83797bc`: 10 commits, 69 files, all of them in `game/characters/` except 7 lines appended to `art/licences.md`.
**Verdict:** F1, F2 and F3 are done. QA passed all 3 batches. The Art Director's F4 review signed off with fix notes, and the one round of fixes is merged. CI is green at 6/6.

## What's in this folder
| File | What it is |
|---|---|
| `characters.bundle` | `git bundle create characters.bundle main..cloud/characters`. The result of this track. |
| `art-director-f4-review.bundle` | Optional, **not** merged into `cloud/characters` (see Decision 2). It holds the F4 sign-off note and a ledger Terms-table ruling. It's built on `cloud/characters` (`cloud/characters..art-director/f4-review`). |
| `F4-art-director-signoff.md` | The Art Director's sign-off note, as a plain file. |
| `captures/sleeper/` | Turntable contact sheets for all 4 looks; the 40 m chest and shoulder emblem shots; first-person views with the rifle and the mortar; third-person preview frames; `summary.txt` with the detector and emblem-check output. |
| `captures/animation/` | A contact sheet and a GIF for each of 15 states, plus `contact_all_states.png` and `summary.txt`. |
| `captures/blank/` | 12 Blank v2 states at 3 m and at 30 m, plus `sheet_3m.png` and `sheet_30m.png`. |

To apply the bundle: `git fetch characters.bundle cloud/characters:cloud/characters`, then merge into `main` and run `bash tools/harness/ci.sh`.

## Summary
1. **Sleeper body (F1).** A segmented humanoid built in code: head, torso, limbs, jacket shell, pack and shoulder pads, with pivots at the joints. It wears `echo_look` in all 4 looks through `set_look(id)`. The amber emblem shows on the chest, the left shoulder and the wrist (first person), and it reads at 40 m on both chest and shoulder.
2. **First-person arms and third-person preview (F1).** `fp_arms.tscn` holds procedural viewmodels of the Lantern rifle and the Breaker mortar. It is a call-for-call replacement for the current `Viewmodel`. The preview scene uses the 3.0 m over-the-shoulder boom.
3. **Animation (F2).** No rig: the poses are procedural, with two-bone IK for legs and arms. It covers idle, walk, sprint, crouch, slide, jump and land, aim, recoil kick, reload, weapon switch and the hard-wake collapse. It's driven by `set_move_state()` and `on_weapon_event()` using the real `MoveState` and `WeaponEvent.Kind` enums. Stepping is deterministic, so the tests run headless. Remote Sleepers are driven from the replicated `EF_SLEEPER_*` flags.
4. **Blank v2 (F3).** Shadow-black mass with a noise-eaten, crawling white edge, a face slit and a white core disc. It uses only `#F7F7F5` and `#111113`, never amber. It's a drop-in for `BlankView` (same API). Every state is animated, and each duration is read from `BlankData`.
5. **Everything is procedural.** There are no imported meshes or animations and no AI generators. The ledger lines read `procedural, project-owned, placeholder-quality`.

## Evidence (exact commands, run on `cloud/characters` at `83797bc`)
- `bash tools/harness/ci.sh`: steps `[1/6]` to `[6/6]` all PASS, ending `CI: PASS (6/6 steps)`.
- `godot --headless --path . --script res://tests/runner.gd -- --filter=characters`: `RESULT: PASS (72 passed, 0 failed, 0 skipped)`. Three engine ERROR lines come from bad-input tests that trigger them on purpose.
- **Hitbox match, required to be within 5 cm.** The tests read the capsules from `movement_tuning.tres`, `sleeper.tscn` and `HitTarget.blank()` at run time.
  - Sleeper standing: 1.795 m tall, 0.405 m half-width, against a capsule of 1.8 m × 0.4 m.
  - Sleeper crouched: 1.110 m and 0.400 m, against 1.1 m.
  - Every animation cycle is sampled and stays inside its capsule. Sprint peaks at 1.757 m.
  - Blank: 1.904 m tall and 0.685 m wide, against 1.900 m and 0.700 m. The core sits at 1.300 m with a 0.32 m diameter, matching the weak point exactly.
- **Blank timings against §3.4, tested.** Lunge wind-up 0.60 s, bolt wind-up 0.40 s, recover 1.2 s, stagger 0.4 s, dissolve 0.6 s, blink afterimage 0.15 s, plus `emerge_s` 0.5 s from the data. Changing `blank.tres` makes the test fail.
- **Captures.** All were rendered on lavapipe under Xvfb. Every log names `llvmpipe (LLVM 20.1.2)`, with no `SHADER ERROR` lines.
  - `capture.sh` reports `SLEEPER CAPTURES: PASS` (62 PNGs, detector: 0 empty or flat frames).
  - `capture_anim.sh` reports `ANIM CAPTURES: PASS` (1,356 frames, at least 957 colours each).
  - `capture_blank_v2.sh` reports `CAPTURE CHECKS: PASS` (24 frames, 0 amber pixels).
- **Emblem at 40 m, 1080p.**
  - Chest: 14 pure-amber pixels (70%) in all 4 looks.
  - Shoulder at 270°: 15–16 pixels (68–73%), contrast 3.8–5.1:1.
  - The pass marks are at least 9 pixels and 50%.
  - The first-person wrist emblem has 96–97 px of margin to the frame edge for both weapons.
- **Negative controls.** QA perturbed every key test and confirmed it then failed: hitbox, timing, palette, recoil (now pinned to `recoil_vertical_deg`), reload length, remote flags, sprint lean and the wrist margin.

## Process record
| Step | Who | Result |
|---|---|---|
| F1 Sleeper body and F3 Blank v2, in parallel worktrees | technical-artist ×2 | merged; the only conflict was both adding ledger lines, and both sets were kept |
| QA batch 1 (`2a2a2d0..3c0ccb9`) | qa-lead, fresh context | **PASS**, 2 medium findings: shoulder emblem never checked at 40 m; loose 40 m pass marks |
| F2 animation, plus fixes for the batch-1 findings | technical-artist | merged |
| QA batch 2 (`3c0ccb9..1892844`) | qa-lead, fresh context | **PASS**, 2 medium findings (wording errors in the swap text) and 1 low (a recoil test that could never fail) |
| F4 review | art-director | **SIGNED OFF WITH FIX NOTES**: 1 must-fix (the mortar wrist emblem was cut off in first person) |
| The one fix round: art-director's must-fix, QA findings, and a sprint lean QA suggested | technical-artist | merged as `83797bc` |
| QA batch 3 (`1892844..83797bc`) | qa-lead, fresh context | **PASS**, 2 low findings and 1 info, listed under open items |

## Swap requests (exact; also in `game/characters/*/README.md`)
**Lead Engineer** (`game/player/sleeper.tscn`, `sleeper.gd`, `game/core/flow/flow_director.gd`):
1. Add `res://game/characters/sleeper/sleeper_body.tscn` as child `Body` of the `Sleeper` root at `position = Vector3(0, -0.9, 0)`. The body's origin is at its feet, while the player root sits at the capsule centre. Set `Body.rotation.y = look_yaw()`; the body faces −Z.
2. Put the body's meshes on a render layer that the local first-person camera's cull mask leaves out. Keep them visible in the hub third-person toggle and on every other Sleeper.
3. On each `stepped(new_state)`, call `$Body.apply_self_state(new_state)`. Optionally also call `$Body.set_move_direction(Vector2(v_local.x, -v_local.z))`.
4. Hard wake: `game/core/flow/flow_director.gd:290` calls `world.drop(slot)` as soon as `hard_wake` arrives. Call `$Body.hard_wake()` first, turn the body's collision off, and delay the drop by `SleeperAnimator.COLLAPSE_S` (1.0 s). `reset_animation()` stands a pooled body back up. *Creative Director needs to check the 1 s collapse against canon.*

**Combat Engineer** (`game/combat/weapon_controller.gd`, `game/combat/hollow/hollow_host.gd`):
1. In `setup_view()`, instance `res://game/characters/sleeper/fp_arms.tscn` under the camera instead of `Viewmodel.new()`, and retype `viewmodel` to `Node3D`. The existing calls then work unchanged:
   - `set_weapon(slot)`
   - `on_fired(kick)`
   - `update_pose(delta, ads, lower)`
   - `muzzle_global()`
   - `hold_flash` has no FpArms equivalent. FpArms draws no muzzle flash.
2. Optional, for the full reload and switch gestures: forward every event in `sim_tick()` to `fp_arms.on_weapon_event(e)`, pass `lower = 0.0`, and drop `on_fired()` (otherwise every shot kicks twice). For the local player, call `fp_arms.share_animator(body.animator)`, and call `body.set_ads_blend(state.ads_blend())` in `_process()`.
3. Blank: in `hollow_host.gd` `_sync_views()`, replace `BlankView.new()` with `(load("res://game/characters/blank/blank_body.tscn") as PackedScene).instantiate() as BlankBody`. Keep `auto_advance = false`, `data = director.data` and `apply_brain(b)` every tick. No offset is needed. Connect `telegraph(kind)` (`lunge_windup`, `bolt_windup`, `blink`) to audio as before. After the swap, point `test_blank_view.gd` at `BlankBody`.

**Network Engineer** (`# NET-OWNED player_remote_view` in `game/player/sleeper.gd`, and `hollow_remote_view`):
1. For each remote Sleeper, take the interpolated sample dictionary that `_net_remote_view` already reads (`smp["flags"]`, `smp["velocity"]`, keys from `net_interpolator.gd`). Each frame, call:
   - `body.apply_remote_flags(smp["flags"], Vector2(smp["velocity"].x, smp["velocity"].z).length())`
   - `body.set_vertical_speed(smp["velocity"].y)`
   `apply_remote_flags` maps the `EF_SLEEPER_*` bits from `game/net/schemas/net_protocol.gd:87-93`. *(This corrects the README's `entity.flags` wording, which QA batch 3 flagged.)*
2. Remote Blanks: from the hook that today drives `BlankView`, call `BlankBody.apply_sample(sample)` with the same dictionary.
3. Server gaps:
   - `Sleeper.to_entity_state()` (`game/player/sleeper.gd:381`) doesn't set `EF_SLEEPER_ADS`, `EF_SLEEPER_RELOADING` or `EF_SLEEPER_FIRING` yet.
   - No weapon slot is replicated, so remote bodies use rifle timings until one is. Then call `body.set_weapon_slot(slot)`.

## Requests to other owners (not swaps)
- **Technical Artist, `game/shaders/echo/` (from F1 and the Art Director):**
  - (a) Add a third emblem slot, 16 cm or larger, on the pack back or the right shoulder pad. The 3.0 m over-right-shoulder boom sees no emblem (`captures/sleeper/tps_back.png`).
  - (b) Draw the emblem as a solid disc once its ring falls below about a pixel, so the strict glyph check in `tools/echo/check_emblem_pixels.py` can pass at 40 m. Today the 40 m verdict uses the track's own distance check (`game/characters/sleeper/dev/check_emblems.py`), which QA reviewed and judged sound.
  - (c) Optional: give `check_emblem_pixels.py` a required-slots argument.
- **Art Director:** amend `sleeper-base-body.md` for the back emblem once (a) is approved. The enlarged shoulder pads (×1.3) were accepted; the commissioned art should read them as soft, rolled pads, not armour.
- **Creative Director:**
  - Check the hard-wake collapse against canon.
  - Look at the Blank's lunge flare, which nearly hides the body at peak at 3 m (Art Director note L4). Not blocking.

## Decisions for Brandon
1. **`OWNERS` has no line for `game/characters/`**, so by the file it falls to producer. Suggest adding `/game/characters/  technical-artist`, which is a Producer edit that you approve. QA allowed it because this handoff grants the path.
2. **Art Director's F4 commit** is in a separate bundle because it writes outside this track's scope (`art/direction/reviews/` and a non-append edit to the ledger's Terms table). It adds a sign-off note and declares `procedural, project-owned, placeholder-quality` the same class as `studio-code-placeholder` for the ship-blocking pipeline. Merge it or drop it.

## Known gaps and later work
- Low-severity items left open (fixes go back only once):
  - The README wording `entity.flags` (corrected above).
  - In the mortar first-person view the support hand is hidden under the tube; only the forearm and cuff show.
  - The trigger hand barely shows in first person.
  - The third-person body holds no weapon, so aim, fire and reload read only as hand gestures.
  - Slide reads as a one-knee kneel.
- At the 1152×648 movie size the 40 m emblem is only 3 px, and that check fails. The checks target 1080p.
- `godot --import` keeps rewriting `game/audio/default_bus_layout.tres` (lead-engineer or audio-designer). We reverted it each time and never committed it.
- Two orphaned capture PNGs in `game/core/dev/` predate this track and have no ledger line (art-director).
- GUT is still blocked, so all tests use the in-house runner.
