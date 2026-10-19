# 节点拓扑与端口速查（storyboard-video）

## 节点清单（共 14 个节点）

| key | typeId | 标题 |
|---|---|---|
| `shots` | `play.script` | 导演分镜与旁白脚本 |
| `charRef` | `asset.image` | 主角与光影定妆锚点 |
| `board1` | `asset.image` | 分镜 1 · 远景定场 |
| `board2` | `asset.image` | 分镜 2 · 中景叙事推进 |
| `board3` | `asset.image` | 分镜 3 · 特写情绪高潮 |
| `clip1` | `asset.video` | 镜头 1 · 定场运镜视频 |
| `clip2` | `asset.video` | 镜头 2 · 中景跟拍视频 |
| `clip3` | `asset.video` | 镜头 3 · 高潮特写视频 |
| `voice1` | `asset.voice` | 镜头 1 旁白配音 |
| `voice2` | `asset.voice` | 镜头 2 旁白配音 |
| `voice3` | `asset.voice` | 镜头 3 高潮旁白 |
| `bgm` | `asset.music` | 电影氛围配乐 |
| `timeline` | `output.timeline` | 分镜成片时间线 |
| `note` | `note.text` | 导演分镜全流程说明 |

## 连线关系（共 24 条连线）

```
shots:out → charRef:in-text
shots:out → board1:in-text
charRef:out → board1:in-image
shots:out → board2:in-text
charRef:out → board2:in-image
shots:out → board3:in-text
charRef:out → board3:in-image
board1:out → clip1:in-image
shots:out → clip1:in-text
board2:out → clip2:in-image
shots:out → clip2:in-text
board3:out → clip3:in-image
shots:out → clip3:in-text
shots:out → voice1:in-text
shots:out → voice2:in-text
shots:out → voice3:in-text
shots:out → bgm:in-text
clip1:out → timeline:in
clip2:out → timeline:in
clip3:out → timeline:in
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
