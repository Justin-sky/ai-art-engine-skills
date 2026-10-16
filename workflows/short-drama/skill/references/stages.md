# 阶段表、端口与参数速查（short-drama）

以下字段名、端口 id 与参数名均对齐应用源码 `src/shared/graph/film/nodes.ts`，可直接用于 `graph_edit`。

## 节点拓扑（6 节点 / 6 连线）

| key | 节点标题 | `typeId` | 职责 |
|---|---|---|---|
| `script` | 剧本 | `play.script` | 存放剧本文本（`params.text`） |
| `breakdown` | 分镜拆解 | `film.breakdown` | 将剧本拆解为结构化分镜文档（场 / 镜头 / 角色场景实体） |
| `review` | 导演审核 | `film.review` | 对照原剧本审查分镜，自动修订不通过镜头（最多 `filmReviewMaxRounds` 轮） |
| `cast` | 角色场景设定 | `film.cast` | 为分镜实体表中的角色 / 地点生成设定图，建立 `filmCast` 设定库 |
| `render` | 批量拍摄 | `film.render` | 按每镜出场名单自动带设定图拍关键帧 → 视觉质检自动重拍 → 以关键帧为首帧生视频 |
| `note` | 流水线说明 | `note.text` | 流程、迭代与精修说明便签 |

### 连线表

| `from` | `fromPort` | `to` | `toPort` | 数据类型 | 说明 |
|---|---|---|---|---|---|
| `script` | `out` | `breakdown` | `in` | `text` | 剧本输入分镜拆解 |
| `script` | `out` | `review` | `in-script` | `text` | 原剧本输入导演审核作对照基准 |
| `breakdown` | `out` | `review` | `in` | `storyboard` | 初始结构化分镜送审 |
| `review` | `out` | `cast` | `in` | `storyboard` | 审核通过/修订后的分镜提取角色与场景设定 |
| `review` | `out` | `render` | `in` | `storyboard` | 审核通过/修订后的分镜送入批量拍摄 |
| `cast` | `out` | `render` | `in-cast` | `filmCast` | 角色与地点设定图库注入批量拍摄锁一致性 |

---

## 各节点端口与关键参数

### 1. `film.breakdown`（分镜拆解）

- **端口**：
  - `in`（`text`）：上游剧本
  - `out`（`storyboard`）：结构化分镜文档
  - `out-text`（`text`）：可读的 Markdown 镜头表
- **关键参数**：
  - `filmAspectRatio`：画幅（预设为 `"9:16"`，可改 `"16:9"` 等）
  - `filmTargetShots`：目标镜头数（`0` = 随剧情自动决定）
  - `filmStyle`：全局视觉风格（如 `"写实电影感、冷暖对比、35mm 胶片颗粒"`）
  - `generateInstruction`：补充导演拆解要求
  - `filmForce`：`true` 时忽略「剧本未变化」缓存强制重拆

### 2. `film.review`（导演审核）

- **端口**：
  - `in`（`storyboard`）：上游分镜文档
  - `in-script`（`text`）：原剧本对照
  - `out`（`storyboard`）：审核修订后的分镜文档
  - `out-text`（`text`）：审核报告
- **关键参数**：
  - `filmReviewMaxRounds`：最多自动修订轮数（预设 `2`；`0` = 只审不改）
  - `generateInstruction`：额外审核标准

### 3. `film.cast`（角色场景设定）

- **端口**：
  - `in`（`storyboard`）：分镜文档
  - `in-image`（`image`，可多连）：现成参考图（标题与实体同名则直接作为该实体设定图，其余作为全局画风参考）
  - `out`（`filmCast`）：角色场景设定库
  - `out-sheets`（`images`）：生成的全部设定图
- **关键参数**：
  - `filmCastKinds`：要生成设定图的实体类别，默认 `"character,location"`（可加 `"prop"`）
  - `generateInstruction`：设定图补充要求
  - `filmForce`：`true` 时忽略描述指纹缓存全部重画

### 4. `film.render`（批量拍摄）

- **端口**：
  - `in`（`storyboard`）：分镜文档
  - `in-cast`（`filmCast`）：设定库（按每镜 `characters` / `location` 自动注入参考图）
  - `out`（`storyboard`）：回填了每镜产物路径的分镜文档
  - `out-images`（`images`）：全部镜头的关键帧图片组
  - `out-videos`（`videos`）：全部镜头的成片视频组
- **关键参数**：
  - `filmRenderMode`：`"keyframe"`（只拍关键帧）或 `"video"`（关键帧 + 图生视频，预设默认 `"video"`）
  - `filmShotRange`：镜头范围过滤（空串 = 全部，支持 `"1-5,8"`）
  - `filmRenderQc`：是否开启关键帧视觉质检（预设 `true`）
  - `filmRenderMaxRetries`：质检不通过自动重拍次数（预设 `1`）
  - `filmForce`：`true` 时忽略提示词指纹缓存全部重拍
  - `generateModel` / `filmVideoModel` / `filmQcModel`：分别覆盖图片、视频、视觉质检模型

---

## 结构化分镜 JSON（`params.filmStoryboard`）

schema 固定为 `aiart.film-storyboard@1`，主要结构：

- `entities[]`：`{ id, kind: "character" | "location" | "prop", name, description, locked? }`
  - 外观描述统一收敛在 `entities[].description`，镜头提示词里直接写实体名称。
- `shots[]`：`{ id, sceneId, shotSize, angle, cameraMove, durationSec, characters, location, props, action, dialogue, imagePrompt, videoPrompt, locked?, status?, review?, media? }`
  - 将某镜的 `locked` 设为 `true` 后，上游重新拆解或导演审核都不会覆盖该镜。
