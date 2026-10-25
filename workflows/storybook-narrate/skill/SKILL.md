---
name: wf-storybook-narrate
description: 用「有声绘本故事」工作流把童话、寓言或睡前故事转化为角色画风高度统一的治愈系有声绘本短片：绘本故事文本（script）→ 绘本主角与水彩画风锚点（styleAnchor，in-image 锁死全书角色与笔触）→ 三页跨页绘本插画（page1~page3）+ 逐页温柔故事朗读配音（voice1~voice3）+ 森林星光环境音效（sfx）+ 八音盒摇篮曲伴奏（bgm）→ 有声绘本成片时间线（timeline: output.timeline），并支持动态扩展页数或插入微动效图生视频。
workflow: storybook-narrate
workflow-version: 1.1.0
argument-hint: "[故事主题/角色设定/适合年龄段] [画幅比例 16:9 或 4:3]"
when-to-use: 用户想要创作儿童有声绘本、睡前故事动画、治愈系插画故事短片或寓言故事视频时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 有声绘本故事 · 角色画风锚定与多轨温情合成 SOP（storybook-narrate v1.1）

## 默认种子拓扑与交付物（Seed Skeleton，12 节点 / 21 连线）

```
绘本故事分幕文本 (text.plain: script)
  ├─out→in-text──→ 绘本主角与水彩画风锚点 (asset.image: styleAnchor, 16:9)
  │                  ├─out→in-image──→ 第1页·奇妙开篇插画 (asset.image: page1) ──out→in-image─┐
  │                  ├─out→in-image──→ 第2页·森林奇遇插画 (asset.image: page2) ──out→in-image─┤
  │                  └─out→in-image──→ 第3页·温暖归途插画 (asset.image: page3) ──out→in-image─┤
  ├─out→in-text──→ 第1页·绘本朗读配音 (asset.voice: voice1) ─────────────────────out→in-voice─┤
  ├─out→in-text──→ 第2页·绘本朗读配音 (asset.voice: voice2) ─────────────────────out→in-voice─┤
  ├─out→in-text──→ 第3页·绘本朗读配音 (asset.voice: voice3) ─────────────────────out→in-voice─┤
  ├─out→in-text──→ 夏夜微风与星光环境音 (asset.sfx: sfx) ────────────────────────out→in-voice─┤
  ├─out→in-text──→ 治愈八音盒睡前伴奏 (asset.music: bgm) ────────────────────────out→in-music─┤
  └─out→in-text───────────────────────────────────────────────────────────────────────────────→ 有声绘本成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "storybook-narrate", mode: "seed" })` 物化后，画布节点 `id` 即为 `script`、`styleAnchor`、`page1`、`page2`、`page3`、`voice1`、`voice2`、`voice3`、`sfx`、`bgm`、`timeline`、`note`。

## 绘本导演与提示词规范

1. **主角与媒介肌理锚定（`styleAnchor` · `asset.image`）**：
   - 在 `generateInstruction` 中明确绘本媒介材质（如「凯迪克大奖级英国水彩手绘绘本风，粗糙水彩纸纹理，柔和彩铅勾线」；或结合 `hand-drawn-styles` 的 21 种手绘配方）与小主角外观特征（毛色、耳朵、小围巾等）。`page1`~`page3` 均通过 `in-image` 继承该锚点，确保小主角跨页零走形。
2. **跨页插画与平滑推拉（`page1`~`page3` → `timeline:in-image`）**：
   - 采用绘本跨页构图（Storybook Spread），纯画面无乱码文字；接入 `timeline:in-image` 后，时间线会自动对齐每页朗读配音时长并施加柔和镜头呼吸推拉。
3. **逐页纯净朗读配音（`voice1`~`voice3` · `asset.voice`）**：
   - **铁律**：将每一页的故事朗读台词直接写入 `voice1`~`voice3` 的 `params.generateInstruction`（**只写要被念出来的故事句子本身，严禁写「温柔亲切的睡前故事音色，朗读第1页」等提示词说明**）。

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

- **扩展绘本页数（4~8 页完整童话）**：
  用 `graph_edit` 的 `node_upsert` 新增 `page4`~`pageN`（`asset.image`）与 `voice4`~`voiceN`（`asset.voice`），从 `styleAnchor:out` 连入 `pageN:in-image`，再将 `pageN:out` 连入 `timeline:in-image`、`voiceN:out` 连入 `timeline:in-voice`。
- **升级为「2.5D 微动效动画绘本」（Static Pages → Animated Video Pages）**：
  若用户希望绘本画面中的小狐狸眨眼、萤火虫飞舞、树叶轻摇，可用 `graph_edit` 为每页插画后串联 `page1Vid`~`page3Vid`（`typeId: "asset.video"`, `params: { generateAudio: false, generateInstruction: "绘本 2.5D 视差微动效：主角轻轻眨眼微笑，星光与萤火虫柔和闪烁，镜头平稳静谧" }`），将 `pageN:out → pageNVid:in-image`、`pageNVid:out → timeline:in`。
- **多角色童趣变声（`timeline_edit`）**：
  若故事里有小精灵/大灰狼角色对白，可在 `timeline_edit` 的 `voiceItems` 中设置 `pitchSemitones: 4` / `voiceEffect: "chipmunk"`（可爱小动物音）或 `pitchSemitones: -4`（憨厚大熊音），增加儿童听觉趣味。

## 标准执行步骤（MCP 自动化路径）

1. `workflow_use_installed({ id: "storybook-narrate", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 填入分幕故事文本（`script`）、主角与画风设定（`styleAnchor`）、各页插画描述（`page1`~`page3`）以及各页纯朗读故事句子（`voice1`~`voice3`）。
3. `task_run({ assetId })` 一键生成并在 `timeline` 合成完整有声绘本短片。

## 端口与参数速查

详见 `references/ports.md`。
