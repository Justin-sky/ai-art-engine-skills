---
name: wf-short-drama
description: 用「影视全流程」工作流通过合一的「影视工作台」（film.workbench）节点把一段剧本或故事梗概自动拆成结构化分镜（镜头数随剧情）、经导演审核自动修订不通过镜头、按出场角色与场景生成设定图，再批量拍摄关键帧（含同场次连续性锁画风与视觉质检自动重拍）与镜头视频。当用户要做一集短剧/漫剧/影视成片、需要角色场景一致性与增量闭环重拍时加载本技能；若只需单镜手动精修请改用 wf-short-drama9。
workflow: short-drama
workflow-version: 1.1.0
---

# 影视全流程 · 影视工作台闭环（short-drama，3 节点 / 1 连线）

> **id 说明**：`short-drama` 是历史安装身份（对应内置预设 `shortDrama`），目前已全面升级为四合一的 `film.workbench`（影视工作台）节点，在单个大屏工作台内完成「①分镜拆解 → ②导演审核 → ③角色场景设定 → ④批量拍摄」全链路闭环。

## 这条工作流产出什么

```
剧本 (play.script) ──out→in──→ 影视工作台 (film.workbench) ←──in-cast── 全剧圣经/上集设定库 (可选)
                                 ├─ out (storyboard：结构化分镜文档)
                                 ├─ out-cast (filmCast：角色与场景设定库，可跨集连入下一集 in-cast)
                                 ├─ out-sheets (images：角色/地点设定图)
                                 ├─ out-images (images：各镜关键帧图片组)
                                 ├─ out-videos (videos：各镜成片视频组)
                                 └─ out-text (text：可读分镜表 / 审核报告)
```

1. **① 分镜拆解（`storyboard`）**：将剧本拆解为含场次、镜头列表（镜头数随剧情自动决定，支持预设单镜时长 `4s/8s/10s/15s`）与角色/场景/道具实体表的结构化分镜。
2. **② 导演审核与自动修订**：对照原剧本逐镜审查，不通过的镜头自动重写并复审（默认最多 2 轮），锁定镜头永不改写。
3. **③ 角色与场景设定图（`out-cast` + `out-sheets`）**：为分镜实体表中的每个角色与地点生成设定图，支持单角色/单场景独立生成或重绘，描述不变自动复用缓存。
4. **④ 批量关键帧与镜头视频（`out-images` / `out-videos`）**：按每镜出场名单自动以 `@1`/`@2` 编号注入角色/地点设定图生成关键帧（开启同场次连续性时自动注入上一镜关键帧）→ 视觉质检不合格自动重拍 → 以关键帧为首帧图生视频（自动将本镜时长透传给视频模型，支持单镜独立拍摄/补拍视频）。

## 核心特性

- **合一工作台，零同步断层**：分镜拆解、导演审核、角色场景设定与批量拍摄共享同一个 `film.workbench` 节点的状态，双击节点进入全屏工作台即可在 4 个阶段页签间无缝切换，既可分阶段单独运行，也可点「⚡ 一键全流程」串联跑完。
- **百集短剧全剧圣经（Series Bible）跨集共享与换装继承**：将全剧总设定工作台或上一集 `film.workbench` 的 `out-cast` 连入本集 `in-cast` 端口后，分镜拆解自动注入常驻角色/场景标准名，设定阶段直接复用已有设定图不重画；若角色换装/换造型（命名为 `角色名#造型` 或 `角色名·造型`，如 `林夏#晚礼服`），设定阶段自动将主角色 `林夏` 设定图作为 `@1` 参考图传入，锁死五官骨相仅切换服饰。
- **场次连续性链并发拍摄（`filmRenderConcurrency`）**：按场次连续性链并发生成关键帧（默认 3 路场次并发，同一场次内严格顺序承接上一镜关键帧），视频生成进入独立并发池后台运行，大幅提速长剧批量出片。
- **可视化绑定与 `@n` 编号对齐**：镜头卡片直接展示绑定角色与场景设定图缩略图及 `@1`/`@2` 参考图编号，与发往生图模型的 `referenceImageUrls` 顺序严格一致。
- **增量指纹缓存 + 镜头/设定锁定 + 单卡独立运行**：修改某一镜提示词或某一个角色描述后，可直接点击该卡片上的「▶ 生成 / ↻ 重跑」单独重跑该项，也可整阶段增量运行（未改动项秒级跳过）。

