# 节点拓扑与端口速查（ecom-ad-deep）

## 节点清单（共 8 个节点）

| key | typeId | 标题 |
|---|---|---|
| `copy` | `text.plain` | 电商卖点与转化策略 |
| `hero` | `asset.image` | 商品白底/棚拍主图（锚点） |
| `scene` | `asset.image` | 痛点种草使用场景图 |
| `closeup` | `asset.image` | 核心工艺微距卖点图 |
| `adVariants` | `image.adVariants` | A/B 投放变体矩阵 |
| `split` | `image.layerSplit` | 主图 PSD 图层分离 |
| `promoVideo` | `asset.video` | 主图种草短视频 |
| `note` | `note.text` | 电商全案流水线说明 |

## 连线关系（共 9 条连线）

```
copy:out → hero:in-text
copy:out → scene:in-text
hero:out → scene:in-image
copy:out → closeup:in-text
hero:out → closeup:in-image
hero:out → adVariants:in
hero:out → split:in
scene:out → promoVideo:in-image
copy:out → promoVideo:in-text
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
