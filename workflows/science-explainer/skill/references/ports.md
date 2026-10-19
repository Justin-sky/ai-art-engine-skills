# 节点拓扑与端口速查（science-explainer）

## 节点清单（共 12 个节点）

| key | typeId | 标题 |
|---|---|---|
| `script` | `play.script` | 科普文章与分镜旁白稿 |
| `svgGen1` | `svg.gen` | 第 1 镜 · 核心概念动态矢量图（SVG） |
| `svgAnim1` | `svg.anim` | 第 1 镜 · 动效烘焙（GIF） |
| `svgGen2` | `svg.gen` | 第 2 镜 · 微观机制动态图解（SVG） |
| `svgAnim2` | `svg.anim` | 第 2 镜 · 动效烘焙（GIF） |
| `illust3` | `asset.image` | 第 3 镜 · 生命工匠高清科普插画 |
| `voice1` | `asset.voice` | 第 1 镜 · 科普旁白配音 |
| `voice2` | `asset.voice` | 第 2 镜 · 科普旁白配音 |
| `voice3` | `asset.voice` | 第 3 镜 · 科普旁白配音 |
| `bgm` | `asset.music` | 探索感科普背景音乐 |
| `timeline` | `output.timeline` | 科普成片时间线 |
| `note` | `note.text` | 科普动画全流程说明 |

## 连线关系（共 16 条连线）

```
script:out → svgGen1:in
svgGen1:out → svgAnim1:in
script:out → svgGen2:in
svgGen2:out → svgAnim2:in
script:out → illust3:in-text
script:out → voice1:in-text
script:out → voice2:in-text
script:out → voice3:in-text
script:out → bgm:in-text
svgAnim1:out-gif → timeline:in-image
svgAnim2:out-gif → timeline:in-image
illust3:out → timeline:in-image
voice1:out → timeline:in-voice
voice2:out → timeline:in-voice
voice3:out → timeline:in-voice
bgm:out → timeline:in-music
```

## 关键端口规范

- `asset.image`：输入 `in-text`（提示词/剧本上下文）、`in-image`（多选参考图/风格与角色锚点）；输出 `out`（单图 `image`）。
- `asset.video`：输入 `in-text`（运镜提示词）、`in-image`（参考图/首帧图）；输出 `out`（单条视频 `video`）。多轨合成时建议设 `generateAudio: false` 避免底噪。
- `asset.voice`：输入 `in-text`（朗读文本）；输出 `out`（语音 `voice`）。注意 `generateInstruction` 中只写要被 TTS 朗读的纯台词正文，不要写导演语气说明。
- `asset.music`：输入 `in-text`（曲风/配器描述）；输出 `out`（音乐 `voice`）。
- `svg.gen` → `svg.anim`：`svg.gen:out`（`svg`）接 `svg.anim:in`（`svg`），`svg.anim:out-video`（`video`）可接入 `output.timeline:in-clips`。
- `output.timeline`（成片时间线）：
  - `in-clips`（`video`，多路输入）：按连线顺序串联各镜头视频与动效片段；
  - `in-voice`（`voice`，多路输入）：接收逐镜配音或全局旁白，自动对齐镜头时长；
  - `in-music`（`voice`）：接收 BGM 背景音乐轨并自动闪避人声；
  - `in-subtitles`（`text`）：接收完整字幕文稿并按句切分铺排底部字幕轨。
