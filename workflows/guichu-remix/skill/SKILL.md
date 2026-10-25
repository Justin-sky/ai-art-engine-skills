---
name: wf-guichu-remix
description: 用「鬼畜魔性卡点剪辑」技能将任意电影名、电视剧、春晚小品、B 站链接或本地视频自动剪辑成 B 站亿级播放量风格的鬼畜视频（致敬《敢杀我的马？！》《改革春风吹满地》同款技法）：只需给出电影名/角色名或链接，即可通过 web_fetch_video 自动去 B 站检索并下载高清原片名场面片段，结合词级台词转写（transcribe_audio）与关键帧表情探测（detect_video_keyframes），按同韵脚重组“念诗之王”顺口溜，配合 126 BPM Funky/Electro-Trap 强拍电音（asset.music）与打击乐采样（asset.sfx），在成片时间线（output.timeline）通过 timeline_edit 自动编排 1/4→1/8 拍高频微切片复读、半音阶梯爬音与人力VOCALOID旋律变调（pitchSemitones + voiceEffect）、灵魂出窍特效、左右护法静音伴舞及逐词弹跳大字幕，并支持用 graph_edit 动态混搭任意画布节点。
workflow: guichu-remix
workflow-version: 1.4.0
argument-hint: "[电影名/角色名/B站链接/本地视频路径，如 让子弹飞·敢杀我的马 / 赵本山·改革春风吹满地 / 亮剑李云龙] [BPM 默认 126] [画幅 16:9]"
when-to-use: 用户想要把任意一部电影（只给电影名即可自动联网下载对应名场面原片）、电视剧、小品、B 站视频或本地视频自动剪辑成鬼畜视频（故事连续完整、四幕因果剧情 + 关键包袱点鬼畜卡点、半音变调爬音、高频复读、灵魂出窍、分屏伴舞），或跨技能组合创作鬼畜短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, web_fetch_video, asset_list, asset_import, transcribe_audio, detect_video_keyframes, grab_video_frames, analyze_video_beats, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, timeline_preview, task_run, task_status, render_svg
---

# 鬼畜魔性卡点剪辑 · 「故事主干 + 卡点爆点」双驱架构（guichu-remix v1.4）

> 技法溯源：深度解构 B 站百大殿堂级双冠王——**《敢 杀 我 的 马？！》**（`BV1yt4y1Q7SS`，1.29 亿播放，UP 主 `哦呼w`，素材《让子弹飞》）与 **《赵本山：我就是念诗之王！【改革春风吹满地】》**（`BV1bW411n7fY`，1.29 亿播放，UP 主 `UP-Sings`，素材赵本山春晚小品合集，同源 `@疯猴pme` 流派 Funky/Electro-Trap 强拍律动与人力 VOCALOID 半音变调）。

## 核心创作哲学：70% 连续故事主干 + 30% 鬼畜卡点锚点（拒绝无逻辑碎切！）

为什么有些鬼畜剪完后让人觉得「不知所云、原片逻辑全丢」？因为把所有台词都一刀切成了 `0.96s` 碎片，既腰斩了 2~3 秒的完整长句，又打乱了原片的「起因 $\to$ 升级 $\to$ 高潮 $\to$ 结局」因果链！
真正的顶级剧情鬼畜必须遵循 **「先规划四幕故事主干与鬼畜卡点蓝图，再按 70/30 松紧呼吸比剪辑」**：
1. **70% 连续叙事歌唱句（Story Singing Spine · 严禁平铺直叙原调干声！）**：按原片故事发生的时间与因果顺序组织四幕剧情；**每一句主干台词必须拆分为「前半句节奏起唱段（`speed: 0.95~1.20`，`pitchSemitones: +2/+3`，`voiceEffect: "autotune-chorus"`）+ 后半句核心词/字尾慢放拉长歌唱段（`speed: 0.48~0.56` 拉长 `1.8x~2.1x`，`pitchSemitones: +3~+5` 或 `-2`，`voiceEffect: "echo"` 自动触发 5.6Hz 歌唱颤音与立体声延音）」**（如：`那你也是` $\to$ `农~村~~~的~呀~~~~~~🎵`、`我的拳头` $\to$ `想~喝~~~血~啦~~~~~~🎵`），既把整句台词的意思 100% 完整唱完，又让每句尾字都拉长变成《敢杀我的马》同款旋律歌声！**严禁将台词设为 `speed: 1` + `pitchSemitones: 0` 的平淡原声！**
2. **30% 鬼畜卡点锚点（Guichu Hook Drops）**：在每幕的**关键包袱点、出牌/亮招瞬间、情绪最高潮**精准插入鬼畜大招——截取刚刚那句台词里的 **2~4 字灵魂短词**（如「有钱的！」「你真的很装」「对着你锤！」「向下兼容」），进行 `0.48s`（1 拍，`speed: 0.65~0.72` 拉长拖腔）或 `0.24s`（半拍，`speed: 0.78~0.85`）的 3~8 连发复读 + `pitchSemitones: [0, 2, 4, 5]` / `[2, 3, 5, 6]` 自然男/女声歌唱音区阶梯爬升（避免 `> +7st` 尖锐花栗鼠失真）+ `voiceEffect: "echo" | "autotune-chorus"`，让鬼畜卡点成为引爆剧情高潮的「旋律放大器」！

