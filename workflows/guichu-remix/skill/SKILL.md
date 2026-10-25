---
name: wf-guichu-remix
description: 用「鬼畜魔性卡点剪辑」工作流将任意电影、电视剧、春晚小品视频或影视名场面主题自动剪辑成 B 站亿级播放量风格的鬼畜视频（致敬《敢杀我的马？！》《改革春风吹满地》同款技法）：支持对用户导入的任意视频做词级台词转写（transcribe_audio）与关键帧表情抽帧（detect_video_keyframes），按同韵脚重组“念诗之王”顺口溜，配合 126 BPM Funky/Electro-Trap 强拍电音（asset.music）、原片动作打击乐采样（asset.sfx）与踩点人声（asset.voice），在成片时间线（output.timeline）通过 timeline_edit 自动编排 1/4→1/8→1/16 拍高频微切片复读、灵魂出窍/故障色差/频闪震屏特效、副歌左右对称画中画伴舞小窗及逐词弹跳洗脑大字幕。
workflow: guichu-remix
workflow-version: 1.0.0
argument-hint: "[电影/小品视频路径或名场面主题，如 让子弹飞·敢杀我的马 / 赵本山·改革春风吹满地] [BPM 默认 126] [画幅 16:9]"
when-to-use: 用户想要把任意一段电影、电视剧、小品或实拍视频自动剪辑成鬼畜视频（押韵念诗、鬼畜高频复读、魔性卡点、灵魂出窍、分屏伴舞），或者指定影视人物/名场面主题从零生成 B 站风格鬼畜卡点短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, asset_list, asset_import, transcribe_audio, detect_video_keyframes, grab_video_frames, analyze_video_beats, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, task_run, task_status
---

# 鬼畜魔性卡点剪辑 · B 站亿级播放六大技法自动化 SOP（guichu-remix v1.0）

> 技法溯源：深度解构 B 站百大殿堂级双冠王——**《敢 杀 我 的 马？！》**（`BV1yt4y1Q7SS`，1.29 亿播放，UP 主 `哦呼w`，素材《让子弹飞》）与 **《赵本山：我就是念诗之王！【改革春风吹满地】》**（`BV1bW411n7fY`，1.29 亿播放，UP 主 `UP-Sings`，素材赵本山春晚小品合集，同源 `@疯猴pme` 流派 Funky/Electro-Trap 强拍律动）。

## 拓扑架构与双模交付物

```
鬼畜名场面母版 (asset.image: meme_src)
  ├─out→in──→ 第1幕·名场面取景（Verse·念诗之王押韵铺垫）(image.process: act1_crop)
  │             ├─out→in────────→ 第1幕·鬼畜押韵台词与微循环卡点策划 (image.toPrompt: act1_recognize)
  │             │                   ├─out→in-text───→ 第1幕·高反差鬼畜表情精修帧 (asset.image: act1_real)
  │             │                   ├─out→in-text───→ 第1幕·魔性押韵念诗卡点视频 (asset.video: act1_video)
  │             │                   ├─out→in-text───→ 第1幕·拍桌重击与折扇脆响采样 (asset.sfx: act1_sfx)
  │             │                   └─out→in-text───→ 鬼畜魔性卡点成片时间线 (output.timeline)
  │             └─out→in-image──→ 第1幕·高反差鬼畜表情精修帧 (asset.image: act1_real)
  │                                 └─out→in-first-frame──→ 第1幕·魔性押韵念诗卡点视频 (asset.video: act1_video) ──out→in─┐
  ├─out→in──→ 第2幕·名场面取景（Build-up·翻译翻译高频复读）→ 策划 → 精修帧 → 鬼畜微循环视频 ────────────────────────out→in─┤
  └─out→in──→ 第3幕·名场面取景（Drop·敢杀我的马副歌爆发）  → 策划 → 精修帧 → 副歌咆哮爆发视频 ──────────────────────out→in─┤
第1~3幕·念诗/复读/咆哮 Hook 人声 (asset.voice: act1_voice..act3_voice) ─────────────────────────────────────────out→in-voice─┤
第1~3幕·拍桌/耳光/枪栓打击乐采样 (asset.sfx: act1_sfx..act3_sfx) ───────────────────────────────────────────────out→in-voice─┤
126BPM 疯猴流派 Funky 电音鬼畜伴奏 (asset.music: bgm) ──────────────────────────────────────────────────────────out→in-music─┴─→ 成片时间线 (output.timeline)
```

