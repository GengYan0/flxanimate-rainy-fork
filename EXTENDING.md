# EXTENDING — flxanimate (rainy fork)

- **Upstream**: Dot-Stuff/flxanimate `dev` branch (vendored @ `768740a`, haxelib 4.0.0)
- **Type**: pure Haxe (no ndll) → every layer is fair game for optimization.
- **Dev link**: `haxelib dev flxanimate <this dir>` (already linked). Engine `Project.xml` does NOT pin a version (per fork contract §6).

## Where to add performance / capability
| Goal | File | Hook |
|---|---|---|
| Skip render when off-screen | `flxanimate/FlxAnimate.hx` `draw()` | done (patch 01: isOnScreen early-out) |
| Frame selection cost | `flxanimate/FlxAnimateFrames.hx`, `data/AnimationData.hx`, `animate/FlxTimeline.hx` | replace name→frame Map lookup with direct index |
| Atlas memory | `flxanimate/frames/FlxAnimateFrames.hx` | LRU eviction for loaded atlases |
| New effect | `flxanimate/effects/` | add `FlxXxx` + register in `FlxAnimateFilterRenderer` |
| Engine-side cache | `source/rainy/flxanimate/PsychFlxAnimate.hx` | `animationCache` Map already caches loaded atlases |

## Rules
- Keep `FlxAnimate` public signatures stable; the engine subclasses `PsychFlxAnimate` and overrides `draw()`/`destroy()`.
- Every perf patch gets a `patches/NN-*.md` with motivation/change/verification/rollback.
- Rendering-path changes need an in-game R2 gate (sprites must not vanish).