## 种子拓扑（Seed Skeleton）+ 动态自由组合架构

本技能采用 **「4 节点预连线种子骨架 + 开放式画布算子自由混搭」** 架构——`workflow.json` 只提供最精简的 4 节点核心闭环（且连线全部预置就绪），绝不限制你的节点组合创造力：

```
[核心种子骨架（4 节点 3 连线，通过 workflow_use_installed 一键带连线落图）]
  鬼畜原片/名场面素材源 (asset.video: src-video) ──out→in───────┐
  126BPM Funky 电音伴奏 (asset.music: bgm) ────────out→in-music─┼─→ 鬼畜魔性卡点成片时间线 (output.timeline: timeline-out)
  鬼畜重拍打击采样 (asset.sfx: sfx-boom) ──────────out→in-voice─┘

[大模型可按需用 graph_edit (node_upsert + edge_connect) 自由挂载的扩展模块]
  ├─ [多源名场面混剪] 新增 src-video-2 / src-video-3 (asset.video, locked: true) ──out→in──→ timeline-out
  ├─ [抠图魔性摇晃大头] 新增 image.process (imageProcessTool: "cutout") ──out→in-image──→ timeline-out (overlay 轨伴舞)
  ├─ [SVG 矢量鬼畜战力表/格斗血条/弹幕风暴] 新增 svg.gen ──out→in──→ svg.anim ──out-gif→in-image──→ timeline-out
  ├─ [AI 毒舌 Rap 念诗 / 多角色对呛] 新增 asset.voice / asset.dialogue ──out→in-voice──→ timeline-out
  └─ [跨技能融合] 调用 workflow_use_installed({ id: "<其它技能>", mode: "merge" }) 将其它技能节点并入同一画布共享 timeline-out！
```

---

## 核心技法拆解：B 站亿级鬼畜七大黄金法则

