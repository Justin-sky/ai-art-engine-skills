# 节点、端口与参数速查（short-drama9）

以下字段名、端口 id 与参数名均对齐应用源码 `src/shared/graph/film/nodes.ts`，可直接用于 `graph_edit`。

## 节点表（6 节点 / 7 连线）

| key | 节点标题 | `typeId` | 关键参数 |
|---|---|---|---|
| `script` | 剧本 | `play.script` | `text`（剧本全文） |
| `workbench` | 影视工作台 | `film.workbench` | `filmAspectRatio: "16:9"`, `filmTargetShots: 0`, `filmShotDurationSec: 0`, `filmReviewMaxRounds: 1`, `filmCastKinds: "character,location"` |
| `shot` | 镜头 1 | `film.shot` | `filmShotRef: "1"`（1 起序号或镜头 id，如 `"S01-02"`） |
| `keyframe` | 关键帧 | `asset.image` | `generateAspectRatio: "16:9"`, `generateModel` |
| `clip` | 镜头视频 | `asset.video` | `generateAspectRatio: "16:9"`, `generateDuration: 5`, `generateModel` |
| `note` | 使用说明 | `note.text` | `text`（只读说明） |

## 连线表（7 条边）

| `from` | `fromPort` | `to` | `toPort` | 数据类型 | 说明 |
|---|---|---|---|---|---|
| `script` | `out` | `workbench` | `in` | `text` | 剧本文本送入影视工作台（完成拆解、审核与角色场景设定） |
| `workbench` | `out` | `shot` | `in` | `storyboard` | 结构化分镜送入单镜头提取节点 |
| `workbench` | `out-cast` | `shot` | `in-cast` | `filmCast` | 角色与场景设定库送入单镜头提取节点 |
| `shot` | `out` | `keyframe` | `in-text` | `text` | 该镜的关键帧画面提示词（自动附带 `@1`/`@2` 设定图对照说明） |
| `shot` | `out-refs` | `keyframe` | `in-image` | `image` | 该镜出场角色与地点的设定参考图（自动按出场名单过滤并对齐 `@n` 顺序） |
| `keyframe` | `out` | `clip` | `in-image` | `image` | 关键帧图片作为视频生成的画面基准 |
| `shot` | `out-motion` | `clip` | `in-text` | `text` | 该镜的动态运镜与表演提示词 |

## `film.shot`（镜头节点）详解

- **输入端口**：
  - `in`（`storyboard`）：来自 `film.workbench`（或旧版 `film.breakdown` / `film.review`）的结构化分镜。
  - `in-cast`（`filmCast`）：来自 `film.workbench` 的 `out-cast`（或旧版 `film.cast` 的 `out`）的角色/地点设定库。
- **输出端口**：
  - `out`（`text`）：当前选中镜头的关键帧生成提示词（自动融合实体外观基准与 `@1`/`@2` 参考图索引）。
  - `out-motion`（`text`）：当前选中镜头的视频动态提示词（含运镜、动作与台词）。
  - `out-refs`（`image`）：当前选中镜头出场角色与地点的设定图集合，可直接连入 `asset.image` 或 `asset.video` 的 `in-image` 端口。
- **如何并行精修多镜**：
  - 不需要重跑 `workbench`；只需复制 `shot` → `keyframe` → `clip` 三节点子图，把新 `film.shot` 的 `filmShotRef` 设为 `"2"`、`"3"`……并同样从 `workbench:out` 与 `workbench:out-cast` 接线即可。
