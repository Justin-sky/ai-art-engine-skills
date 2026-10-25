# 节点拓扑与端口速查（game-ua-video）

## 默认种子节点清单（共 12 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `script` | `text.plain` | 买量三段式脚本 |
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

## 默认连线关系（共 21 条连线）

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
script:out → timeline:in-text
```

## 核心算子端口与参数规范

- `asset.image`：输入 `in-text`（提示词/剧本上下文）、`in-image`（多路参考图/角色与风格锚点）；输出 `out`（单图 `image`）、`out-all`（图库全部图 `images`）。
- `asset.video`：输入 `in-text`（运镜提示词）、`in-image`（参考图；若设 `generateFrameMode: "first"` 则自动切换为 `in-first-frame` 首帧口，`"first_last"` 则增加 `in-last-frame` 尾帧口）；输出 `out`（单条视频 `video`）。多轨配音合成时建议设 `generateAudio: false` 避免底噪干扰旁白。
- `asset.voice`：输入 `in-text`（朗读文本）；输出 `out`（语音 `voice`）。**铁律**：`generateInstruction` 中只写要被 TTS 直接朗读的纯台词正文，严禁写入"请用温柔女声朗读"等导演说明文字（否则 TTS 会把说明词当台词念出来）。
- `asset.sfx`：输入 `in-text`（音效描述）；输出 `out`（音效 `voice`），可连入 `output.timeline` 的 `in-voice`。
- `asset.music`：输入 `in-text`（曲风/配器/BPM 描述）；输出 `out`（音乐 `voice`），连入 `output.timeline` 的 `in-music`。
- `video.lipSync`：输入 `in-video`（人物底板视频 `video`）+ `in-voice`（口播音频 `voice`）；输出 `out`（唇形同步视频 `video`）。
- `svg.gen` → `svg.anim`：`svg.gen:out`（`svg`）接 `svg.anim:in`（`svg`）；`svg.anim:out-gif`（`image`，烘焙后的透明/动态 GIF 资产）接 `output.timeline:in-image`。
- `frame.animGen` → `anim.2d`：`frame.animGen:out`（`image` 序列图）接 `anim.2d:in`（`image`）；`anim.2d` 输出 `out`（单帧）、`out-all`（全部帧）与 `out-gif`（合成循环 GIF）。
- `output.timeline`（成片时间线）：
  - `in`（`video`，多路输入）：按连线顺序串联各镜头视频片段；
  - `in-videos`（`videos`，多路输入）：接收分集工作台等输出的复数视频列表；
  - `in-image` / `in-images`（`image` / `images`，多路输入）：接收静态插画或 `svg.anim` / `anim.2d` 的 `out-gif` 动态图；
  - `in-voice` / `in-voices`（`voice` / `voices`，多路输入）：接收逐镜配音、全局旁白或音效（`asset.sfx`），支持在 `timeline_edit` 中配置 `pitchSemitones`（`-12 ~ +12` 半音变调）与 `voiceEffect`（`chipmunk` / `demon` / `robot` / `echo` / `megaphone` / `autotune-chorus`）；
  - `in-music`（`voice`）：接收 BGM 背景音乐轨并自动闪避人声；
  - `in-text`（`text`）：接收完整字幕/歌词文稿并自动按句切分铺排底部字幕轨（亦可通过 `timeline_edit` 精细编排 `subtitleItems`、`@{tag}` 字级音画锚点与 `karaoke-pop` 综艺花字）。
