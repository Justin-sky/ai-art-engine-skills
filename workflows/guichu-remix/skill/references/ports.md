# 节点拓扑与端口速查（guichu-remix v1.0）

## 节点清单（共 20 个节点）

| key | typeId | 标题 |
|---|---|---|
| `meme_src` | `asset.image` | 鬼畜名场面母版 · 鹅城三巨头交锋全景 |
| `act1_crop` | `image.process` | 第1幕·名场面取景（Verse·念诗之王押韵铺垫） |
| `act2_crop` | `image.process` | 第2幕·名场面取景（Build-up·翻译翻译高频复读） |
| `act3_crop` | `image.process` | 第3幕·名场面取景（Drop·敢杀我的马副歌爆发） |
| `act1_recognize` | `image.toPrompt` | 第1幕·鬼畜押韵台词与微循环卡点策划 |
| `act2_recognize` | `image.toPrompt` | 第2幕·高频鬼畜复读与正反抽帧策划 |
| `act3_recognize` | `image.toPrompt` | 第3幕·副歌咆哮爆发与分屏伴舞策划 |
| `act1_real` | `asset.image` | 第1幕·高反差鬼畜表情精修帧 |
| `act2_real` | `asset.image` | 第2幕·高反差鬼畜表情精修帧 |
| `act3_real` | `asset.image` | 第3幕·高反差鬼畜表情精修帧 |
| `act1_video` | `asset.video` | 第1幕·魔性押韵念诗卡点视频（含原声） |
| `act2_video` | `asset.video` | 第2幕·高频质问鬼畜微循环视频（含原声） |
| `act3_video` | `asset.video` | 第3幕·副歌咆哮爆发高潮视频（含原声） |
| `act1_voice` | `asset.voice` | 第1幕·念诗之王押韵踩点人声 |
| `act2_voice` | `asset.voice` | 第2幕·翻译惊喜高频复读人声 |
| `act3_voice` | `asset.voice` | 第3幕·敢杀我的马副歌咆哮 Hook |
| `act1_sfx` | `asset.sfx` | 第1幕·拍桌重击与折扇脆响采样 |
| `act2_sfx` | `asset.sfx` | 第2幕·拉枪栓与清脆耳光打击采样 |
| `act3_sfx` | `asset.sfx` | 第3幕·重低音拍案爆裂与锣鼓齐鸣采样 |
| `bgm` | `asset.music` | 126BPM 疯猴流派 Funky 电音鬼畜伴奏 |
| `timeline` | `output.timeline` | 鬼畜魔性卡点成片时间线 |
| `note` | `note.text` | B站亿级鬼畜（敢杀我的马×改革春风吹满地）六大剪辑法则指南 |

## 连线关系（共 34 条连线）

```
meme_src:out → act1_crop:in
meme_src:out → act2_crop:in
meme_src:out → act3_crop:in
act1_crop:out → act1_recognize:in
act2_crop:out → act2_recognize:in
act3_crop:out → act3_recognize:in
act1_crop:out → act1_real:in-image
act1_recognize:out → act1_real:in-text
act2_crop:out → act2_real:in-image
act2_recognize:out → act2_real:in-text
act3_crop:out → act3_real:in-image
act3_recognize:out → act3_real:in-text
act1_real:out → act1_video:in-first-frame
act1_recognize:out → act1_video:in-text
act2_real:out → act2_video:in-first-frame
act2_recognize:out → act2_video:in-text
act3_real:out → act3_video:in-first-frame
act3_recognize:out → act3_video:in-text
act1_recognize:out → act1_sfx:in-text
act2_recognize:out → act2_sfx:in-text
act3_recognize:out → act3_sfx:in-text
act1_video:out → timeline:in
act2_video:out → timeline:in
act3_video:out → timeline:in
act1_voice:out → timeline:in-voice
act2_voice:out → timeline:in-voice
act3_voice:out → timeline:in-voice
act1_sfx:out → timeline:in-voice
act2_sfx:out → timeline:in-voice
act3_sfx:out → timeline:in-voice
act1_recognize:out → timeline:in-text
act2_recognize:out → timeline:in-text
act3_recognize:out → timeline:in-text
bgm:out → timeline:in-music
```

## 鬼畜时间线精编参数速查（`timeline_edit`）

- **节拍网格换算（以 126 BPM 为例）**：
  - 1 拍（1/4 音符）：`0.48s`
  - 半拍（1/8 音符）：`0.24s`
  - 1/4 拍（1/16 音符机关枪复读）：`0.12s`
- **高频微切片复读（`track: "video"`）**：
  - 通过 `sourceOffsetSec` 指定原片金句或夸张表情起点，`durationSec` 设为 `0.48` / `0.24` / `0.12`，配合 `speed: 1.2 ~ 1.6` 连续铺轨。
- **重拍视觉特效（`visualEffect`）**：
  - `"soul-out"`（灵魂出窍残影抽离）、`"rgb-glitch"`（红蓝通道故障色差）、`"strobe-flash"`（快节奏明暗频闪）、`"beat-pulse"`（随节拍呼吸缩放）、`"earthquake"`（重低音剧烈震屏）、`"zoom-burst"`（急推冲击）、`"split-3v"`（三联分屏）。
- **副歌左右护法画中画伴舞（`track: "overlay"`）**：
  - 左翼伴舞（`lane: 0`）：`overlayX: 0.02, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64, tweenLoop: "pendulum", tweenLoopSpeedHz: 2.1`
  - 右翼伴舞（`lane: 1`）：`overlayX: 0.72, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64, tweenLoop: "pulse", tweenLoopSpeedHz: 2.1`
- **洗脑大字幕与语义锚点爆点音效（`track: "subtitle"` + `op: "syncCues"`）**：
  - `subtitleKaraokeMode: "karaoke-pop"`, `fontSize: 56`, `textColor: "#FACC15"`, `subtitleHighlightColor: "#F43F5E"`，内联 `@{boom}敢杀我的马@{/boom}` 自动吸附 `sfxPresetId: "boom"` 与 `stickerId: "comic-boom"`。