| 法则编号 | 鬼畜核心手法 | 经典案例对照（《敢杀我的马》《改革春风吹满地》） | AI Art Engine 自动化落地方案 |
|---|---|---|---|
| **法则 ①** | **126 BPM 疯猴流派 Funky/Trap 强拍网格** | 均采用 120~128 BPM 四四拍电子律动，底鼓（Kick）与军鼓（Snare）极其鲜明，中高频留出人声空隙。 | 单拍周期 $T_{\text{beat}} = 60 / \text{BPM}$（126 BPM 下 $1\text{拍} \approx 0.48\text{s}$，$\frac{1}{2}\text{拍} = 0.24\text{s}$）。所有镜头切点严格吸附在 $k \cdot 0.24\text{s}$ 网格上。 |
| **法则 ②** | **台词切碎重组与“念诗之王”押韵（Sentence Mixing）** | 《改革春风》将跨年小品散句按 `an/u/i/ang` 尾韵拼成顺口溜（`白天想，夜里哭，做梦都想去首都`）；《敢杀我的马》将张麻子/黄四郎/师爷台词按 `ang` 韵重组。 | 用 `web_fetch_video` 自动下载原片名场面后，跑 `transcribe_audio` 获取词级时间戳（`segments` + `words`），挑选同韵母金句重排，确保 `sourceOffsetSec` 100% 对应原片真实发声秒数。 |
| **法则 ③** | **机关枪高频鬼畜复读（Stutter Roll `1/4 → 1/8`）** | 副歌前将核心金句切碎连发：`吹满地—吹满地—春风吹满地`、`什么叫惊喜—什么叫惊喜—什么叫TMD惊喜`。 | 用 `timeline_edit` 对同一 `sourceOffsetSec` 连续插入 4 个 `0.48s`（1 拍）+ 8 个 `0.24s`（半拍，`speed: 1.25`）微切片，制造层层逼近的机关枪蓄力感。 |
| **法则 ④** | **原声尾字慢放拉长变唱歌 + 五声音阶旋律变调（Vowel-Stretched Singing & Pitch Shift）** | 《敢杀我的马》（`BV1yt4y1Q7SS`）调音灵魂：将演员原本短促的说话字尾慢放拉长 `1.8x~2.2x` 并移调至五声音阶唱区（`-2st ~ +5st`），配合歌唱颤音（Vibrato）把每句台词都变成悠扬歌声（如`一~日~千~里~~~`、`恍~如~隔~世~~~`）！ | 在 `timeline_edit` 中对每句主干台词采用**「前半句起唱 + 后半句尾字拉长变唱歌」双切片法**：<br>• **前半句节奏起唱段（Lead-In）**：设 `speed: 0.95~1.20`、`pitchSemitones: 2` 或 `3`、`voiceEffect: "autotune-chorus"`，紧凑卡在正拍起唱；<br>• **后半句原声拉长歌唱段（Vowel-Stretched Singing Tail）**：截取该句最后 2~4 个核心字（通过 `words` 时间戳精准定位），设 **`speed: 0.48~0.56`（慢放拉长 `1.8x~2.1x`）**、**`pitchSemitones: 4` 或 `5`（或低音 `-2`）**、**`voiceEffect: "echo"`**（引擎在 `speed < 0.85` 时自动注入 `5.6Hz` 歌唱颤音 `vibrato` + 共振峰护嗓补偿 EQ + 立体声合唱延音）；<br>• **蓄力阶梯爬音（Pitch Riser）**：4 连发 `0.48s` 设 `speed: 0.68, pitchSemitones: [0, 2, 4, 5], voiceEffect: "echo"`；4 连发 `0.24s` 设 `speed: 0.80, pitchSemitones: [2, 3, 5, 6], voiceEffect: "autotune-chorus"`（保持在自然人声唱区，避免 `> +7st` 尖细失真）。 |
| **法则 ⑤** | **夸张鬼畜画面变形与高能特效轰炸（Exaggerated Guichu Visuals & VFX）** | 画面绝不能平铺直叙！必须密集运用**正倒放抽搐、卡点左右镜像翻转、二/四/九宫格鬼畜分屏、彩虹蹦迪色相狂飙、核爆过载、灵魂出窍**与**刀光/雷电/爆炸透明特效贴纸**！ | • **主视频轨夸张鬼畜特效（`visualEffect`）**：高频轮换 `"guichu-pingpong"`（正倒放镜像抽搐）、`"beat-flip"`（6Hz 卡点左右翻转）、`"split-2h"`（双屏镜像对称）、`"split-4grid"`（四宫格鬼畜矩阵）、`"split-9grid"`（九宫格鬼畜大合唱）、`"rainbow-rave"`（彩虹蹦迪色相旋转）、`"deep-fry"`（表情包核爆过载）、`"soul-out"`（1.28x 灵魂出窍残影）、`"earthquake"`（剧烈震屏），配合 `effectIntensity: 95` 与 `cameraMotion: "impactPush"`；<br>• **重拍程序化 VFX 视觉贴纸（`track: "overlay", lane: 2`）**：在每个鬼畜爆点叠加 `"vfx://comic-boom"`、`"vfx://neon-slash-combo"`、`"vfx://electric-lightning"`、`"vfx://fire-burst"`、`"vfx://laser-beam"`、`"vfx://shockwave-ring"`（`volume: 0`）。 |
| **法则 ⑥** | **副歌三联屏 / 左右护法静音伴舞（Flanking PIP Dancers）** | 《敢杀我的马》高潮段左右两侧对称放置狂笑伴舞小窗，正中央高速闪切拍桌举枪。 | 在 `output.timeline` 的 `overlay` 轨 `lane: 0`（左翼 `overlayX: 0.02, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64, volume: 0`）与 `lane: 1`（右翼 `overlayX: 0.72, volume: 0`）对称铺入微循环切片，设置 `tweenLoop: "pendulum"` 或 `"pulse"`（**必须 `volume: 0` 静音以免干扰主轨人声**）。 |
| **法则 ⑦** | **动作声当打击乐 + 逐词弹跳大字幕（Percussive SFX & Karaoke Pop）** | 把原片里的**拍桌子、扇耳光、拉枪栓、火车汽笛、嚎叫**直接对齐在重拍鼓点上，配黄红撞色粗描边大字幕。 | `asset.sfx` 采样铺入 `sfx` 轨；`subtitle` 轨开启 `subtitleKaraokeMode: "karaoke-pop"`（字号 `54~64`，明黄 `#FACC15` / 暴击红 `#F43F5E`），内联 `@{boom}敢杀我的马@{/boom}` 并跑 `op: "syncCues"` 自动对齐爆点音效与贴纸。 |

