# 节点拓扑、变调算子与自由扩展速查（guichu-remix v1.3）

## 1. 核心种子节点清单（4 节点 3 连线）

| key | typeId | 标题 | 作用 |
|---|---|---|---|
| `src-video` | `asset.video` | 鬼畜原片/名场面素材源 | 绑定 `web_fetch_video` 下载或本地导入的原片视频（`locked: true, params: { sourceRelativePath }`），或直接 AI 生成名场面视频 |
| `bgm` | `asset.music` | 126BPM 疯猴流派 Funky 电音鬼畜伴奏 | 生成 126 BPM 四四拍 Funky Electro-Trap 纯音乐伴奏 |
| `sfx-boom` | `asset.sfx` | 鬼畜重拍打击采样（拍桌/耳光/枪栓） | 生成卡在重拍上的动作打击音效采样 |
| `timeline-out` | `output.timeline` | 鬼畜魔性卡点成片时间线 | 汇聚全部视频、伴奏、音效、贴纸与字幕，承载鬼畜变调与微切片时间线 |

### 预设连线关系

```
src-video:out  → timeline-out:in
bgm:out        → timeline-out:in-music
sfx-boom:out   → timeline-out:in-voice
```

## 2. 按需自由挂载的扩展节点配方（通过 `graph_edit` 组合）

- **多视频源混剪**：`{ op: "node_upsert", nodeId: "src-video-2", typeId: "asset.video", locked: true, params: { sourceRelativePath: "..." } }` + `{ op: "edge_connect", fromNodeId: "src-video-2", fromPort: "out", toNodeId: "timeline-out", toPort: "in" }`
- **人物抠图魔性摇晃大头**：`{ op: "node_upsert", nodeId: "head-cutout", typeId: "image.process", params: { imageProcessTool: "cutout" } }` + `{ op: "edge_connect", fromNodeId: "head-cutout", fromPort: "out", toNodeId: "timeline-out", toPort: "in-image" }`
- **SVG 动态鬼畜血条 / VS 计分板 / 弹幕贴纸**：`svg.gen` (`out -> in`) $\to$ `svg.anim` (`out-gif -> in-image`) $\to$ `timeline-out`
- **AI 押韵念诗配音 / 多角色对呛**：`asset.voice` / `asset.dialogue` (`out -> in-voice`) $\to$ `timeline-out`

## 3. 鬼畜时间线变调与精编参数速查（`timeline_edit`）

- **半音阶梯爬音与人力 VOCALOID 调音（`pitchSemitones`）**：
  - 取值范围：`-12 ~ +12`（单位：半音，基于 `asetrate + atempo` 反向补偿，**100% 保持切片时长 `durationSec` 不变**，绝不破坏 BPM 拍点网格！）。
  - 预副歌 4 连发 1/4 拍爬音：`pitchSemitones: 0` $\to$ `2` $\to$ `4` $\to$ `7`
  - 预副歌 8 连发 1/8 拍冲顶：`pitchSemitones: 0` $\to$ `3` $\to$ `5` $\to$ `7` $\to$ `9` $\to$ `12` $\to$ `12` $\to$ `12`
  - 大调五声音阶旋律调音：`[0, 2, 4, 7, 9, 12]`；小调电音旋律：`[0, 3, 5, 7, 10, 12]`
- **鬼畜人声变声特效（`voiceEffect`）**：
  - `"autotune-chorus"`：鬼畜电音双声道合唱（适合 Build-up 冲顶与副歌 Drop 主唱）
  - `"chipmunk"`：高频花栗鼠搞笑变声（适合滑稽反打、跟班附和）
  - `"demon"`：深渊恶魔低沉咆哮（适合反派质问、霸气放狠话）
  - `"robot"`：环形调制机械电音颤音（适合机械复读）
  - `"echo"`：洗脑多重回声（适合终极点题金句定格收尾）
  - `"megaphone"`：复古大喇叭中频过载喊话（适合开场定场口号）
- **夸张鬼畜画面特效（`visualEffect`，建议搭配 `effectIntensity: 95`）**：
  - `"guichu-pingpong"`：鬼畜正倒放镜像抽搐（前半段正放 + 后半段水平镜像倒放，制造经典鬼畜魔性抽动）
  - `"beat-flip"`：6Hz 卡点左右镜像高频翻转 + 红青色差闪烁
  - `"split-2h"` / `"split-3v"` / `"split-4grid"` / `"split-9grid"`：双屏镜像对称、三联屏、四宫格鬼畜矩阵、九宫格鬼畜大合唱（FFmpeg 真实多路分流克隆拼贴 + 霓虹分割线）
  - `"rainbow-rave"`：彩虹蹦迪 360° 色相高速旋转狂飙
  - `"deep-fry"`：核爆过载（表情包级超高对比度 + 锐化 + 热力红黄爆表 + 高频震屏）
  - `"soul-out"`：1.28x 放大红青偏移半透明灵魂出窍残影抽离
  - `"rgb-glitch"` / `"strobe-flash"` / `"earthquake"` / `"zoom-burst"`：故障色差、强拍频闪、重低音剧烈震屏、急推冲击
- **程序化透明视觉爆炸贴纸（`track: "overlay"`, `lane: 2`, `volume: 0`）**：
  - 支持直接将 `relativePath` 设为 `"vfx://comic-boom"`（漫威式爆炸星芒）、`"vfx://neon-slash-combo"`（霓虹双色交叉刀光）、`"vfx://electric-lightning"`（高压雷霆电弧）、`"vfx://fire-burst"`（烈焰爆燃）、`"vfx://laser-beam"`（高能贯穿激光炮）、`"vfx://shockwave-ring"`（超新星冲击波环）。
- **副歌左右护法画中画伴舞（`track: "overlay"`，必须设 `volume: 0`）**：
  - 左翼伴舞（`lane: 0`）：`overlayX: 0.02, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64, volume: 0, tweenLoop: "pendulum", tweenLoopSpeedHz: 2.1`
  - 右翼伴舞（`lane: 1`）：`overlayX: 0.72, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64, volume: 0, tweenLoop: "pulse", tweenLoopSpeedHz: 2.1`
