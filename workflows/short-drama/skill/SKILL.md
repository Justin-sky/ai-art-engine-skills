---
name: wf-short-drama
description: 用「影视全流程」工作流把一段剧本或故事梗概自动拆成结构化分镜（镜头数随剧情）、经导演审核自动修订不通过镜头、按出场角色与场景生成设定图，再批量拍摄关键帧（含视觉质检自动重拍）与镜头视频。当用户要做一集短剧/漫剧/影视成片、需要角色场景一致性与增量闭环重拍时加载本技能；若只需单镜手动精修请改用 wf-short-drama9。
workflow: short-drama
workflow-version: 1.1.0
---

# 影视全流程 · Agent 闭环（short-drama，6 节点 / 6 连线）

> **id 说明**：`short-drama` 是历史安装身份（对应内置预设 `shortDrama`），自 v1.1.0 起已全面升级为 `film.*` 结构化影视节点家族，彻底取代旧版 137 节点 9×4 宫格手搭链路。

## 这条工作流产出什么

```
剧本 (play.script)
  ├─→ 分镜拆解 (film.breakdown) ──storyboard──→ 导演审核 (film.review)
  └──────────────────────────in-script─────────↗
                                                ├─storyboard→ 角色场景设定 (film.cast) ──filmCast─┐
                                                └─storyboard→ 批量拍摄 (film.render) ←──in-cast───┘
```

1. **结构化分镜文档（`storyboard`）**：含场次、镜头列表（镜头数随剧情自动决定，不再硬编码 9 格或 36 格）与角色/场景/道具实体表。
2. **导演审核与自动修订**：对照原剧本逐镜审查，不通过的镜头自动重写并复审（默认最多 2 轮），锁定镜头永不改写。
3. **角色与场景设定图（`filmCast` + `out-sheets`）**：为分镜实体表中的每个角色与地点生成设定图，描述不变自动复用缓存。
4. **批量关键帧与镜头视频（`out-images` / `out-videos`）**：按每镜出场名单自动注入对应角色/地点设定图生成关键帧 → 视觉质检不合格自动重拍 → 以关键帧为首帧图生视频。

## 为什么不再用 9 宫格 / 36 格流水线

- **镜头数随剧情**：短剧本可拆 5~8 镜，长剧本可拆 15~30+ 镜，不再为了凑满 9/36 格而注水或截断结局。
- **结构化端口防接错**：`storyboard` 与 `filmCast` 均为专用强类型端口，同时保留 `out-text` / `out-sheets` / `out-images` / `out-videos` 供下游预览或接其它节点。
- **增量指纹缓存 + 镜头锁定**：改一镜的提示词再运行，`film.render` 只重拍被修改的那一镜，其余已拍好的镜头直接复用；在检查器或 JSON 中把满意镜头设为 `locked: true`，重新拆解或导演复审都不会覆盖它。

## 怎么操作（MCP 路径）

1. `workflow_list_installed` 确认 id；`workflow_use_installed({ id: "short-drama", name?, generateAspectRatio?, imageModel?, videoModel? })` 落盘为工程资产，得到 `assetId`（竖屏短剧默认 `9:16`，横屏影视可传 `16:9`）。
2. `graph_read({ assetId, includeParams: true })` 读取节点列表，按标题或 `typeId` 定位：
   - `play.script`（标题「剧本」）
   - `film.breakdown`（标题「分镜拆解」）
   - `film.review`（标题「导演审核」）
   - `film.cast`（标题「角色场景设定」）
   - `film.render`（标题「批量拍摄」）
3. `graph_edit` 写入剧本与拍摄参数：
   - 必填：把 `play.script` 的 `params.text` 替换为完整剧本。
   - 按需设置 `film.breakdown` 的 `filmAspectRatio`（`"9:16"` / `"16:9"`）、`filmTargetShots`（`0` = 自动，或指定镜头数）、`filmStyle`（视觉风格）。
   - **先看关键帧还是直出视频**：`film.render` 默认 `filmRenderMode: "video"`（拍关键帧 + 图生视频）。若用户想先确认分镜画面再出视频，先把 `filmRenderMode` 设为 `"keyframe"`，跑完确认后再改回 `"video"` 增量补拍视频（关键帧会直接命中缓存，不会重画）。
4. `task_run({ assetId })` → `mcpTaskId`；用 `task_status({ mcpTaskId })` 轮询至完成，汇报设定图、关键帧与各镜视频路径。

## 闭环迭代与局部返工

- **改某一镜并只重拍这一镜**：
  1. `graph_read({ assetId, includeParams: true })` 读取 `film.review`（或 `film.breakdown`）节点的 `params.filmStoryboard`（JSON 字符串，schema 为 `aiart.film-storyboard@1`）。
  2. 修改目标镜头的 `imagePrompt` / `videoPrompt` / `action` / `dialogue`，并将该镜头的 `locked` 设为 `true`。
  3. 用 `graph_edit` 写回 `filmStoryboard`，再对 `film.render` 执行运行 —— 指纹未变的镜头全部秒级跳过，只重拍修改过的镜头。
- **只拍指定镜头区间**：设置 `film.render` 的 `params.filmShotRange`（如 `"1-4,7"`），适合长剧本先试拍前几镜。
- **单镜深度精修**：若某一镜需要手调特定模型或接额外精修节点，可从 `film.review` 和 `film.cast` 引出一个 `film.shot`（镜头）节点，接 `asset.image` / `asset.video` 单独打磨（或直接使用 `wf-short-drama9` 工作流）。

## 参数与端口速查

详细的节点端口、参数名与 `aiart.film-storyboard@1` 数据结构见 `references/stages.md`。
