# 节点拓扑与端口速查（storybook-narrate）

## 节点清单（共 12 个节点）

| key | typeId | 标题 |
|---|---|---|
| `script` | `text.plain` | 绘本故事分幕文本 |
| `styleAnchor` | `asset.image` | 绘本主角与水彩画风锚点 |
| `page1` | `asset.image` | 第 1 页 · 奇妙开篇插画 |
| `page2` | `asset.image` | 第 2 页 · 森林奇遇插画 |
| `page3` | `asset.image` | 第 3 页 · 温暖归途插画 |
| `voice1` | `asset.voice` | 第 1 页 · 绘本朗读配音 |
| `voice2` | `asset.voice` | 第 2 页 · 绘本朗读配音 |
| `voice3` | `asset.voice` | 第 3 页 · 绘本朗读配音 |
| `sfx` | `asset.sfx` | 夏夜微风与星光环境音 |
| `bgm` | `asset.music` | 治愈八音盒睡前伴奏 |
| `timeline` | `output.timeline` | 有声绘本成片时间线 |
| `note` | `note.text` | 有声绘本制作说明 |

## 连线关系（共 19 条连线）

```
script:out → styleAnchor:in-text
script:out → page1:in-text
styleAnchor:out → page1:in-image
script:out → page2:in-text
styleAnchor:out → page2:in-image
script:out → page3:in-text
styleAnchor:out → page3:in-image
script:out → voice1:in-text
script:out → voice2:in-text
script:out → voice3:in-text
script:out → sfx:in-text
script:out → bgm:in-text
page1:out → timeline:in-image
page2:out → timeline:in-image
page3:out → timeline:in-image
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