## 怎么操作（MCP 路径）

1. `workflow_list_installed` 确认 id；`workflow_use_installed({ id: "short-drama", name?, generateAspectRatio?, textModel?, imageModel?, videoModel? })` 落盘为工程资产，得到 `assetId`（竖屏短剧默认 `9:16`，横屏影视可传 `16:9`；传入的 `textModel` / `imageModel` / `videoModel` 会自动写入 `film.workbench` 的拆解审核文本模型、设定图与关键帧生图模型、图生视频模型）。
2. `graph_read({ assetId, includeParams: true })` 读取节点列表，按标题或 `typeId` 定位：
   - `play.script`（标题「剧本」）
   - `film.workbench`（标题「影视工作台」）
3. `graph_edit` 写入剧本与拍摄参数：
   - 必填：把 `play.script` 的 `params.text` 替换为完整剧本。
   - 按需设置 `film.workbench` 的 `filmAspectRatio`（`"9:16"` / `"16:9"`）、`filmTargetShots`（`0` = 自动，或指定镜头数）、`filmShotDurationSec`（`0` = 随剧情自动，或 `4` / `8` / `10` / `15` 秒对齐视频模型时长档位）、`filmStyle`（视觉风格）、`filmRenderConcurrency`（`1`~`8`，默认 `3`）。
   - **运行范围控制（`filmWorkbenchRunMode`）**：
     - 留空或设为 `"all"`：一次运行串联跑完「拆解 → 审核 → 设定 → 拍摄」全流程；
     - 设为 `"breakdown"` / `"review"` / `"cast"` / `"render"`：只运行指定阶段（适合先拆解审核确认分镜、再生成设定图、最后批量拍摄）。
   - **先看关键帧还是直出视频**：`film.workbench` 在本预设中默认 `filmRenderMode: "video"`（拍关键帧 + 图生视频）。若用户想先确认分镜画面再出视频，先把 `filmRenderMode` 设为 `"keyframe"`，跑完确认后再改回 `"video"` 增量补拍视频（关键帧会直接命中缓存，不会重画）。
4. `task_run({ assetId })` → `mcpTaskId`；用 `task_status({ mcpTaskId })` 轮询至完成，汇报设定图、关键帧与各镜视频路径。

## 闭环迭代与局部返工

- **改某一镜并只重拍这一镜**：
  1. `graph_read({ assetId, includeParams: true })` 读取 `film.workbench` 节点的 `params.filmStoryboard`（JSON 字符串，schema 为 `aiart.film-storyboard@1`）。
  2. 修改目标镜头的 `imagePrompt` / `videoPrompt` / `action` / `dialogue` / `characters` / `location`，并将该镜头的 `locked` 设为 `true`。
  3. 用 `graph_edit` 写回 `filmStoryboard`，并可设置 `filmWorkbenchRunMode: "render"` 与 `filmRenderOnlyShotId: "<镜头id>"`（或留空跑增量），再执行运行 —— 指纹未变的镜头全部秒级跳过，只重拍目标镜头。
- **只重画某一个角色/场景设定图**：设置 `film.workbench` 的 `filmWorkbenchRunMode: "cast"` 与 `filmCastOnlyId: "<实体id>"` 后运行。
- **只拍指定镜头区间**：设置 `film.workbench` 的 `params.filmShotRange`（如 `"1-4,7"`），适合长剧本先试拍前几镜。
- **单镜深度精修**：若某一镜需要手调特定模型或接额外精修节点，可从 `film.workbench` 的 `out` 与 `out-cast` 引出一个 `film.shot`（镜头）节点，接 `asset.image` / `asset.video` 单独打磨（或直接使用 `wf-short-drama9` 工作流）。

## 参数与端口速查

详细的节点端口、参数名与 `aiart.film-storyboard@1` 数据结构见 `references/stages.md`。