---

## 核心技法拆解：B 站亿级鬼畜六大黄金法则

| 法则编号 | 鬼畜核心手法 | 经典案例对照（《敢杀我的马》《改革春风吹满地》） | AI Art Engine 自动化落地方案 |
|---|---|---|---|
| **法则 ①** | **126 BPM 疯猴流派 Funky/Trap 强拍网格** | 均采用 120~128 BPM 四四拍电子律动，底鼓（Kick）与军鼓（Snare）极其鲜明，中高频留出人声空隙。 | 单拍周期 $T_{\text{beat}} = 60 / \text{BPM}$（126 BPM 下 $1\text{拍} \approx 0.48\text{s}$，$\frac{1}{2}\text{拍} = 0.24\text{s}$，$\frac{1}{4}\text{拍} = 0.12\text{s}$）。所有镜头切点严格吸附在 $k \cdot 0.24\text{s}$ 网格上。 |
| **法则 ②** | **台词切碎重组与“念诗之王”押韵（Sentence Mixing）** | 《改革春风》将跨年小品散句按 `an/u/i/ang` 尾韵拼成顺口溜（`白天想，夜里哭，做梦都想去首都`）；《敢杀我的马》将张麻子/黄四郎/师爷台词按 `ang` 韵重组（`大风起兮云飞扬…枪在手跟我杀四郎`）。 | 对原视频跑 `transcribe_audio` 获取词级时间戳（`segments` + `words`），挑选同韵母（如 `ang` / `i` / `u`）金句重排，或由 `image.toPrompt` 自动撰写四四拍押韵台词。 |
| **法则 ③** | **机关枪高频鬼畜复读（Stutter Roll `1/4 → 1/8 → 1/16`）** | 副歌前将核心金句切碎连发：`吹满地—吹满地—春风吹满地`、`什么叫惊喜—什么叫惊喜—什么叫TMD惊喜`。 | 用 `timeline_edit` 对同一 `sourceOffsetSec` 连续插入 4~8 个微切片，时长按 `0.48s → 0.24s → 0.24s → 0.12s × 4` 递减，配合 `speed: 1.25~1.6` 制造电音机关枪感。 |
| **法则 ④** | **鬼畜正反微循环与频闪甩镜（Micro-Loop & Whip Cut）** | 截取人物 0.2 秒最魔性微动作（赵本山缩脖拍手、白云摇头、张麻子竖指、黄四郎举单筒望远镜）高频抽动。 | 微切片交替赋予 `cameraMotion: "kenBurnsIn" / "kenBurnsOut"` 与 `transitionType: "flash" | "zoomin" | "smoothleft" | "smoothright"`，重拍叠加 `strobe-flash` / `camera-shake`。 |
| **法则 ⑤** | **副歌三联屏 / 左右护法伴舞画中画（Flanking PIP Dancers）** | 《敢杀我的马》`1:18` 高潮段左右两侧对称放置黄四郎狂笑伴舞小窗，正中央高速闪切拍桌举枪；《改革春风》扭秧歌分屏伴舞。 | 在 `output.timeline` 的 `overlay` 轨 `lane: 0`（左翼 `overlayX: 0.02, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64`）与 `lane: 1`（右翼 `overlayX: 0.72`）对称铺入人物微循环切片，设置 `tweenLoop: "pendulum"`（钟摆摇晃，`tweenLoopSpeedHz: 2.1`）或 `"pulse"`。 |
| **法则 ⑥** | **动作声当打击乐 + 逐词弹跳大字幕（Percussive SFX & Karaoke Pop）** | 把原片里的**拍桌子、扇耳光、拉枪栓、火车汽笛、拐杖敲地、嚎叫（`嗷—`）**直接对齐在重拍鼓点上，配黄红撞色粗描边大字幕。 | `asset.sfx` 采样动作声踩点铺入 `sfx` 轨；`subtitle` 轨开启 `subtitleKaraokeMode: "karaoke-pop"`（字号 `52~64`，明黄 `#FACC15` / 暴击红 `#F43F5E`），内联 `@{boom}敢杀我的马@{/boom}` 并跑 `op: "syncCues"` 自动吸附 `sfxPresetId: "boom" | "pop"` 与 `comic-boom` 贴纸。 |

