---
name: wf-storyboard-video
description: 用「分镜出片」工作流将分镜脚本转化为视觉连续的三镜电影短片：先生成角色与场景画风主锚点（in-image 锁死全片角色外观与色调），再依次生成远景环境建立、中景动作推进、特写情绪高潮三镜关键帧与图生视频，配合逐镜旁白配音（asset.voice）、电影感配乐（asset.music）在成片时间线（output.timeline）自动合成。
workflow: storyboard-video
workflow-version: 2.0.0
argument-hint: "[分镜故事梗概/三镜画面描述] [画幅比例 16:9 或 9:16]"
when-to-use: 用户有一段短片构思或三幕分镜脚本，想要快速生成角色画风统一、带逐镜配音、BGM 与字幕的完整多镜头短片时使用（若需要 9~36 镜长剧与自动质检闭环请用 wf-short-drama）。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 分镜出片 · 视觉锚点锁死三幕叙事成片 SOP（storyboard-video v2.0）

## 拓扑架构与交付物

```
分镜脚本与字幕 (play.script)
  ├─out→in-text──→ 角色与场景视觉锚点 (asset.image)
  │                  ├─out→in-image──→ 镜头1·远景建立关键帧 (asset.image) ──out→in-image──→ 镜头1·远景视频 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 镜头2·中景推进关键帧 (asset.image) ──out→in-image──→ 镜头2·中景视频 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 镜头3·特写高潮关键帧 (asset.image) ──out→in-image──→ 镜头3·特写视频 (asset.video) ──out→in-clips─┤
  ├─out→in-text──→ 镜头1·旁白配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 镜头2·旁白配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 镜头3·旁白配音 (asset.voice) ──────────────────────────────────────────────────────────────────────out→in-voice─────┤
  ├─out→in-text──→ 电影感环境配乐 (asset.music) ──────────────────────────────────────────────────────────────────────out→in-music─────┤
  └─out→in-subtitles──────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 电影摄影三镜法则（Establishing → Medium Action → Emotional Close-Up）

1. **视觉锚点（`styleAnchor`）**：锁定主角五官穿搭、核心场景光影氛围与胶片色温，通过 `in-image` 注入全部 3 个镜头关键帧，彻底解决多镜短片「每镜换一张脸」的痛点。
2. **镜头 1（远景建立 Establishing Shot）**：交代空间地理环境、时间气候与人物所处位置，平稳推进或横移运镜。
3. **镜头 2（中景动作 Medium Action Shot）**：聚焦主角核心动作与戏剧冲突推进，跟拍（Tracking Shot）保持动态连贯。
4. **镜头 3（特写高潮 Close-Up Shot）**：浅景深面部眼神或关键道具特写，微推镜头捕捉情绪决堤瞬间。
5. **逐镜配音（`voice1`~`voice3`）**：在每个 `asset.voice` 的 `generateInstruction` 中只写该镜对应的纯旁白/独白台词正文。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "storyboard-video", name: "<短片名>·分镜成片", generateAspectRatio: "16:9" })`。
2. `graph_read({ assetId, includeParams: true })` 获取节点 ID。
3. `graph_edit` 覆写 `script`（三句字幕稿）、`styleAnchor`、三组关键帧/视频指令以及 `voice1`~`voice3` 纯朗读台词。
4. `task_run({ assetId })` 一键跑通全图并合成 `output.timeline` 成片。

## 端口与参数速查

详见 `references/ports.md`。
