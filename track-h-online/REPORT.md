# Track H report: Epic Online Services (invites and lobbies, no port forwarding)

Branch `cloud/online` from `main` (d09c6d3). Deliverable: `online.bundle` (in this folder; `git bundle create online.bundle main..cloud/online`).
Apply on your machine: `git fetch <path>/online.bundle cloud/online:cloud/online`.

## Summary
- **H1 login:** done in code. The `Online` autoload signs in with EOS Connect + Device ID (no Epic account needed), using a display name.
  - The cloud network blocks `api.epicgames.dev`, so the real login is **untested here**. One command on your PC tests it.
- **H2 lobbies:** done. Lobbies are 4 slots, found only by a 6-character code such as `K7M-Q2P`. The host leaving ends the session.
  - Unit tests and a flow test against an Epic fake both pass.
- **H3 transport:** done. `EosTransport` keeps the net core's 5-channel ENet semantics over EOS P2P, with relays allowed.
  - Two bugs in the plugin had to be worked around (see "Findings").
  - An adapter test against a fake passes, and it runs the real Hello/Welcome/Input/Snapshot messages through SceneMultiplayer.
- **H4 UI:** done as a self-contained panel: **Host online** (code + Copy), **Join with code**, and a status line for every state.
  - Captures pass the detector.
  - The main menu itself (`game/core/`) is locked, so wiring it in is an integration request.
- **CI:** `bash tools/harness/ci.sh` gives PASS (6/6).
- **Secret:** never in git or this bundle. The cloud session did not need it, because Epic is unreachable from here.
- **QA:** a fresh-context review found 3 transport issues (should-fix), all in fragment reassembly and sequence numbering, plus 4 nits. All are fixed or documented, and the fixes have adversarial tests. No blockers.

## Evidence
| Item | Proof |
|---|---|
| CI | `bash tools/harness/ci.sh`: import, test_runner, levels_validate, net_selftest, registry_sync and retired_theme all PASS. `CI: PASS (6/6 steps)` |
| Track H tests | `godot --headless --path . --script res://tests/runner.gd -- --filter=online`: `RESULT: PASS (40 passed, 0 failed)` |
| H1 in the cloud | `eos_smoke.sh login` with a dummy secret reaches the SDK: platform created, then the Device ID call returns `[NoConnection]` after 15 s. The network policy blocks `api.epicgames.dev` |
| Plugin | EOSG main@9869720 (2.3.1 plus lobby and crash fixes) built from source against your EOS SDK 1.19.2 zips. Linux .so loads in Godot 4.7.2. Windows .dll checked statically: entry symbol exported, all 550 EOS imports resolve, and it depends only on system DLLs plus the EOS DLL |
| H4 captures | `captures/`: 5 states × 1920x1080 and 1280x800, rendered on llvmpipe, 644–1316 colours each (`game/ui/online/dev/capture.sh`) |
| Secret | `git log -p main..cloud/online` contains only `PASTE_CLIENT_SECRET_HERE` and a test value `"fake"`. `game/online/.gitignore` ignores `eos_credentials.local.cfg`. I never asked for or held the real secret |

## Findings worth knowing
1. **EOSG turns "unreliable ordered" into reliable.** Used as-is, input and snapshots would queue behind resends. `EosTransport` sends them unreliable with a sequence number and drops stale ones, which is what ENet does.
2. **EOS packets max out at 1170 B, and EOSG silently uses 6 B of that.** A 1200 B snapshot would fail to send. `EosTransport` splits big messages into fragments.
3. **The cloud can't reach Epic.** Every real-EOS step needs your PC.

## What needs your PC (about 5 minutes)
1. Apply the bundle: `git fetch <path>/online.bundle cloud/online:cloud/online && git checkout cloud/online`.
2. Copy `game/online/eos_credentials.example.cfg` to `eos_credentials.local.cfg` and paste the client secret. The file is git-ignored.
3. Run `godot --headless --path . --import`.
4. Run `game\online\dev\eos_smoke.cmd login`. Expect `EOS login OK <ProductUserId>`.
5. Run `game\online\dev\eos_smoke.cmd host` in one window, then `eos_smoke.cmd join-here <CODE>` in a second window. Better: a friend runs `join <CODE>` on their PC. Both sides should print `EOS SMOKE PASS`.

## Integration request (full text in `game/online/README.md`)
- **network-engineer, `net_session.gd`:**
  - Add `host_with_peer(peer, listen)` and `join_with_peer(peer, name)`.
  - Type `_peer` as `MultiplayerPeer`, and guard the four ENet-only calls.
  - `_flush()` calls `EosTransport.flush()`. Disconnects can fire re-entrantly from it, so defer them.
  - No schema change.
- **lead-engineer:**
  - MainMenu gets a "LAN / direct" heading plus `OnlinePanel`.
  - Boot handles `start_online_requested(is_host)` by calling `Online.create_transport()` and then the new Net calls. It calls `Online.leave()` on `session_ended`.
  - Add BootArgs `--online-host` and `--online-join CODE`.
  - Show the join code in the F1 menu.
  - In `export_presets.cfg`, include `game/online/eos_credentials.local.cfg` and exclude the new `dev/` folders.
- **Already in the branch:**
  - Two `autoloads.cfg` lines (`EOSGRuntime`, `Online`).
  - `project.godot` synced with `registry_sync`, as the audio track did.
  - Two `art/licences.md` lines (EOSG MIT, EOS SDK).

## Deviations from the handoff
- **One session did all the work.** The tasks were not dispatched to separate network-engineer and ui-engineer agents, but the one fresh-context QA check still ran.
- **Binaries are built from source.** EOSG release zips are on GitHub release pages, which the network blocks, so the plugin was compiled here. Each platform gets one library, the release build.
- **The UI is a standalone panel**, because `game/core/` is locked. Wiring it into MainMenu is in the integration request.
- **The panel's text is literal English**, matching the Phase 0 MainMenu. Text keys come later.