---

## 两条标准自动化执行路径（MCP 路径）

### 路径 A：给定任意现有电影 / 小品 / 实拍视频文件，自动切片剪辑鬼畜（推荐原片重混）

当用户提供了本地视频路径或工程内已有电影/小品视频资产时，按以下 6 步全自动切片合成鬼畜成片：

1. **创建工作流与导入源视频**：
   - 调用 `workflow_use_installed({ id: "guichu-remix", name: "<作品名>·鬼畜卡点", generateAspectRatio: "16:9" })` 创建剧本资产 `assetId`。
   - 若视频在磁盘外部，调用 `asset_import` 导入工程，记下源视频 `videoAssetId` 与 `relativePath`。
2. **多模态台词转写与高能表情/节拍探测**：
   - 调用 `transcribe_audio({ assetId: videoAssetId, language: "zh" })` 获取全片带时间戳的 `segments` 与词级 `words`。
   - 调用 `detect_video_keyframes({ assetId: videoAssetId, maxFrames: 24 })`（或 `grab_video_frames`）定位人物夸张表情、拍桌、转身、挥手、大笑的精确秒数 `t_action`。
   - 若用户自带 BGM 则调用 `analyze_video_beats` 获取鼓点网格；若需要生成鬼畜 BGM，则仅运行图上的 `bgm`（`asset.music`）与 `act1_sfx..act3_sfx`（`asset.sfx`）节点。
3. **提炼「念诗之王」押韵顺口溜与洗脑副歌 Hook**：
   - 从 `transcribe_audio` 的台词片段中挑出 6~12 个短句/词汇（每句 `0.24s ~ 0.96s`），优先选择相同尾韵（如 `ang` / `i` / `u` / `ao`）的词尾，并选定 1 句最具爆发力的咆哮金句作为副歌 Hook（如 `[t_hook, t_hook + 0.48]`）。
