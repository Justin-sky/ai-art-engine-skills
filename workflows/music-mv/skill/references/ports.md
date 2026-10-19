# 节点拓扑与端口速查（music-mv）

## 节点清单（共 11 个节点）

| key | typeId | 标题 |
|---|---|---|
| `script` | `play.script` | 歌曲意境与 MV 分幕脚本 |
| `charAnchor` | `asset.image` | MV 主角与色调锚点 |
| `bgm` | `asset.music` | 原创主题音乐（BGM/歌曲） |
| `shot1` | `asset.image` | 第一幕 · 主歌氛围关键帧 |
| `shot2` | `asset.image` | 第二幕 · 副歌高潮关键帧 |
| `shot3` | `asset.image` | 第三幕 · 尾奏余韵关键帧 |
| `vid1` | `asset.video` | 第一幕 · 主歌慢镜视频 |
| `vid2` | `asset.video` | 第二幕 · 副歌律动视频 |
| `vid3` | `asset.video` | 第三幕 · 尾奏收束视频 |
| `timeline` | `output.timeline` | MV 成片时间线 |
| `note` | `note.text` | 音乐 MV 制作说明 |

## 连线关系（共 18 条连线）

```
script:out → charAnchor:in-text
script:out → bgm:in-text
script:out → shot1:in-text
charAnchor:out → shot1:in-image
script:out → shot2:in-text
charAnchor:out → shot2:in-image
script:out → shot3:in-text
charAnchor:out → shot3:in-image
shot1:out → vid1:in-image
script:out → vid1:in-text
shot2:out → vid2:in-image
script:out → vid2:in-text
shot3:out → vid3:in-image
script:out → vid3:in-text
vid1:out → timeline:in
vid2:out → timeline:in
vid3:out → timeline:in
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
