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
