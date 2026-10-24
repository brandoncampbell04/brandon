# Track G (music): return report

**For:** Brandon and the main-session Producer · **Date:** 2026-10-24 · **Branch:** `cloud/music` (11 commits on `main` at 2a2a2d0)

## What's here
- `music.bundle`: restore with `git fetch music.bundle cloud/music:cloud/music` in the studio repo (prerequisite `main` at 2a2a2d0).
- `previews/`: 8 listening files rendered from the committed stems. They are not in the repo.

| File | What you hear |
|---|---|
| `music_director_demo.ogg` | **Start here.** 76 s capture of the live MusicDirector going through every state: hub → sync (duck + flash) → explore → combat (intensity 0 → 1) → hold (anchor lost) → seal (rift sealed) → wake → hub |
| `combat_low_x2.ogg`, `combat_high_x2.ogg` | Combat at intensity layer 1 only, and with all three layers |
| `hub_full_x2.ogg`, `seal_full_x2.ogg` | The Anchorpoint hub and the Seal climax, full mix, looped twice so you can hear the seam |
| `sync_build_flash_bloom.ogg` | Going under: the build, the flash at 2.8 s and the bloom |
| `anchor_lost_over_hold.ogg`, `wake_over_wake.ogg` | Two of the four stingers over their state |

## Summary
1. **G1, score brief** (`design/audio/music/score-brief.md`). Seven states, all in D, share one chord cycle (D, C, G, A). The mission states are each exactly 48 s long, so crossfades keep their place in the bar. The "Sleeper" motif rises a fifth, an octave and a major sixth. The "Hollow" anti-motif is subtractive (`set_hollow_thin`): it thins the music, never adds a dark layer, and never uses the reserved SFX Hollow note.
2. **G2, generator and files** (`game/audio/music/gen/`, numpy plus ffmpeg, deterministic). 33 Ogg files, 10 MB: 28 loop stems, 4 stingers (sync flash, anchor lost, rift sealed, wake) and a one-shot. Every file decodes to its exact bar length, and every state's full mix is −16.0 LUFS ±0.1 at ≤ −1.9 dBTP. Each file has a row in `art/licences.md`, marked placeholder quality.
3. **G3, MusicDirector** (`game/audio/music/music_director.tscn`). It runs on a 48 kHz musical clock and provides `set_state`, `set_intensity`, `stinger`, `set_hollow_thin` and `set_sync_duck`. Crossfades are synced to the bar, stingers to the beat or bar, and the music ducks under the Sync. The demo hits 24 of 24 events on their boundary. In the real audio capture, events land at most 10.7 ms late, which is one engine mix step.
4. **Gates:** each batch got a fresh-context QA Lead check (G1, G2 and G3 all PASS with findings). Every finding was fixed before the merge.
5. **CI on `cloud/music`:** PASS 6/6. The music tests pass 34 of 34; the Python generator tests (run by hand, not in CI) pass 23 of 23.

## Evidence (re-run by Producer on `cloud/music` at 011defd)
```
$ bash tools/harness/ci.sh
[1/6] PASS import (10s)
[2/6] PASS test_runner (78s)
[3/6] PASS levels_validate (2s)
[4/6] PASS net_selftest (2s)
[5/6] PASS registry_sync (1s)
[6/6] PASS retired_theme (0s)
CI: PASS (6/6 steps)
$ godot --headless --path . --script res://tests/runner.gd -- --filter=music
RESULT: PASS  (34 passed, 0 failed, 0 skipped)  exit 0
$ git diff --name-only main..cloud/music | grep -vE '^(game/audio/music/|design/audio/music/|art/licences.md$)'
(nothing: only Track G paths; art/licences.md has 33 appended rows and 0 deletions)
```
QA also confirmed the tests are real by breaking things on purpose in scratch copies:
- **Seam test:** catches a click or a step at the loop point, and a stem cut 1,000 samples short.
- **Director tests:** caught 8 of 9 deliberate logic breaks, plus the author's own 5. The one miss is now covered by a new test.
- **Re-render:** a fresh run of the generator reproduces every file byte for byte.

Detailed logs are in the branch: `design/audio/music/evidence/g3-demo-run.txt` and `g3-capture-timing.txt`.

## Wiring requests (full list with paste-ready snippets: `design/audio/music/wiring-requests.md`)
- **Lead Engineer:**
  - Add the autoload line in `game/registry/autoloads.cfg`.
  - Hook up FlowDirector: flow states and events go to music calls.
  - Remove `Music` from `FlowAudio.DUCK_BUSES` (`game/core/flow/flow_audio.gd:15`). Otherwise the Sync duck is applied twice and also ducks the flash stinger.
- **Level Designer:** route HighcutMission `mission_event` (BEAT_CHANGE) and `cue_event` to the music calls.
- **Network Engineer:** add the cue kinds that clients need for music (brief §7.3).
- **Main-session Audio Designer:**
  - Agree who owns the Music bus volume and duck alongside the `AudioDirector`.
  - Raise the Music voice budget from 4 to 12 decoded voices.
  - Note that a fresh `godot --import` rewrites `default_bus_layout.tres`. This already happens on main; it was not caused by this track.
- **Art Director:**
  - Add the new licence term `procedural-generated, project-owned, placeholder-quality` to the Terms table.
  - Update the out-of-date "no audio files" lines.
  - Two ledger orphans were already on main (`game/core/dev/capture_*.png`, Lead Engineer's).

## Known limits
- 12 of the 28 stems couldn't use the encoder fix for the loop point. They still pass the seam test. `seal_l3` has the weakest seam; it passes, but give it a listen.
- Drift correction against real audio hardware can't run in the cloud, so it needs a check on your machine.
- This is placeholder music. Shipped music still comes from a human composer or a licensed library (your call).

## Needs Brandon (brief §10)
1. Keep D as the home key and the seven-note Sleeper motif for now?
2. Stems at −16 LUFS on the −6 dB Music bus give about −22 LUFS in game. Is that right against VO and SFX, or should stems be −18?
3. Is it OK for the runtime `MusicStems` / `MusicStingers` sub-buses to be created in code?
4. After a lost Lock, should the results loop also avoid resolving?
5. Hard wake: keep the duck with no stinger, or add a `hard_wake` stinger?
6. Ask Level Designer for `group_started` / `pocket_cleared` cue events to drive Hold intensity?
7. Add a lift at ready-up, or keep the hub steady?
