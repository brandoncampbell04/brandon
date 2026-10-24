# Track D: Anchorpoint city hub, return report

From the Track D Producer session, 2026-10-24. The handoff was `handoffs/HANDOFF-HUB.md`.

## How to apply
```bash
cd somnarium-studio && git fetch /path/to/hub.bundle cloud/hub:cloud/hub && git checkout cloud/hub
bash tools/harness/ci.sh        # CI: PASS (6/6 steps)
```
- `hub.bundle` is `main..cloud/hub`: 40 commits, head `fecd638`. Its base is `4eea7f4`, the handoff commit.
- It was verified by applying it to a fresh clone of the original bundle and running CI there: 6/6 pass.
- `main` was not touched.

## Summary
- **D1:** the city brief is canon-signed. It has 9 districts, including one quarter per faction (Stewards' Close, the Survey Yard, the Night School), 4 landmarks, 11 ray-checked sightlines and a day-time look. The Art Director's notes cover colours, lighting and gate states.
- **D2:** a generated graybox city at `game/levels/anchorpoint/anchorpoint_city.tscn`. It has a baked navmesh and 426 CSG boxes against a budget of 500.
  - The Somnarium interior is identical to the old hub: same markers, positions and meta. The flow should therefore work unchanged; that was checked against the scene contract, not run through SyncFlow.
  - 8 gates; only Front Lines is live.
  - The farthest district, the Night School, is 612 m from the spawn on the navmesh, about **2:02** at walking speed.
- **D3:** UI in `game/ui/`: a world-select gate board, gate prompt, nameplates, a district toast and a 3-emote wheel. All of it is pad-navigable and every string comes from the string table.
- **D4:** 91 string keys: district signs, one sentence per gate, the world-select and emote UI text, and 20 placeholder NPC lines. Passed the canon gate and the Fair Play gate.

## Gates and evidence
| Check | Result |
|---|---|
| `bash tools/harness/ci.sh` (baseline, then after every merge, then on the final head) | `CI: PASS (6/6 steps)` |
| `godot --headless --path . --script res://game/world/hub/hub_validate.gd` | `HUB VALIDATE: PASS (39 checks, 0 failed)`. Farthest district `District_night_school` 611.7 m = 2:02. Largest up-edge 0.040 m, steepest climb 7.1° (limit 46°). |
| Unit tests (`tests/runner.gd`) | `--filter=anchorpoint_city`: 6 pass. `--filter=game/ui`: 57 pass. Each set was shown to fail on the base. |
| Pad-only navigation bot (`game/ui/dev/run_nav_bot.gd`) | `reached 10 of 10 controls`, `NAV BOT: PASS` |
| Captures (Xvfb + lavapipe) | 8 city and 10 UI frames. All pass the detector with `llvmpipe` in the log. City frames: darkest V 0.20 or above, 0% of pixels at V ≥ 0.95, 0% amber. Landmark contrast against the sky (V ratio): tower 2.07–2.13, dish 2.34–2.56. |
| Retired-theme detector | no matches |
| QA Lead (fresh context, one per batch) | D1 PASS · D2+D4 PASS · D3 PASS · fix round PASS |
| Creative Director canon gate | Brief signed. D4 and D3 both PASS WITH FIXES; all fixes applied. |
| Security Engineer (Fair Play) | PASS WITH FIXES. F1 applied: "repair benches" removed from the Market sign, since they read as paid gear repair. |
| Art Director | Art review of D2 applied. Ledger audit: no Track D orphans. |
| Ownership | Every file changed is in a Track D path. `game/registry/` has no net change. The ledger has no removed lines. |

Captures are in `captures/`:
- `city/`: overview, the plaza east and west, a closed gate, the Main Line, the Survey Tower, the dish, and Lindenrow looking toward the clock tower.
- `ui/`: the gate board (live and not-yet-stable), nameplates, the district toast and the emote wheel, each at 1920×1080 and 1280×800.
- `layout.png`: the top-down plan.

The city frames are 1152×648, the project window size.

## Requests for the main session (locked paths)
1. **Hub scene:** in `game/core/flow/flow_director.gd:28`, point `HUB_SCENE` at `res://game/levels/anchorpoint/anchorpoint_city.tscn`. Add the city to `LEVELS` in `game/core/tests/test_look_profile.gd`.
2. **World select in the flow:**
   - Connect `HubUi.world_chosen(gate_id, branch)` to crew booking; Front Lines → `highcut`.
   - Wire the `HubUi` API (`setup`, `update_player`, `open_world_select`) and freeze movement while `modal_changed(true)`.
   - The API is documented in `game/ui/README.md`.
3. **Net replication** (Network Engineer):
   - Emotes: client to server `EmoteMsg{emote_id:u8, tick:u32}`, rate-limited to one per 0.5 s, rebroadcast as `EmoteEvent{entity_id:u16, emote_id:u8}`.
   - Nameplates: send `display_name`, `crew_id` and `accent` per Sleeper at spawn.
4. **Registry and project.godot** (Lead Engineer):
   - Append the `emote_wheel` input line given verbatim in `game/ui/README.md`, then run `registry_sync`.
   - Add `&"emote_wheel"` to `InputBits.FALLBACK_ORDER`.
   - Register `game/data/text/anchorpoint/anchorpoint.csv` as a translation.
   - Until then, `UiBoot` adds both at runtime. Appending the line now would turn CI red, so it was left out.
5. **CI** (QA Lead): add `hub_validate.gd` as a `ci.sh` step. Also make `tools/harness/capture.sh` honour `CAP_SIZE` for the game window; it currently records at 1152×648. `game/ui/dev/capture_ui.sh` shows a way to do it.
6. **Old hub** (Level Designer): `TerminalLight` `d9efe8` in `game/world/blockout/hub_builder.gd` is in the Hollow White band. The city scene uses `b4dccf`.
7. **Player cap:** raise the hub cap later. Phase 0 networking supports 4 players.
8. **Ledger:** two orphans that predate this track: `game/core/dev/capture_highcut_sleeper.png` and `capture_main_menu_export.png`.

## Open questions for Brandon (each has a recommendation)
1. **Homes:** what is a Home? *Recommendation:* your own rooms in a Lindenrow co-op house, at home in the city. No sync, no stamp and no power. This is provisional for now.
2. **Outpost (ruling E5):** *Recommendation:* the Windrow homestead's ledger. Materials stay in Windrow, notes come home as knowledge, and a capped Windrow-stamped credit is recorded. Never sold. Until you decide, its gate shows its name and state only. Economy v1 already mints 3 credits an hour for it, so its copy needs the Fair Play gate again once you answer.
3. Should the Homes gate say "not yet open" instead of "not yet stable"? *Recommendation:* yes. One new key would do it.
4. **Booking:** choosing a gate books your crew, and you then walk 30–90 m to the hall and sync at a pod. Is that walk the ritual you want? *Recommendation:* keep the walk; there is no fast travel.
5. **Scale:** a 630 m city will feel empty for 4 players. *Recommendation:* keep the scale, add ambient NPCs early, and later add a tram on the Main Line.
6. **Great dish:** it is lowered to face the horizon along the plaza rather than the sky. *Recommendation:* approve.
7. **Ruins (theme bible §10 Q3):** are ruins Hollow casts? The Ruin Raids text depends on the answer.
8. **Repair benches:** keep them in the Market? Only if repair is cosmetic. They were removed from the sign for now.
9. **Wave emote:** the Creative Director ruled it allowed as a hand gesture, and its icon is an open hand.

## Process notes
- The roles ran as separate agents, each in its own worktree, at most 3 at a time. Merges were one at a time, with CI after each.
- In batch 3 a QA agent checked out a detached commit in the shared checkout mid-pass. The Creative Director's D3 review merge landed off-branch as a result. The final QA caught it, and it was merged again (`fecd638`).
- Future QA passes should run in their own worktree.
