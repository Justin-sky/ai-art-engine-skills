---
name: wf-storybook-narrate
description: 用「有声绘本故事」工作流把一段童话、寓言或睡前故事转化为角色画风高度统一的儿童绘本动画短片：主角水彩设定锚点锁死三幕插画形象，配合微动效图生视频、三段亲切故事配音（asset.voice）、八音盒背景音乐（asset.music）与成片时间线（output.timeline）自动合成。
workflow: storybook-narrate
workflow-version: 1.0.0
argument-hint: "[故事主题/角色设定/适合年龄段] [画幅比例 16:9 或 4:3]"
when-to-use: 用户想要创作儿童有声绘本、睡前故事动画、治愈系插画故事短片或寓言故事视频时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 有声绘本故事 · 角色画风锚定与配音合成 SOP（storybook-narrate）

## 拓扑架构与交付物

```
绘本故事文稿 (play.script)
  ├─out→in-text──→ 主角与绘本画风锚点 (asset.image)
  │                  ├─out→in-image──→ 第一幕·开篇插画 (asset.image) ──out→in-image──→ 第一幕·绘本动态画面 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 第二幕·奇遇插画 (asset.image) ──out→in-image──→ 第二幕·绘本动态画面 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 第三幕·温暖结局插画 (asset.image) ──out→in-image──→ 第三幕·绘本动态画面 (asset.video) ──out→in-clips─┤
  ├─out→in-text──→ 第一幕·故事配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 第二幕·故事配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 第三幕·故事配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 绘本八音盒 BGM (asset.music) ───────────────────────────────────────────────────────────────────────out→in-music─────┤
  └─out→in-subtitles──────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 绘本导演与提示词规范

1. **主角锚点锁死（`heroAnchor`）**：在 `generateInstruction` 中明确绘本媒介材质（如「英国经典水彩手绘绘本风，粗糙水彩纸纹理，柔和暖色铅笔勾线」）、主角外观特征（毛色、服饰、标志性小道具）。三幕插画均通过 `in-image` 继承该锚点，确保小动物/小主角跨页不走形。
2. **三幕跨页插画（`page1Img`~`page3Img`）**：采用绘本跨页构图（Storybook Double-Page Spread），留出呼吸感空间，严禁在画面内直接生成变形文字。
3. **微动效呼吸感（`page1Vid`~`page3Vid`）**：绘本视频宜采用「视差微动效（2.5D Parallax）」：角色眨眼、招手、萤火虫闪烁、树叶轻摇，镜头极其平稳，保持绘本静谧治愈感。
4. **三段纯净旁白配音（`voice1`~`voice3`）**：将第一、二、三幕的故事旁白台词直接写入 `voice1`~`voice3` 的 `params.generateInstruction`（**切记只写要朗读的故事句子本身，不要写提示词说明**）。

## 标准执行步骤（MCP 自动化路径）

1. `workflow_use_installed({ id: "storybook-narrate", name: "<故事名>·有声绘本", generateAspectRatio: "16:9" })`。
2. `graph_read({ assetId, includeParams: true })` 获取节点 ID。
3. `graph_edit` 填入三幕故事字幕稿（`storyScript`）、主角设定（`heroAnchor`）、三幕插画与微动效描述（`page1Img`~`page3Vid`）以及三段故事朗读台词（`voice1`~`voice3`）。
4. `task_run({ assetId })` 一键生成并在 `output.timeline` 合成完整有声绘本短片。

## 端口与参数速查

详见 `references/ports.md`。
