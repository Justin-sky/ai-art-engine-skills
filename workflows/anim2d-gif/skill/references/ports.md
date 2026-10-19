# 节点拓扑与端口速查（anim2d-gif）

## 节点清单（共 5 个节点）

| key | typeId | 标题 |
|---|---|---|
| `charRef` | `asset.image` | 角色立绘基准（锚点） |
| `frameGen` | `frame.animGen` | 生成帧动画序列图 |
| `sheet` | `asset.image` | 备用直出序列图 |
| `anim` | `anim.2d` | 2D帧动画 |
| `note` | `note.text` | 帧动画工业管线说明 |

## 连线关系（共 3 条连线）

```
charRef:out → frameGen:in
charRef:out → sheet:in-image
frameGen:out → anim:in
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
