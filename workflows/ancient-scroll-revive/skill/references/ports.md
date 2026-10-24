# 节点拓扑与端口速查（ancient-scroll-revive v1.3）

## 节点清单（共 22 个节点）

| key | typeId | 标题 |
|---|---|---|
| `scroll_src` | `asset.image` | 古画全卷母版 · 清明上河图 |
| `act1_crop` | `image.process` | 第1幕·古画取景（右段·城郊柳岸） |
| `act2_crop` | `image.process` | 第2幕·古画取景（中段·虹桥争渡） |
| `act3_crop` | `image.process` | 第3幕·古画取景（左段·繁华酒楼） |
| `act1_recognize` | `image.toPrompt` | 第1幕·多模态识图与对白/解说策划 |
| `act2_recognize` | `image.toPrompt` | 第2幕·多模态识图与对白/解说策划 |
| `act3_recognize` | `image.toPrompt` | 第3幕·多模态识图与对白/解说策划 |
| `act1_real` | `asset.image` | 第1幕·北宋实景还原（灵活中近景） |
| `act2_real` | `asset.image` | 第2幕·北宋实景还原（灵活中近景） |
| `act3_real` | `asset.image` | 第3幕·北宋实景还原（灵活中近景） |
| `act1_video` | `asset.video` | 第1幕·人物说话与活动复活（含原声对白） |
| `act2_video` | `asset.video` | 第2幕·人物说话与活动复活（含原声对白） |
| `act3_video` | `asset.video` | 第3幕·人物说话与活动复活（含原声对白） |
| `act1_voice` | `asset.voice` | 第1幕·画面解说旁白 |
| `act2_voice` | `asset.voice` | 第2幕·画面解说旁白 |
| `act3_voice` | `asset.voice` | 第3幕·画面解说旁白 |
| `act1_sfx` | `asset.sfx` | 第1幕·移镜入画与柳岸环境声 |
| `act2_sfx` | `asset.sfx` | 第2幕·移镜入画与虹桥环境声 |
| `act3_sfx` | `asset.sfx` | 第3幕·移镜入画与酒楼环境声 |
| `bgm` | `asset.music` | 北宋雅乐环境配乐 |
| `timeline` | `output.timeline` | 古画一镜入画成片时间线 |
| `note` | `note.text` | 古画一镜入画 · 灵活中近景与视频直出人物说话+旁白解说指南 |

## 连线关系（共 37 条连线）

```
scroll_src:out → act1_crop:in
scroll_src:out → act2_crop:in
scroll_src:out → act3_crop:in
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
act1_crop:out → timeline:in-image
act2_crop:out → timeline:in-image
act3_crop:out → timeline:in-image
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

## 关键端口与参数规范

- `image.process`（`imageProcessTool: "crop"`）：输入 `in`（`image`），输出 `out`（`image`）。当配置了 `params.generateInstruction` 时，多模态视觉模型（VLM）根据画面内容灵活调整最佳 16:9 中景/中近景取景框（不死板固定宽高），兼顾人物神态动作完整度与场景意境。
- `image.toPrompt`：输入 `in`（`image`），输出 `out`（`text`）。同时输出 `【第k幕 · 场景名】`、直接传给视频模型的人物说话对白台词，以及供旁白轨使用的画面解说词。
- `asset.video`（`generateAudio: true`）：接收首帧实景图与人物对白指令，由视频模型直接生成口型同步的人物开口说话聊天声与现场环境原声。
- `asset.voice`（`act1_voice` ~ `act3_voice`）：独立旁白轨道（`voice` 轨），专门用于对画面进行专业历史与场景解说。
- `output.timeline`：支持同类型多条子轨道（`lane: 0..7`），在 `video` 轨保留视频模型直出的人物说话原声（`volume: 0.9`），在 `voice` 轨叠加画面解说旁白，在 `sfx` 双轨（`lane 0` 移镜快门 + `lane 1` 现场环境）与 `subtitle` 轨同步合成。
