---
name: wf-course-narrate
description: 用「知识口播」工作流制作专业数字人课程与知识博主视频：课程结构化讲稿（script）→ 主讲人定妆照（portrait）→ 主讲人出镜底片（talking）+ 讲师口播配音（voice）→ 数字人口型同步（lipSync: video.lipSync），并同步生成核心知识点 SVG 动态板书（svgGen → svgAnim:out-gif）、课程案例 B-Roll 插图（broll）、课堂轻音乐（bgm）与字幕汇入成片时间线（timeline: output.timeline）。
workflow: course-narrate
workflow-version: 2.1.0
argument-hint: "[课程主题/讲稿内容/讲师人设形象] [画幅比例默认 9:16 或 16:9]"
when-to-use: 用户要制作知识博主口播视频、在线课程微课、数字人讲师对口型解说视频或带知识板书图解的口播成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, task_run, task_status
---

# 知识口播 · 数字人口型同步（LipSync）+ 动态板书成片 SOP（course-narrate v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，11 节点 / 16 连线）

```
课程结构化讲稿 (text.plain: script)
  ├─out→in-text──→ 主讲人定妆照·锚点 (asset.image: portrait)
  │                  └─out→in-image──→ 主讲人出镜底片 (asset.video: talking) ──out→in-video─┐
  ├─out→in-text──→ 讲师口播配音 (asset.voice: voice) ──────────────────────┬───out→in-voice─┴─→ 数字人口型同步 (video.lipSync: lipSync) ──out→in────────┐
  │                                                                        └───out→in-voice───────────────────────────────────────────────────────────┤
  ├─out→in───────→ 核心知识点动态板书 (svg.gen: svgGen) ──out→in──→ 动态板书烘焙 (svg.anim: svgAnim) ─────────────────────────────out-gif→in-image─┤
  ├─out→in-text──→ 课程案例视觉插图 B-Roll (asset.image: broll) ──────────────────────────────────────────────────────────────────────out→in-image────┤
  ├─out→in-text──→ 知识课堂轻音乐铺底 (asset.music: bgm) ─────────────────────────────────────────────────────────────────────────────out→in-music────┤
  └─out→in-text───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 课程成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "course-narrate", mode: "seed" })` 物化后，节点 `id` 即为 `script`、`portrait`、`talking`、`voice`、`lipSync`、`svgGen`、`svgAnim`、`broll`、`bgm`、`timeline`、`note`。

## 关键技术规范

1. **口型同步双源机制（`video.lipSync`）**：
   - `portrait`（主讲人定妆照）锁定讲师专业形象与演播室柔和布光；
   - `talking`（主讲人出镜底片）基于定妆照生成自然点头、眼神交流与温和手势（设置 `generateAudio: false` 静音）；
   - `voice`（讲师口播配音）朗读课程讲稿；
   - `lipSync` 同时接收 `talking:out`（`in-video`）与 `voice:out`（`in-voice`），使讲师唇形与语音毫秒级严丝合缝，输出连入 `timeline:in`。
2. **SVG 动态板书 + B-Roll 插片**：
   - `svgGen`（`svg.gen`）将核心公式/思维导图生成含 SMIL `<animate>` 动效的矢量图，经 `svgAnim`（`svg.anim`）烘焙后从 **`out-gif` 端口**连入 `timeline:in-image`；
   - `broll`（`asset.image`）生成具象案例插图连入 `timeline:in-image`，打破纯大头口播的单调感。
3. **配音防呆铁律（`asset.voice`）**：
   - `voice` 节点的 `params.generateInstruction` **只准写纯朗读台词正文**，严禁写「请用知性女声朗读」等导演说明文字（否则 TTS 会把说明词当台词念出来！）。

## 叙事结构、动态算子伸缩与 `timeline_edit` 精剪指南

1. **三段式知识叙事弧线（先规划讲稿再落图）**：
   - **黄金 3 秒反常识钩子（Hook）** → **核心模型/原理拆解（Core Mechanism，配 SVG 动态板书）** → **可执行行动清单（Actionable Takeaway，配 B-Roll 案例）**，保证逻辑严密、有头有尾。
2. **动态增删算子（`graph_edit`）**：
   - **多段分镜口播**：若课程较长（30~60s），可新增 `voice2`、`talking2`、`lipSync2` 或新增第二张 `svgGen2 → svgAnim2` 知识图解；
   - **纯板书无真人出镜**：若用户只想做「无真人出镜的动态板书解说课」，可用 `node_delete` 删除 `portrait`、`talking`、`lipSync`，或改用 `science-explainer`。
3. **时间线字级对位（`timeline_edit`）**：
   - 生成完成后可调用 `timeline_edit` 使用 `@{board}` / `@{broll}` 字级语音锚点，让 `svgAnim` 动态板书和 `broll` 插图在讲师念到对应概念的精确毫秒处弹出，或设置 `subtitleStyle: "bold" | "karaoke-pop"` 强化知识重点词。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "course-narrate", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 将完整讲稿写入 `script 的 params.`，将纯口播台词写入 `voice 的 params.`，并根据课程主题定制 `portrait`、`talking`、`svgGen`、`broll` 的提示词及画幅比例（竖屏 `"9:16"` / 横屏 `"16:9"`）。
3. `task_run({ assetId })` 一键跑通口型同步与板书动效，在 `timeline` 合成完整课程视频；如需精调画中画或花字可追加 `timeline_edit` + `timeline_export`。

## 端口与参数速查

详见 `references/ports.md`。
