# Patch 01 — isOnScreen early-out in FlxAnimate.draw()

## Motivation
`FlxAnimate.draw()` runs an expensive recursive `parseElement()` over the whole
symbol tree every frame, even for sprites fully outside the camera view
(off-screen characters, modchart sprites scrolled away, note splashes).
For songs with many animated elements this is pure wasted CPU/GPU submission.

## Change
- `FlxAnimate.draw()` (flxanimate/FlxAnimate.hx): before the matrix setup,
  early-return when the sprite is off every camera it renders to.
- New helper `isOnAnyCamera()`:
  - target cameras = `this.cameras` if set, else `FlxG.cameras`;
  - returns true if `isOnScreen(cam)` for any target camera.
- Conservative guards: never cull while `width == 0 || height == 0`
  (bounds are filled by `parseElement` on the first draw, so this avoids
  vanishing on frame 1), and only skip when off ALL target cameras.

## Verification
- Engine release build: type-check + link gate (run by caller; no auto-build this pass).
- In-game gate (R2 eval mode / manual song): **failure signal = any animated
  sprite vanishing or popping when it should be visible** (esp. sprites with
  custom `cameras` set, or non-default `scrollFactor`).
- Rollback: remove the 3-line guard at the top of `draw()` and the
  `isOnAnyCamera()` helper. No API/signature change.

## Expected benefit
Skips recursive symbol render for off-screen flxanimate instances. Benefit scales
with (off-screen animated instances) × (symbol-tree depth); zero cost for
on-screen sprites.
