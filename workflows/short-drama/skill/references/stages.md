# 阶段表、端口与参数速查（short-drama）

以下字段名、端口 id 与参数名均对齐应用源码 `src/shared/graph/film/nodes.ts`，可直接用于 `graph_edit`。

## 节点拓扑（3 节点 / 1 连线）

| key | 节点标题 | `typeId` | 职责 |
|---|---|---|---|
| `script` | 剧本 | `play.script` | 存放剧本文本（`params.text`） |
| `workbench` | 影视工作台 | `film.workbench` | 合一承载「①分镜拆解 → ②导演审核 → ③角色场景设定 → ④批量拍摄」全流程，既可按阶段单独运行，也可一键全流程闭环 |
| `note` | 流水线说明 | `note.text` | 流程、迭代与精修说明便签 |

### 连线表

| `from` | `fromPort` | `to` | `toPort` | 数据类型 | 说明 |
|---|---|---|---|---|---|
| `script` | `out` | `workbench` | `in` | `text` | 剧本输入影视工作台 |

---

## `film.workbench`（影视工作台）端口与关键参数

### 端口定义

- **输入端口**：
  - `in`（`text`）：上游剧本文本
  - `in-cast`（`filmCast`，可多连）：全剧/上一集共享设定库（Series Bible）。接入后：① 分镜拆解自动将常驻角色/场景标准名称注入提示词并继承外观描述；③ 角色场景设定直接复用已有设定图不重画，仅生成新出场实体；若角色名写 `角色名#造型`（如 `林夏#晚礼服`），会自动将主角色设定图作为 `@1` 参考图生成换装设定图
  - `in-image`（`image`，可多连）：现成参考图（标题与实体同名则直接作为该实体设定图，其余作为全局画风参考）
- **输出端口**：
  - `out`（`storyboard`）：结构化分镜文档（含已回填的每镜关键帧/视频路径）
  - `out-cast`（`filmCast`）：角色与场景设定库（可直接连给下游单镜头节点 `film.shot` 的 `in-cast`，或连给下一集 `film.workbench` 的 `in-cast` 跨集共享）
  - `out-sheets`（`images`）：生成的全部角色/场景设定图
  - `out-images`（`images`）：全部镜头的关键帧图片组
  - `out-videos`（`videos`）：全部镜头的成片视频组
  - `out-text`（`text`）：可读的 Markdown 分镜表与审核报告

### 阶段与运行控制参数

- `filmWorkbenchStage`：当前工作台激活页签（`"breakdown"` | `"review"` | `"cast"` | `"render"`）
- `filmWorkbenchRunMode`：运行模式
  - `""` 或 `"all"`：一键运行全流程（按需依次执行分镜拆解 → 导演审核 → 角色场景设定 → 批量拍摄）
  - `"breakdown"`：仅执行 ① 分镜拆解
  - `"review"`：仅执行 ② 导演审核
  - `"cast"`：仅执行 ③ 角色场景设定
  - `"render"`：仅执行 ④ 批量拍摄
- `filmCastOnlyId`：在设定阶段仅生成/重跑指定实体 id（如 `"E01"`），跑完自动清空
- `filmRenderOnlyShotId`：在拍摄阶段仅生成/重跑指定镜头 id（如 `"S01-01"`），跑完自动清空

### ① 分镜拆解参数

- `filmAspectRatio`：画幅（本预设为 `"9:16"`，可改 `"16:9"`、`"2.39:1"` 等）
- `filmTargetShots`：目标镜头数（`0` = 随剧情自动决定）
- `filmShotDurationSec`：默认单镜时长秒数（`0` = 随剧情自动分配，或设 `4` / `8` / `10` / `15` 对齐视频模型时长档位）
- `filmStyle`：全局视觉风格（如 `"写实电影感、冷暖对比、35mm 胶片颗粒"`）
- `filmBreakdownInstruction`：拆解阶段专属导演要求
- `filmForce`：`true` 时忽略缓存强制重跑

### ② 导演审核参数

- `filmReviewMaxRounds`：最多自动修订轮数（预设 `2`；`0` = 只审不改）
- `filmReviewInstruction`：审核阶段专属标准
- `filmReviewVerdict` / `filmReviewReport`：最近一次审核结论（`"PASS"` / `"FAIL"`）与报告文本

### ③ 角色场景设定参数

- `filmCastKinds`：要生成设定图的实体类别，默认 `"character,location"`（可加 `"prop"`）
- `filmCastInstruction`：设定阶段专属补充要求
- `filmCast`：角色场景设定库 JSON（schema 为 `aiart.film-cast@1`）

### ④ 批量拍摄与模型参数

- `filmRenderMode`：`"keyframe"`（只拍关键帧）或 `"video"`（关键帧 + 图生视频，本预设默认 `"video"`）
- `filmShotRange`：镜头范围过滤（空串 = 全部，支持 `"1-5,8"`）
- `filmRenderConcurrency`：最大并发场次数（`1`~`8`，默认 `3`；按场次连续性链并发拍关键帧，同一场内顺序承接上一镜关键帧，视频在独立并发池后台生成）
- `filmRenderContinuity`：同场次相邻镜头连续性参考（默认 `true`，自动将同场上一镜已生成的关键帧作为额外参考图注入）
- `filmRenderQc`：是否开启关键帧视觉质检（本预设 `true`）
- `filmRenderMaxRetries`：质检不通过自动重拍次数（预设 `1`）
- `generateInstruction`：批量拍摄阶段画面补充要求
- `filmTextModel` / `filmTextProviderInstanceId`：拆解与审核使用的文本模型
- `generateModel` / `generateProviderInstanceId`：设定图与关键帧使用的生图模型
- `filmVideoModel` / `filmVideoProviderInstanceId`：图生视频模型
- `filmQcModel` / `filmQcProviderInstanceId`：视觉质检多模态模型

---

## 结构化分镜 JSON（`params.filmStoryboard`）

schema 固定为 `aiart.film-storyboard@1`，主要结构：

- `entities[]`：`{ id, kind: "character" | "location" | "prop", name, description, locked? }`
  - 外观描述统一收敛在 `entities[].description`，镜头提示词里直接写实体名称，拍摄时按出场顺序自动绑定 `@1`、`@2` 参考图。
- `shots[]`：`{ id, sceneId, shotSize, angle, cameraMove, durationSec, characters, location, props, action, dialogue, imagePrompt, videoPrompt, locked?, status?, review?, media? }`
  - 将某镜的 `locked` 设为 `true` 后，重新拆解或导演审核都不会覆盖该镜。
