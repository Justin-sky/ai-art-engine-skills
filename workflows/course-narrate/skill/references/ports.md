# 节点拓扑与端口速查（course-narrate）

## 节点清单（共 11 个节点）

| key | typeId | 标题 |
|---|---|---|
| `script` | `play.script` | 课程结构化讲稿 |
| `portrait` | `asset.image` | 主讲人定妆照（锚点） |
| `talking` | `asset.video` | 主讲人出镜底片 |
| `voice` | `asset.voice` | 讲师口播配音 |
| `lipSync` | `video.lipSync` | 数字人口型同步 |
| `svgGen` | `svg.gen` | 核心知识点动态板书（SVG） |
| `svgAnim` | `svg.anim` | 动态板书烘焙（GIF） |
| `broll` | `asset.image` | 课程案例视觉插图（B-Roll） |
| `bgm` | `asset.music` | 知识课堂轻音乐铺底 |
| `timeline` | `output.timeline` | 课程成片时间线 |
| `note` | `note.text` | 精品知识课全流程说明 |

## 连线关系（共 15 条连线）

```
script:out → portrait:in-text
script:out → voice:in-text
script:out → talking:in-text
portrait:out → talking:in-image
talking:out → lipSync:in-video
voice:out → lipSync:in-voice
script:out → svgGen:in
svgGen:out → svgAnim:in
script:out → broll:in-text
script:out → bgm:in-text
lipSync:out → timeline:in
svgAnim:out-gif → timeline:in-image
broll:out → timeline:in-image
voice:out → timeline:in-voice
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
