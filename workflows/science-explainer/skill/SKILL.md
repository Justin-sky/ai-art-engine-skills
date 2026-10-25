---
name: wf-science-explainer
description: 用「科普动画」工作流把硬核科学/医学/工程/商业原理解析转化为双路动态矢量图解 + 高清科普插画 + 逐镜旁白成片：科普脚本（script）→ 核心概念 SVG 动态图解（svgGen1 → svgAnim1:out-gif）+ 微观机制 SVG 动态图解（svgGen2 → svgAnim2:out-gif）+ 高清科普插画（illust3）+ 三段逐镜科普配音（voice1~voice3）+ 探索感背景音乐（bgm）→ 科普成片时间线（timeline: output.timeline）。
workflow: science-explainer
workflow-version: 1.1.0
argument-hint: "[科普主题/核心科学原理/目标受众]"
when-to-use: 用户想要制作科普动画短片、硬核科学原理解析视频（类似 Kurzgesagt / 回形针 / 3Blue1Brown 风格）、带 SVG 动态机制图解的知识科普成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, render_svg, task_run, task_status
---

# 科普动画 · 双路 SVG 动态图解与多轨合成 SOP（science-explainer v1.1）

## 默认种子拓扑与交付物（Seed Skeleton，12 节点 / 17 连线）

```
科普文章与分镜旁白稿 (text.plain: script)
  ├─out→in───────→ 第1镜·核心概念动态矢量图 (svg.gen: svgGen1) ──out→in──→ 第1镜·动效烘焙 (svg.anim: svgAnim1) ──out-gif→in-image─┐
  ├─out→in───────→ 第2镜·微观机制动态图解 (svg.gen: svgGen2) ───out→in──→ 第2镜·动效烘焙 (svg.anim: svgAnim2) ──out-gif→in-image─┤
  ├─out→in-text──→ 第3镜·高清科普插画 (asset.image: illust3) ────────────────────────────────────────────────────out→in-image─────┤
  ├─out→in-text──→ 第1镜·科普旁白配音 (asset.voice: voice1) ─────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 第2镜·科普旁白配音 (asset.voice: voice2) ─────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 第3镜·科普旁白配音 (asset.voice: voice3) ─────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 探索感科普背景音乐 (asset.music: bgm) ────────────────────────────────────────────────────────out→in-music─────┤
  └─out→in-text───────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 科普成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "science-explainer", mode: "seed" })` 物化后，画布上的节点 `id` 与 `workflow.json` 的 `key` 完全一致（`script`、`svgGen1`、`svgAnim1`、`svgGen2`、`svgAnim2`、`illust3`、`voice1`、`voice2`、`voice3`、`bgm`、`timeline`、`note`），可直接传入 `graph_edit`。

## 科普视觉与配音核心规范

1. **三幕递进式科普叙事（先定因果逻辑再填节点）**：
   - **第 1 镜（宏观现象/悬念引入，`svgGen1 → svgAnim1` + `voice1`）**：从日常直觉或反常识现象切入，用清晰的几何扁平动态图展示宏观概念；
   - **第 2 镜（微观/底层机制拆解，`svgGen2 → svgAnim2` + `voice2`）**：深入分子/算法/物理能量传递层，利用 SVG SMIL `<animate>` / `<animateTransform>` 展示粒子流动、齿轮咬合、信号传导与中文标注箭头；
   - **第 3 镜（具象应用与总结升华，`illust3` + `voice3`）**：用高审美科普插画（或扩展为 3D 微观摄影视频）收束全片，点明科学原理对现实世界的意义。
2. **SVG 动效烘焙端口规范（`svg.gen → svg.anim`）**：
   - `svgGen1` / `svgGen2` 提示词须要求：**深色科研背景（`#0F172A`）、高对比霓虹科研配色（青/琥珀/品红）、中文标签排版整洁、内嵌原生 `<animate>` / `<animateTransform>` 循环动效**；
   - `svgAnim1` / `svgAnim2` 将动态 SVG 烘焙为 GIF 动图，通过 **`out-gif` 端口**连入 `timeline:in-image`。
3. **旁白配音防呆铁律（`voice1`~`voice3`）**：
   - `voice1`、`voice2`、`voice3` 的 `params.generateInstruction` **必须只写该镜要被 TTS 朗读的纯旁白句子**，严禁写「生动抓耳的科普音色，朗读第1镜旁白：」等前缀说明！

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

- **将第 3 镜升级为 3D 微观动态视频（`illust3 → vid3`）**：
  若用户想要震撼的微观流体/宇宙星体动态镜头，可用 `graph_edit` 新增 `vid3`（`typeId: "asset.video"`, `params: { generateAudio: false }`），连接 `illust3:out → vid3:in-image` 与 `vid3:out → timeline:in`。
- **扩展为 4~6 镜深度科普长片**：
  按需复制 `svgGenN → svgAnimN` 或 `illustN → vidN` + `voiceN` 分支，依次连入 `timeline` 的 `in-image` / `in` 与 `in-voice`。
- **混搭数字人主讲（`course-narrate` 融合）**：
  使用 `workflow_use_installed({ id: "course-narrate", assetId, mode: "merge" })` 或新增 `video.lipSync`，实现「数字人学者出镜讲解 + 全屏 SVG 动态原理图切镜」的纪录片效果。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "science-explainer", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 将三段分镜旁白稿写入 `script 的 params.`，将纯朗读旁白分别写入 `voice1`~`voice3` 的 `params.generateInstruction`，并定制 `svgGen1`、`svgGen2`、`illust3`、`bgm` 的具体科学主题提示词。
3. `task_run({ assetId })` 一键跑通全图并合成 `timeline` 科普成片；需要精细调整字幕与音画对位时可调用 `timeline_edit`。

## 端口与参数速查

详见 `references/ports.md`。
