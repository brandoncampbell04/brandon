# Art Director sign-off: Track F characters (F4)

Reviewer: Art Director. Date: 2026-10-24. Branch reviewed: `cloud/characters` at 1892844 (F1 Sleeper body and FP arms, F2 animation, F3 Blank v2).
Against: style bible, briefs `sleeper-base-body.md` and `blank.md`, theme bible, prototype spec 3.4 and 3.6, `HANDOFF-CHARACTERS.md`.
Evidence viewed: F2 emblem turntables (all 4 looks, 8 angles), 40 m 1080p zooms (chest and shoulder), FP rifle and mortar, 3.0 m boom back and turned, F2 contact sheet of all states, F3 sheets at 3 m and 30 m, both READMEs.

## Verdict: SIGNED OFF WITH FIX NOTES
No blocking violation. Procedural only (no imports, no generators); amber appears only as the player emblem; the Blank is Shadow Black and Hollow White only; the placeholder-quality is honest and brief-faithful. One small must-fix (1); the rest can follow.

## What passes
- **Emblem, Sleeper:** chest disc and left-shoulder disc are visible in all 4 looks across the turntable (front, three-quarters, side). At 40 m (1080p, 8x zoom) chest reads as an amber blob and the shoulder as an amber crescent, in every look; the README measures 14 and 15 strict amber pixels. Amber stays off cloth, trim, gloves and weapons (viewmodels are granite and Garden Teal).
- **Looks:** the four looks differ by value and hue (Night Blue, Garden Teal, olive-brown, umber-and-ivory); patterns stay subtle and the trim bands are small. No amber, near-white or near-black large area.
- **Silhouette:** upright person, big pack mass, round limbs, soft shapes; clearly not the Blank. Capsule fit is test-measured (stand 1.795 m, crouch 1.110 m, reach 0.41 m against r 0.40).
- **Shoulder pads (x1.3, 22 cm disc): accepted.** They now carry the 40 m shoulder read and sit inside the capsule. They are the heaviest "armour-like" shapes on the body, so do not grow them further (see L3).
- **Blank v2:** reads at 30 m in every state. Humanoid with no neck, small tipped oval head, long arms, white core of at least 0.32 m, and a torn white edge that makes the outline feel "not yet drawn". Lunge wind-up (huge white star flare), bolt glint (cross), blink afterimage (banded white copy) and dissolve (white flakes) all change the silhouette strongly. Only the two Hollow tokens appear; no amber, no gore, no face detail. Notches (coat-hem step, strap groove, rung bite) are present and subtle. Not a ghost, reaper or mannequin.
- **Animation:** all 11 states appear in the contact sheet with plausible posing; the hard-wake collapse is face-down by 1.0 s with no gore. Emblems stay on the chest and the shoulder in every state, with arm raise included (pads ride on Spine).
- **Ledger:** the 7 Track F lines exist (asset table, lines 48-54); see ruling below.

## Must fix before merge
1. **Mortar wrist emblem is cut off (`game/characters/sleeper/fp_arms.gd`, mortar hold pose / `viewmodel_kit.gd` mortar grip offset).** In `fp_breaker_mortar.png` the wrist emblem is only about half inside the bottom edge of the 1152x648 frame. Raise or pull in the support-forearm in the mortar pose (roughly 8-10 cm up and 3 cm inward in camera space) so the whole 5.1 cm disc is in frame at the default FOV and 16:9, with a margin of at least 1 emblem radius from the bottom edge. Passes: a re-captured `fp_breaker_mortar.png` shows the full amber disc, plus the existing emblem-pixel check on the FP frame. The rifle pose already passes (disc fully visible).

## Later / requests to other owners
L1. **FP trigger hand (Technical Artist, optional polish).** The trigger-hand arm barely shows in either pose; that is acceptable (the wrist emblem lives on the support wrist, by design in the README). If the FP rig is reworked, show the gloved hand on the grip. No amber requirement here.
L2. **Back-of-body emblem (request to the shader owner, Technical Artist, `game/shaders/echo/`, via Producer).** The 3.0 m over-the-right-shoulder boom (`tps_back.png`) shows neither chest nor left-shoulder emblem: the pack hides the back and the left pad is turned away. This is within the brief (chest and shoulder only), so it does not block, but "amber always visible" is the rule, and teammates see this view most. Request a **third emblem slot (slot 3) on the pack back or the right shoulder pad**, at least 16 cm, 15 cm clean zone, same locked emblem term, so that every azimuth at 40 m shows at least one emblem. Art Director will amend `sleeper-base-body.md` once Producer approves; commissioned art then adds a back emblem decal quad.
L3. **Commissioned body (brief `sleeper-base-body.md`).** Keep pad volume modest and give them a soft, rolled-collar, shoulder-bag read, not hard pauldrons; hands are mittens and the head is a bald ovoid, acceptable as placeholder only.
L4. **Blank lunge wind-up (Creative Director look, not blocking).** At peak the flare almost hides the body at 3 m. It reads as a telegraph and passes 30 m; if playtests find it too loud or it hides the hitbox, shorten the flare to about 0.8 m.
L5. **Blank notches at 30 m.** Notches fall below a pixel at 30 m and are invisible by design; judge them only at 3 m.

## Ledger ruling (`art/licences.md`)
The Track F lines stay in the asset table (they name real `.tscn`, `.gd` and `.gdshader` paths, not codeless procedurals). The brief wording `procedural, project-owned, placeholder-quality` is kept as written. I added it as a term in the Terms table and extended the pipeline sentence, so it is treated as the same class as `studio-code-placeholder` (flag before shipping, never final art). No Track F line was altered.

## Sign-off state
Merge once fix 1 is re-captured and Producer sees the new `fp_breaker_mortar.png`. L1-L5 go on the board. No second fix round is needed from Art Director unless the re-capture shows the emblem still cropped.
