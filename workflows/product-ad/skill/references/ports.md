# 节点拓扑与端口速查（product-ad）

## 节点清单（共 10 个节点）

| key | typeId | 标题 |
|---|---|---|
| `copy` | `text.plain` | 品牌卖点与脚本 |
| `hero` | `asset.image` | 产品棚拍主视觉（锚点） |
| `scene` | `asset.image` | 真实生活使用场景 |
| `detail` | `asset.image` | 微距工艺卖点特写 |
| `videoHero` | `asset.video` | 产品流光亮相视频 |
| `videoScene` | `asset.video` | 场景使用体验视频 |
| `voice` | `asset.voice` | 品牌质感旁白配音 |
| `bgm` | `asset.music` | 商业品牌氛围配乐 |
| `timeline` | `output.timeline` | 广告成片时间线 |
| `note` | `note.text` | 商业广告制作说明 |

## 连线关系（共 16 条连线）

```
copy:out → hero:in-text
copy:out → scene:in-text
hero:out → scene:in-image
copy:out → detail:in-text
hero:out → detail:in-image
hero:out → videoHero:in-image
copy:out → videoHero:in-text
scene:out → videoScene:in-image
copy:out → videoScene:in-text
copy:out → voice:in-text
copy:out → bgm:in-text
videoHero:out → timeline:in
videoScene:out → timeline:in
detail:out → timeline:in-image
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