---

## 标准自动化执行 SOP（MCP 路径）

1. **一键落下 4 节点预连线种子拓扑 + 获取原片素材**：
   - 调用 `workflow_use_installed({ id: "guichu-remix", mode: "seed" })`——系统会在当前画布一键创建并连好 `src-video`、`bgm`、`sfx-boom` $\to$ `timeline-out`（含完整连线与自动布局），返回画布 `assetId`！
   - **获取原片视频**：
     - 若用户给出**电影名 / 电视剧名 / 小品名 / 角色名**，调用 `web_fetch_video({ query: "<电影名/角色名> 名场面 原片", count: 1, maxDurationSec: 240, preferOriginalMaterial: true })` 下载无水印原片至 `Cache/imports/...`；
     - 若用户给出 **B 站链接或 BV 号**，调用 `web_fetch_video({ url: "<链接或BV号>", maxDurationSec: 240 })`；
     - 若用户给出**本机视频路径**，调用 `asset_import`。
   - 调用 `graph_edit` 将下载好的 `relativePath` 锁定绑定到 `src-video` 节点：
     `{ op: "node_update", nodeId: "src-video", locked: true, params: { sourceRelativePath: "<relativePath>" } }`（若你额外新增了创意节点，**必须同时附带 `edge_connect` 连入 `timeline-out`**；**严禁调用 `asset_create({ type: "screenplay" })` 另建独立剧本资产**）。
2. **多模态台词转写、关键帧探测与电音伴奏生成**：
   - 调用 `transcribe_audio({ relativePath, language: "zh" })` 获取全片 `segments` 与词级 `words` 时间戳。
   - 调用 `detect_video_keyframes({ relativePath })` 定位夸张表情与动作秒数。
   - 调用 `task_run({ assetId, nodeIds: ["bgm", "sfx-boom"] })` 生成 126 BPM Funky 电音伴奏与重拍打击采样。
