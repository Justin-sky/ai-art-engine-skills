---
name: wf-short-drama9
description: 用「影视精修·单镜头」工作流在「影视工作台」（film.workbench）完成结构化分镜与角色场景设定之后，通过「镜头」（film.shot）节点按序号取出指定镜头的关键帧提示词（含 @1/@2 参考图编号）、动态提示词与出场设定图，直接驱动图片生成与视频生成节点做单镜精修。当用户要逐镜精细调参、或复制多路「镜头 → 关键帧 → 镜头视频」并行打磨重点镜头时加载本技能；若要整集一键批量出片请改用 wf-short-drama。
workflow: short-drama9
workflow-version: 1.1.0
---

# 影视精修·单镜头工作台（short-drama9，6 节点 / 7 连线）

> **id 说明**：`short-drama9` 是历史安装身份（对应内置预设 `shortDrama9`），自 v1.1.0 起已升级为基于 `film.workbench` + `film.shot` 的单镜头精修工作台。

## 这条工作流产出什么

```
剧本 (play.script) ──out→in──→ 影视工作台 (film.workbench)
                                 ├─ out (storyboard) ────→ 镜头 1 (film.shot)
                                 └─ out-cast (filmCast) ─→   ├─ out (关键帧提示词) ──→ 关键帧 (asset.image) ──out──┐
                                                             ├─ out-refs (出场设定图) ─↗                           ↓
                                                             └─ out-motion (动态提示词) ─────────────→ 镜头视频 (asset.video)
```

- 前半段由合一的 **`film.workbench`（影视工作台）节点**承载分镜拆解、导演审核与角色/场景设定图生成。
- 后半段接 **`film.shot`（镜头）节点**：
  - 按 `filmShotRef`（镜头序号如 `"1"` 或镜头 id 如 `"S01-02"`）从工作台输出的分镜中抽出单镜数据；
  - 自动把该镜的**关键帧提示词**（`out`，内置 `@1`/`@2` 参考图映射说明）和**该镜出场角色/地点设定图**（`out-refs`）送入 `asset.image`；
  - 自动把关键帧图片与该镜的**动态提示词**（`out-motion`）送入 `asset.video`。

## 什么时候选它（对比 `wf-short-drama`）

- **用 `wf-short-drama9`（单镜精修）**：
  - 用户想对第 N 镜单独换图片/视频模型、单独改画幅或时长、或在关键帧与视频之间插入图像处理（`image.process`：局部重绘 / 抠图 / 高清放大等）或多视角节点；
  - 用户想先跑通第 1 镜看画风与运镜效果，满意后再复制「镜头 → 关键帧 → 镜头视频」分支并行精修多镜。
- **用 `wf-short-drama`（全流程批量拍摄）**：
  - 用户希望在 `film.workbench` 内一键把整集所有镜头的关键帧与视频批量拍完，并开启关键帧视觉质检自动重拍。

## 怎么操作（MCP 路径）

1. `workflow_use_installed({ id: "short-drama9", name?, generateAspectRatio?, textModel?, imageModel?, videoModel? })` 落盘为工程资产，得到 `assetId`（预设默认横屏 `16:9`，竖屏短剧可传 `9:16`；传入的 `textModel` / `imageModel` / `videoModel` 会自动写入 `film.workbench` 与下游 `asset.image` / `asset.video` 节点）。
2. `graph_read({ assetId, includeParams: true })` 获取各节点真实 `nodeId`。
3. `graph_edit` 将剧本写入 `play.script` 的 `params.text`，并可按需设置：
   - `film.workbench` 的 `filmAspectRatio`、`filmTargetShots`、`filmShotDurationSec`、`filmStyle`；
   - `film.shot` 的 `params.filmShotRef`：填镜头序号（从 `"1"` 起）或镜头 id；
   - `asset.image`（关键帧）与 `asset.video`（镜头视频）的模型、画幅与 `generateDuration`。
4. `task_run({ assetId })` → `task_status({ mcpTaskId })` 轮询至完成。
5. **扩展多镜精修**：用 `graph_edit` 新增一组 `film.shot`（设 `filmShotRef: "2"`）+ `asset.image` + `asset.video`，照原样连到 `film.workbench` 的 `out`（接 `film.shot` 的 `in`）与 `out-cast`（接 `film.shot` 的 `in-cast`），即可并行精修多镜（`graph_edit` 会自动将新增的末端节点连入子图输出边界，且上游工作台的分镜与设定图会自动复用、不会重复生成）。

## 节点端口与参数速查

详见 `references/pipeline.md`。
