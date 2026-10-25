---
name: wf-storyboard-video
description: 用「分镜出片」工作流把导演分镜脚本转化为角色与画风高度统一的电影级多镜头短片：导演分镜与旁白脚本（shots）→ 主角与光影定妆锚点（charRef，通过 in-image 锁定全片角色一致性）→ 远景定场/中景推进/特写高潮三镜分镜图（board1~board3）→ 三镜图生视频（clip1~clip3）+ 逐镜旁白配音（voice1~voice3）+ 电影氛围配乐（bgm）→ 分镜成片时间线（timeline: output.timeline）。
workflow: storyboard-video
workflow-version: 2.1.0
argument-hint: "[故事梗概/分镜构想/视觉风格] [画幅比例 16:9 或 9:16]"
when-to-use: 用户想要把一段故事创意、小说片段或导演分镜脚本制作成角色一致、带逐镜配音、配乐与字幕的完整多镜头叙事短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 分镜出片 · 角色锚定与因果叙事多轨成片 SOP（storyboard-video v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，14 节点 / 25 连线）

```
导演分镜与旁白脚本 (text.plain: shots)
  ├─out→in-text──→ 主角与光影定妆锚点 (asset.image: charRef)
  │                  ├─out→in-image──→ 分镜1·远景定场 (asset.image: board1) ────out→in-image──→ 镜头1·定场运镜视频 (asset.video: clip1) ──out→in───────┐
  │                  ├─out→in-image──→ 分镜2·中景叙事推进 (asset.image: board2) ─out→in-image──→ 镜头2·中景跟拍视频 (asset.video: clip2) ──out→in───────┤
  │                  └─out→in-image──→ 分镜3·特写情绪高潮 (asset.image: board3) ─out→in-image──→ 镜头3·高潮特写视频 (asset.video: clip3) ──out→in───────┤
  ├─out→in-text──→ 镜头1旁白配音 (asset.voice: voice1) ───────────────────────────────────────────────────────────────────────────────────out→in-voice─┤
  ├─out→in-text──→ 镜头2旁白配音 (asset.voice: voice2) ───────────────────────────────────────────────────────────────────────────────────out→in-voice─┤
  ├─out→in-text──→ 镜头3高潮旁白 (asset.voice: voice3) ───────────────────────────────────────────────────────────────────────────────────out→in-voice─┤
  ├─out→in-text──→ 电影氛围配乐 (asset.music: bgm) ───────────────────────────────────────────────────────────────────────────────────────out→in-music─┤
  └─out→in-text───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 分镜成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "storyboard-video", mode: "seed" })` 物化后，画布节点 `id` 即为 `shots`、`charRef`、`board1`、`board2`、`board3`、`clip1`、`clip2`、`clip3`、`voice1`、`voice2`、`voice3`、`bgm`、`timeline`、`note`。

## 核心技术机制与提示词铁律

1. **先立因果故事弧线，再拆镜头（Narrative Continuity First）**：
   - 严禁把分镜写成互不相关的风景拼盘。必须围绕同一核心冲突设计「**起（环境与异常触发）→ 承（主角行动与逼近）→ 转/合（高潮对决或真相揭晓）**」的连贯因果链。
2. **视觉锚点锁死（`charRef` → `board1`~`board3`）**：
   - `charRef` 生成包含主角面部特征、标志性穿搭、主色调与光影质感的定妆基准图，通过 `in-image` 喂给所有分镜图，彻底消灭多镜头换脸与画风割裂。
3. **景别递进与首帧运镜（`board1~3` → `clip1~3`）**：
   - 遵循电影视听语言：`board1`（远景定场 Establishing Shot）→ `board2`（中景动作跟拍 Medium Tracking）→ `board3`（面部/关键道具大特写 Extreme Close-Up）；
   - `clip1`~`clip3` 均设置 `generateAudio: false` 静音，避免视频底噪干扰旁白与配乐。
4. **逐镜配音防呆铁律（`voice1`~`voice3`）**：
   - `voice1`~`voice3` 的 `params.generateInstruction` **只写该镜要被 TTS 朗读的纯旁白/独白台词正文**，严禁写「低沉电影旁白音色，朗读镜头1台词」等说明词！

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

`workflow.json` 的 3 镜结构只是默认种子，Agent 应根据故事体量通过 `graph_edit` 自由伸缩：

- **扩展为 4~8 镜完整四幕剧（Scale Up）**：
  当故事需要更丰富的起承转合时，用 `graph_edit` 的 `node_upsert` 新增 `board4`~`boardN`（`asset.image`）、`clip4`~`clipN`（`asset.video`）、`voice4`~`voiceN`（`asset.voice`），将 `charRef:out` 连入 `boardN:in-image`、`boardN:out` 连入 `clipN:in-image`、`clipN:out` 连入 `timeline:in`、`voiceN:out` 连入 `timeline:in-voice`。
- **双角色对手戏锚定**：
  若有正反派两名核心角色，可新增 `charRef2`（反派定妆锚点），在双人同框分镜 `boardN` 上同时连入 `charRef:out` 与 `charRef2:out` 到 `boardN:in-image`（多路参考图口）。
- **单镜快速出片（Prune Down）**：
  若用户只需要 1~2 个镜头，可用 `node_delete` 删掉多余的 `board3` / `clip3` / `voice3` 分支。
- **叠加环境音效与多轨精剪（`timeline_edit`）**：
  可新增 `sfx`（`asset.sfx`，如雨声、脚步声、拔剑声）连入 `timeline:in-voice`，并用 `timeline_edit` 精调镜头转场（`videoTransition: "dissolve" | "smoothleft" | "fadeblack"`）与旁白音色特效。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "storyboard-video", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 将完整分镜与旁白脚本写入 `shots 的 params.`，更新 `charRef`、`board1`~`board3`、`clip1`~`clip3`、`bgm` 的提示词，并将各镜纯朗读台词写入 `voice1`~`voice3` 的 `params.generateInstruction`（按需增删镜头分支）。
3. `task_run({ assetId })` 一键跑通全图并合成 `timeline` 电影短片。

## 端口与参数速查

详见 `references/ports.md`。