3. **【必做】先制定「四幕因果故事主干 + 鬼畜卡点锚点」规划蓝图，再用 `timeline_edit` 精编**（直接传画布 `assetId`；设 $\text{BPM} = 126$，单拍 $B = 0.48\text{s}$，半拍 $H = 0.24\text{s}$，总时长建议 `30s ~ 40s` 以保证故事完整）：
   - **第一幕 · 起因与交锋（Act 1: Setup & Conflict，约 `0.00s ~ 9.60s`，20 拍）**：
     - **连续叙事歌唱句（前半句起唱 + 后半句拉长变唱歌）**：按原片开头顺序选取 3~4 句完整对话交代背景与矛盾起因；**每句拆为两段**：前半句起唱段（`durationSec: 0.64~1.20, speed: 0.95~1.15, pitchSemitones: 2, voiceEffect: "autotune-chorus"`）+ 后半句核心词/字尾慢放拉长歌唱段（通过 `words` 时间戳定位句尾 2~4 字，设 **`durationSec: 0.80~1.20, speed: 0.48~0.56, pitchSemitones: 4 或 5, voiceEffect: "echo"`**，将尾字拉长 1.8~2.1 倍化作悠扬拖腔歌声）；
     - **🔥 鬼畜卡点锚点 #1（幕尾包袱拉长连击）**：紧接第一幕最后一句狠话（如「有钱的！」「对着你锤！」），截取其核心词做 2~3 次 `0.48s` 慢放拉长阶梯升调连击（`speed: 0.65, pitchSemitones: 3 → 5, voiceEffect: "echo"`）。
   - **第二幕 · 矛盾白热化升级（Act 2: Escalation & Duel，约 `9.60s ~ 19.20s`，20 拍）**：
     - **连续叙事歌唱句**：顺着时间线推进冲突升级；**每句同样执行「前半句紧凑起唱（`+2st/+3st`）+ 后半句尾字慢放拉长 `speed: 0.48~0.54` 歌唱拖腔（`+4st/+5st` 或低音 `-2st`）」**（如「我的拳头」$\to$「想~喝~~~血~啦~~~~~~🎵」、「这本来就是」$\to$「等~价~~~交~换~~~~~~🎵」），严禁出现 `speed: 1, pitchSemitones: 0` 的平直干声！
   - **第三幕 · 鬼畜狂飙高潮与决战爆发（Act 3: Guichu Climax & Drop，约 `19.20s ~ 28.80s`，20 拍）**：
     - **🔥 鬼畜卡点锚点 #2 · 拉长歌唱阶梯爬升 + 多宫格裂变（Pitch Riser，`19.20s ~ 22.08s`，6 拍）**：
       - 前 4 拍：截取 2~4 字爆点词连发 4 次 `durationSec: 0.48`（`speed: 0.68` 拉长歌唱），设置 `pitchSemitones: 0, 2, 4, 5` + `voiceEffect: "echo"`，画面依次裂变为 `visualEffect: "guichu-pingpong" → "beat-flip" → "split-2h" → "split-4grid"`（`effectIntensity: 98`）；
       - 后 2 拍：截取决战词连发 4 次 `durationSec: 0.24`（`speed: 0.80`），设置自然高音冲顶 `pitchSemitones: 2, 3, 5, 6` + `voiceEffect: "autotune-chorus"`，画面直接引爆 `"split-9grid"`（九宫格矩阵）与 `"rainbow-rave"`（彩虹蹦迪）+ `"deep-fry"`（核爆过载）！
     - **🔥 鬼畜卡点锚点 #3 · 副歌名场面拉长高音大合唱 + 左右护法伴舞 + VFX 特效贴纸（Chorus Drop，`22.08s ~ 28.80s`，14 拍）**：
       - 中央主轨（`track: "video"`）：按决战交锋顺序切入核心宣言与大招台词，每句分为「起唱段（`+2st/+3st`）+ 尾字慢放 `0.48x~0.52x` 拉长高音歌唱段（`+4st/+5st, voiceEffect: "echo"`）」，并叠加 `visualEffect: "soul-out" | "split-9grid" | "deep-fry" | "rainbow-rave"`；
       - 左右护法伴舞小窗（`track: "overlay"`，`lane: 0` 左翼 `overlayX: 0.02`，`lane: 1` 右翼 `overlayX: 0.72`，**强制 `volume: 0` 静音**，`tweenLoop: "pendulum" / "pulse"`）；
       - 程序化 VFX 视觉爆炸层（`track: "overlay"`，`lane: 2`，`volume: 0`）：在各幕重拍爆点铺入 `relativePath: "vfx://comic-boom"`、`"vfx://neon-slash-combo"`、`"vfx://electric-lightning"`、`"vfx://fire-burst"`、`"vfx://laser-beam"`、`"vfx://shockwave-ring"`。
   - **第四幕 · 结局反转与余音回声收束（Act 4: Resolution & Payoff，约 `28.80s ~ 36.48s`，16 拍）**：
     - **连续叙事歌唱收束**：切入决战后的震惊反应与全片点题升华句，同样将每句核心尾词做 `speed: 0.50~0.54` 慢放拉长歌唱（`pitchSemitones: +4 / +5, voiceEffect: "echo"`）；
     - **终极延长音定格**：最后一句反转金句的尾词设 **`durationSec: 1.44, speed: 0.48, pitchSemitones: 5, voiceEffect: "echo"`**（2.1 倍超长拖腔颤音高歌）、`freezeTailSec: 0.36` 与 `fadeOutSec: 0.65`；`video`、`music`（`loop: true, fadeOutSec: 0.8`）与 `subtitle` 轨严格在同一秒对齐收尾！
   - **逐词弹跳大字幕与爆点锚点（`track: "subtitle"` + `op: "syncCues"`）**：开启 `subtitleKaraokeMode: "karaoke-pop"`，字幕文本与每一句完整台词及鬼畜连发词 100% 同步对应，内联 `@{boom}...@{/boom}` 并跑 `syncCues`（切勿调用 `applyTemplate`）。
4. **单次导出与抽帧自检**：
   - 调用一次 `timeline_export({ assetId })` 导出成片 MP4，并用 `timeline_preview` 抽帧核验画面。

## 端口与参数速查

详见 `references/ports.md`。
