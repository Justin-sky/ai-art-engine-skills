---
name: wf-science-explainer
description: 用「科普动画全流程」工作流把一篇科普文章或知识点一键转化为含风格锚点、SVG 矢量原理图解动画、三幕电影级科普视频、逐镜专业配音、轻快 BGM 与自动字幕的完整科普成片（output.timeline）。当用户说「根据文章生成科普视频」「做一条科普动画/知识解说短片」时优先加载本技能。
workflow: science-explainer
workflow-version: 1.0.0
argument-hint: "[科普文章/知识点主题] [画幅比例 16:9 或 9:16]"
when-to-use: 用户希望把一段科普文章、百科知识、科学原理或技术概念转化为带配音、字幕、矢量原理图解与多镜头画面的完整科普视频时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 科普动画全流程 · 工业级多轨成片 SOP（science-explainer）

## 拓扑架构与交付物

```
科普脚本与字幕 (text.plain)
  ├─out→in-text──→ 科普视觉风格锚点 (asset.image)
  │                  ├─out→in-image──→ 镜头1·现象引入关键帧 (asset.image) ──out→in-image──→ 镜头1·现象引入视频 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 镜头2·核心机制关键帧 (asset.image) ──out→in-image──→ 镜头2·核心机制视频 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 镜头3·生活应用关键帧 (asset.image) ──out→in-image──→ 镜头3·生活应用视频 (asset.video) ──out→in-clips─┤
  ├─out→in───────→ SVG 原理图解生成 (svg.gen) ──out→in──→ SVG 原理动效合成 (svg.anim) ───────────────────────────────out-video→in-clips─┤
  ├─out→in-text──→ 镜头1·引入配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 镜头2·原理解析配音 (asset.voice) ──────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 镜头3·总结配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 科普轻音乐 BGM (asset.music) ──────────────────────────────────────────────────────────────────────out→in-music─────┤
  └─out→in-subtitles──────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 导演分镜与提示词工程规范（Agent 改写脚本与提示词必读）

当用户给出一篇科普文章或一句话主题时，Agent 必须通过 `graph_edit` 将文章转化为结构化导演方案，严禁保留模板占位符直接开跑：

| 阶段节点 | 角色定位 | 提示词与参数覆写公式 |
|---|---|---|
| `script` (`text.plain`) | 全局叙事与字幕源 | 将文章提炼为 **3 段式口播字幕稿**（每句以句号/感叹号/问号或换行分隔，以便 `output.timeline` 自动切分字幕时间轴）：① 现象悬念引入 → ② 核心微观机制拆解 → ③ 现实生活启发。 |
| `styleAnchor` (`asset.image`) | 视觉母版锁（锁色板与画风） | 公式：`Kurzgesagt / 现代扁平科学插画设定表 + 主题核心视觉符号 + 统一明快撞色色板（深空蓝底 + 青绿/暖橙高亮） + 干净无文字无水印`。其 `out` 已连入 3 个分镜图的 `in-image`，强制三幕画面色调与图形语言 100% 统一。 |
| `svgDiagram` (`svg.gen`) → `svgAnim` (`svg.anim`) | 矢量机制图解动效 | 在 `svgDiagram` 的 `generateInstruction` 中写明核心因果/流程结构图（如「ATP 能量循环四步箭头图」「温室效应红外反射示意图」），由 `svg.anim` 自动生成平滑描边与脉冲补间并输出 MP4 视频流汇入时间线。 |
| `shot1Img`~`shot3Img` (`asset.image`) | 三幕叙事关键帧 | 严格遵循「1镜微距/悬念特写 → 2镜机制剖面展示 → 3镜宏观生活场景」景别递进，全部显式声明 `严格沿用参考图色板与扁平矢量材质，画面干净无文字`。 |
| `shot1Vid`~`shot3Vid` (`asset.video`) | 三幕图生视频 | 写明具体镜头运动（如 `缓慢推进 dolly-in`、`微观粒子沿轨迹流动`、`平稳拉远揭示全景`），且设置 `generateAudio: false` 避免视频底噪干扰人声配音。 |
| `voice1`~`voice3` (`asset.voice`) | 逐镜同步配音 | **重要规则**：`asset.voice` 的 `generateInstruction` 是直接送入 TTS 朗读的台词正文！必须将第 1、2、3 幕对应的口播词分别写入 `voice1`、`voice2`、`voice3` 的 `generateInstruction`，**只写纯朗读台词，严禁写入"请用温和语气朗读"等导演指令**（否则 TTS 会把指令念出来）。 |

## 标准执行步骤（MCP 自动化路径）

1. **落盘工作流**：调用 `workflow_use_installed({ id: "science-explainer", name: "<科普主题>·科普视频", generateAspectRatio: "16:9" })`（竖屏科普传 `"9:16"`）。
2. **读取节点 ID**：调用 `graph_read({ assetId, includeParams: true })` 获取图上所有节点的真实 `id`。
3. **一次性注入定制内容**：调用 `graph_edit` 批量更新：
   - `script` 的 `params.text`（完整三段式字幕文稿）；
   - `styleAnchor`、`svgDiagram`、`shot1Img`、`shot2Img`、`shot3Img`、`shot1Vid`、`shot2Vid`、`shot3Vid` 的 `params.generateInstruction`；
   - `voice1`、`voice2`、`voice3` 的 `params.generateInstruction`（三段纯口播台词，与 `script` 字幕逐段对应）；
   - 若用户只需要三幕写实/插画视频而不想要额外的 SVG 图解片头，可断开 `svgAnim` 到 `timeline` 的连线，避免空白占位。
4. **启动全图合成**：调用 `task_run({ assetId })` 拿到 `mcpTaskId`，用 `task_status({ mcpTaskId })` 跟进直至 `output.timeline` 完成多轨音画对位与字幕压制。
5. **双击进时间线微调**：提醒用户可双击画布右侧的「成片时间线」节点（`output.timeline`）进入多轨非编视图，实时预览画面、拖动剪辑块或调整字幕样式后一键导出 MP4 成片。

## 端口与参数速查

详见 `references/ports.md`。