4. **用 `timeline_edit` 一键铺设「四段式鬼畜魔性卡点时间线」**（设 $\text{BPM} = 126$，单拍 $B = 0.48\text{s}$，半拍 $H = 0.24\text{s}$，四分之一拍 $Q = 0.12\text{s}$）：
   - **第 1 段 · 前奏洗脑定场（Intro，`0.00s ~ 3.84s`，8 拍）**：
     - 在 `video` 轨按每 `0.48s`（1 拍）切入原片最经典的 4 个亮相动作（指定 `assetId: videoAssetId`, `sourceOffsetSec`, `durationSec: 0.48`, `volume: 1`, `visualEffect: "beat-pulse"`）。
   - **第 2 段 · 主歌「念诗之王」押韵卡点（Verse，`3.84s ~ 11.52s`，16 拍）**：
     - 将挑选出的押韵台词原片区间按 `sourceOffsetSec` 依次排入 `video` 轨（每句时长对齐 $1B = 0.48\text{s}$ 或 $2B = 0.96\text{s}$，`speed: 1.15`），相邻镜头交替设置 `transitionType: "smoothleft" | "smoothright" | "zoomin"`（`transitionInSec: 0.12`）。
   - **第 3 段 · 预副歌「机关枪高频复读」（Build-up，`11.52s ~ 15.36s`，8 拍）**：
     - 锁定副歌核心金句的起止秒数 `sourceOffsetSec = t_hook`，在 `video` 轨做 **3 级加速鬼畜复读**：
       - 前 4 拍（`11.52s ~ 13.44s`）：连续 4 次 `durationSec: 0.48`（1/4 音符复读），`visualEffect: "zoom-burst"`；
       - 第 5~6 拍（`13.44s ~ 14.40s`）：连续 4 次 `durationSec: 0.24`（1/8 音符高频复读，`speed: 1.35`），`visualEffect: "strobe-flash"`；
       - 第 7~8 拍（`14.40s ~ 15.36s`）：连续 8 次 `durationSec: 0.12`（1/16 音符极限鬼畜抽帧，`speed: 1.6`），`visualEffect: "rgb-glitch"`，直冲高潮 Drop！
   - **第 4 段 · 副歌高潮「灵魂出窍 + 左右护法画中画伴舞」（Chorus Drop，`15.36s ~ 23.04s`，16 拍）**：
     - **中央主画面（`track: "video"`）**：每 `0.48s` 高速切换原片拍桌、咆哮、拔刀、大笑等爆发镜头，交替挂载 `visualEffect: "soul-out"`（灵魂出窍）、`"earthquake"`（剧烈地动）、`"rgb-glitch"`（故障色差），转场使用 `flash`（闪白卡点，`transitionInSec: 0.15`）；
     - **左右对称护法伴舞小窗（`track: "overlay"`）**：
       - 左护法（`lane: 0`）：`overlayX: 0.02, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64`，取人物魔性摇头/大笑片段，`tweenIn: "popIn"`, `tweenLoop: "pendulum"`, `tweenLoopSpeedHz: 2.1`, `tweenLoopIntensity: 1.4`；
       - 右护法（`lane: 1`）：`overlayX: 0.72, overlayY: 0.18, overlayWidth: 0.26, overlayHeight: 0.64`，对称放置伴舞片段，`tweenIn: "popIn"`, `tweenLoop: "pulse"`, `tweenLoopSpeedHz: 2.1`, `tweenLoopIntensity: 1.4`；
     - **逐词弹跳大字幕与语义锚点音效（`track: "subtitle"` + `op: "syncCues"`）**：
       - 字幕片段设置 `fontSize: 56`, `textColor: "#FACC15"`, `subtitleKaraokeMode: "karaoke-pop"`, `subtitleHighlightColor: "#F43F5E"`，在爆点词上标注 `@{boom}敢杀我的马？！@{/boom}`；
       - 追加 `overlay` 贴纸片段（`stickerId: "comic-boom"`, `anchorCueTag: "boom"`）与 `sfx` 片段（`sfxPresetId: "boom"`, `anchorCueTag: "boom"`），并在 `timeline_edit` 末尾执行 `{ op: "syncCues" }` 与 `{ op: "applyTemplate", templateId: "high-energy-beat" }`。
5. **自动导出与预览验证**：
   - 调用 `timeline_export({ assetId })` 合成最终 MP4 成片，并用 `timeline_preview` 抽帧校验分屏伴舞、大字幕与卡点对齐效果。

---

### 路径 B：给定电影名 / 角色主题或单张剧照，AI 全自动生成三幕鬼畜大片

当用户没有现成视频文件、而是指定一个主题（如“用《让子弹飞》做一个鬼畜视频”、“给诸葛亮骂王朗做个念诗鬼畜”、“给老板开会画饼做个鬼畜卡点”）时：

1. `workflow_use_installed({ id: "guichu-remix", name: "<主题>·鬼畜魔性卡点", generateAspectRatio: "16:9" })`。
2. `graph_read({ assetId, includeParams: true })` 获取全部 20 个节点 ID。
3. 用 `graph_edit` 覆写：
   - `meme_src`：角色与名场面三联戏剧构图母版提示词；
   - `act1_recognize` ~ `act3_recognize`：分别写入第 1 幕【主歌押韵念诗】、第 2 幕【预副歌高频复读质问】、第 3 幕【副歌咆哮爆发】的台词与动作策划；
   - `act1_video` ~ `act3_video`：保持 `generateAudio: true` 与 `generateFrameMode: "first"`，让视频模型直接生成带人物夸张表情与踩点咆哮原声的素材；
   - `act1_voice` ~ `act3_voice`：写入押韵顺口溜与副歌咆哮 Hook 纯台词正文；
   - `act1_sfx` ~ `act3_sfx` 与 `bgm`：写入 126 BPM 强拍打击音效与 Funky/Trap 电音伴奏描述。
4. 调用 `task_run({ assetId })` 一键跑通全图生成，随后可额外调用 `timeline_edit` 在 `output.timeline` 上叠加 `1/8 拍` 微切片复读、左右护法伴舞画中画（`overlay` 轨 `lane 0/1`）与 `soul-out` 灵魂出窍特效，导出魔性鬼畜成片。

## 端口与参数速查

详见 `references/ports.md`。
