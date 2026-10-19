# 节点拓扑与端口速查（game-ua-video）

## 节点清单（共 12 个节点）

| key | typeId | 标题 |
|---|---|---|
| `script` | `play.script` | 买量三段式脚本 |
| `keyart` | `asset.image` | 角色与画风锚点 |
| `hookImg` | `asset.image` | 黄金3秒·危机钩子帧 |
| `hookVid` | `asset.video` | 黄金3秒·钩子视频 |
| `coreImg` | `asset.image` | 核心爽点·觉醒爆发帧 |
| `coreVid` | `asset.video` | 核心爽点·实机演示视频 |
| `ctaVid` | `asset.video` | 福利收尾·转化镜头 |
| `voice1` | `asset.voice` | 钩子痛点口播 |
| `voice2` | `asset.voice` | 爽点与福利转化口播 |
| `bgm` | `asset.music` | 高燃卡点买量 BGM |
| `timeline` | `output.timeline` | 买量成片时间线 |
| `note` | `note.text` | 买量全流程说明 |

## 连线关系（共 20 条连线）

```
script:out → keyart:in-text
script:out → hookImg:in-text
keyart:out → hookImg:in-image
hookImg:out → hookVid:in-image
script:out → hookVid:in-text
script:out → coreImg:in-text
keyart:out → coreImg:in-image
coreImg:out → coreVid:in-image
script:out → coreVid:in-text
keyart:out → ctaVid:in-image
script:out → ctaVid:in-text
script:out → voice1:in-text
script:out → voice2:in-text
script:out → bgm:in-text
hookVid:out → timeline:in
coreVid:out → timeline:in
ctaVid:out → timeline:in
voice1:out → timeline:in-voice
voice2:out → timeline:in-voice
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
