# EXTENDING — rainy-flxanimate

- **基线**: `dev` @ `768740a`，haxelib 版本 4.0.0
- **类型**: 纯 Haxe（无 ndll）→ 每一层都可以改。
- **Dev 链接**: `haxelib dev flxanimate <this dir>`（已链接）。引擎 `Project.xml` 不钉版本。

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
