# EXTENDING — rainy-flxanimate

- 基线 `768740a`，haxelib 版本 4.0.0。
- 纯 Haxe，无 ndll，每一层都可以改。
- 引擎侧子类是 `source/rainy/flxanimate/RainyFlxAnimate.hx`，它覆盖 `draw()` 与 `destroy()`；`Project.xml` 不钉版本。

| 目标 | 文件 | 入口 |
|---|---|---|
| 屏幕外剔除 | `flxanimate/FlxAnimate.hx` | `draw()` 开头判断是否落在任一目标相机内 |
| 帧选择开销 | `FlxAnimateFrames.hx`、`data/AnimationData.hx`、`animate/FlxTimeline.hx` | 名字→帧的 Map 查找可换成直接索引 |
| 图集显存 | `frames/FlxAnimateFrames.hx` | 已加载图集的 LRU 淘汰 |
| 新滤镜 | `flxanimate/effects/` | 新增 `FlxXxx` 并在 `FlxAnimateFilterRenderer` 注册 |
| 引擎侧图集缓存 | `source/rainy/flxanimate/RainyFlxAnimate.hx` | `animationCache` 缓存已加载图集 |

`FlxAnimate` 的公开签名保持不变。
