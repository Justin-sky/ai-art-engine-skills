---
name: wf-short-drama9
description: 用「影视精修·单镜头」工作流在结构化分镜与角色场景设定之后，通过「镜头」（film.shot）节点按序号取出指定镜头的关键帧提示词、动态提示词与出场设定图，直接驱动图片生成与视频生成节点做单镜精修。当用户要逐镜精细调参、或复制多路「镜头 → 关键帧 → 镜头视频」并行打磨重点镜头时加载本技能；若要整集一键批量出片请改用 wf-short-drama。
workflow: short-drama9
workflow-version: 1.1.0
---

# 影视精修·单镜头工作台（short-drama9，8 节点 / 10 连线）

> **id 说明**：`short-drama9` 是历史安装身份（对应内置预设 `shortDrama9`），自 v1.1.0 起已升级为基于 `film.shot` 的单镜头精修工作台。

## 这条工作流产出什么

```
剧本 (play.script)
  ├─→ 分镜拆解 (film.breakdown) ──storyboard──→ 导演审核 (film.review)
  └──────────────────────────in-script─────────↗
                                                ├─storyboard→ 角色场景设定 (film.cast) ──filmCast─┐
                                                └─storyboard→ 镜头 1 (film.shot) ←─────in-cast───┘
                                                                ├─out (关键帧提示词)──→ 关键帧 (asset.image) ──out──┐
                                                                ├─out-refs (出场设定图)─↗                           ↓
                                                                └─out-motion (动态提示词)──────────────→ 镜头视频 (asset.video)
```

- 前半段（`play.script` → `film.breakdown` → `film.review` → `film.cast`）与 `wf-short-drama` 完全同构，负责产出经导演审核的结构化分镜与全套角色/场景设定图。
- 后半段不走黑盒批量拍摄，而是接 **`film.shot`（镜头）节点**：
  - 按 `filmShotRef`（镜头序号如 `"1"` 或镜头 id 如 `"S01-02"`）从分镜中抽出单镜数据；
  - 自动把该镜的**关键帧提示词**（`out`）和**该镜出场角色/地点设定图**（`out-refs`）送入 `asset.image`；
  - 自动把关键帧图片与该镜的**动态提示词**（`out-motion`）送入 `asset.video`。

## 什么时候选它（对比 `wf-short-drama`）

- **用 `wf-short-drama9`（单镜精修）**：
  - 用户想对第 N 镜单独换图片/视频模型、单独改画幅或时长、或在关键帧与视频之间插入局部重绘（`image.redraw`）/ 抠图 / 多视角节点；
  - 用户想先跑通第 1 镜看画风与运镜效果，满意后再复制「镜头 → 关键帧 → 镜头视频」分支并行精修多镜。
- **用 `wf-short-drama`（全流程批量拍摄）**：
  - 用户希望一键把整集所有镜头的关键帧与视频批量拍完，并开启关键帧视觉质检自动重拍。

## 怎么操作（MCP 路径）

1. `workflow_use_installed({ id: "short-drama9", name?, generateAspectRatio?, imageModel?, videoModel? })` 落盘为工程资产，得到 `assetId`（预设默认横屏 `16:9`，竖屏短剧可传 `9:16`）。
2. `graph_read({ assetId, includeParams: true })` 获取各节点真实 `nodeId`。
3. `graph_edit` 将剧本写入 `play.script` 的 `params.text`，并可按需设置：
   - `film.shot` 的 `params.filmShotRef`：填镜头序号（从 `"1"` 起）或镜头 id。
   - `asset.image`（关键帧）与 `asset.video`（镜头视频）的模型、画幅与 `generateDuration`。
4. `task_run({ assetId })` → `task_status({ mcpTaskId })` 轮询至完成。
5. **扩展多镜精修**：用 `graph_edit` 新增一组 `film.shot`（设 `filmShotRef: "2"`）+ `asset.image` + `asset.video`，照原样连到 `film.review`（`out` → `in`）与 `film.cast`（`out` → `in-cast`），即可并行精修多镜，且上游分镜与设定图会自动复用、不会重复生成。

## 节点端口与参数速查

详见 `references/pipeline.md`。
