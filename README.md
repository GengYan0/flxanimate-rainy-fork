# rainy-flxanimate

Adobe Animate 导出的纹理图集播放器。读取 `Animation.json`、`TextureAtlas` 图集与 `.zip`
包，按 Animate 的时间轴结构播放 Symbol 动画，含图层、关键帧、滤镜与按钮元件。

## 能力

| 能力 | 入口 |
|---|---|
| 加载导出的图集目录 | `new FlxAnimate(x, y, path)` |
| 加载 zip 包 | `loadAtlas('foo.zip')` |
| 复用已有的图集与 JSON | `loadSeparateAtlas(json, frames)` |
| 按 Symbol 播放 | `anim.anim.play('Symbol')` |
| 按帧标签播放 | `anim.anim.play('label')` |
| 自定义动画序列 | `addBySymbol` / `addByAnimIndices` / `addByCustomTimeline` |
| 帧标签回调 | `addCallbackTo(label, fn)` |
| 按钮元件交互 | `setButtonPack(name, callbacks)` |
| 图层级滤镜 | `filters` |
| 屏幕外剔除 | 自动 |

## 安装

```bash
haxelib git flxanimate https://github.com/GengYan0/flxanimate-rainy-fork
```

`haxelib` 名仍是 `flxanimate`。

## 导出的目录结构

```
assets/animate/boyfriend/
├── Animation.json
├── TextureAtlas.png
└── TextureAtlas.json      （或 spritemap1.json + spritemap1.png 多页）
```

## 播放 Symbol

```haxe
var boy = new FlxAnimate(400, 300, 'assets/animate/boyfriend');
boy.antialiasing = true;
add(boy);

boy.anim.play('idle');
```

`play` 的第二、三个参数是 `force` 与 `reverse`：

```haxe
boy.anim.play('idle', true);        // 强制从头重播
boy.anim.play('walk', true, true);  // 倒放
```

控制：

```haxe
boy.anim.pause();
boy.anim.resume();
boy.anim.stop();
```

## 帧率与完成回调

构造时传设置：

```haxe
var boy = new FlxAnimate(400, 300, 'assets/animate/boyfriend', {
	FrameRate: 30,
	Reversed: false,
	OnComplete: function() trace('loop done'),
	Antialiasing: true
});
```

`Settings` 字段：`FrameRate`、`Reversed`、`OnComplete`、`ShowPivot`、`Antialiasing`、
`ScrollFactor`、`Offset`、`ButtonSettings`。

## 自定义动画序列

不按 Symbol，自行挑帧：

```haxe
// 按 Symbol 名拼一段
boy.anim.addBySymbol('walk', 'WalkCycle', 24, true);

// 按帧序号拼一段
boy.anim.addByAnimIndices('blink', [0, 1, 2, 1, 0], 24);

// 按帧标签拼一段
boy.anim.addByFrameLabel('wave', 'wave_start', 24, true);

boy.anim.play('walk');
```

`addBySymbol` 的参数依次是动画名、Symbol 名、帧率（0 表示沿用源帧率）、是否循环、X、Y 偏移。

## 复用图集

多个角色共用同一份图集时，避免重复加载：

```haxe
var frames = FlxAnimateFrames.fromTextureAtlas('assets/animate/shared');
var json = Assets.getText('assets/animate/shared/Animation.json');

var a = new FlxAnimate(0, 0);
a.loadSeparateAtlas(json, frames);

var b = new FlxAnimate(200, 0);
b.loadSeparateAtlas(json, frames);
```

`loadSeparateAtlas` 的第一个参数是 JSON **内容**，不是路径。

## 帧标签回调

```haxe
boy.anim.addCallbackTo('hit', function() {
	FlxG.camera.shake(0.01, 0.2);
});
```

`removeCallbackFrom(label, fn)` 移除。

## 按钮元件

```haxe
boy.setButtonPack('playButton', {
	onOver: function() boy.color = 0xFFFFFF,
	onOut: function() boy.color = 0xAAAAAA,
	onDown: function() boy.color = 0x666666,
	onUp: function() trace('clicked')
});
```

## 帧标签跳转

```haxe
boy.anim.goToFrameLabel('checkpoint');
var next = boy.anim.getNextToFrameLabel('checkpoint');
```

## 本构建的改动

按屏幕范围剔除。`draw()` 在真正提交前判断精灵是否落在任一目标相机的可见区内，不在就跳过
整棵时间轴的绘制。`cameras` 有自定义值时只测这些相机，为 `null` 时测全部。

```haxe
boy.cameras = [FlxG.camera];   // 只对这一台相机做可见性判断
```

对场景里挂了几十个 Animate 精灵、大部分在画面外的关卡有明显收益。仍在画面内的精灵
行为不变。

## 边界

- 不支持需要三维变换的 Symbol。三维旋转与透视在二维图集里没有对应表达。
- 混合模式只覆盖固定管线能做的那些，需要双纹理的模式不做。
- 图层遮罩不做。
- 大图集受 GPU 纹理上限约束，单页超过 `GL_MAX_TEXTURE_SIZE` 时会被驱动拒绝。

逐条改动与验证方式记录在 `EXTENDING.md`。
