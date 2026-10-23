# 阶段表、端口与参数速查（short-drama v2.0.0）

以下字段名、端口 id 与参数名均对齐应用源码 `src/shared/graph/`，可直接用于 `graph_edit`。

## 节点拓扑（9 节点 / 9 连线）

| key | 节点标题 | `typeId` | 阶段与职责 |
|---|---|---|---|
| `outline` | 故事大纲与世界观 | `text.plain` | 存放全剧故事梗概、核心冲突与世界观设定（`params.text`） |
| `screenplay` | 创建剧本 · 剧本工作台 | `asset.screenplay` | **阶段一：创建剧本**。根据上游大纲生成包含 `【人物小传】` 与 `第1集：...`、`第2集：...` 的标准多集剧本，自动按集切分并从 `out-all` 输出分集数组（双击可进入剧本工作台及 3D 剧本运镜预演） |
| `ep1_script` | 剧本分集 · 第 1 集 | `text.select` | **阶段二：剧本分集**。`selectedTextId: "ep:1"`，从 `screenplay` 的 `out-all` 提取第 1 集剧本（含共享【人物小传】） |
| `ep2_script` | 剧本分集 · 第 2 集 | `text.select` | **阶段二：剧本分集**。`selectedTextId: "ep:2"`，从 `screenplay` 的 `out-all` 提取第 2 集剧本（含共享【人物小传】） |
| `ep1_workbench` | 第 1 集 · 短剧工作台 | `film.workbench` | **阶段三：分集短剧工作台（第 1 集）**。合一承载第 1 集的「①分镜拆解 → ②导演审核 → ③角色场景设定 → ④批量拍摄」，并将产出的设定库 `out-cast` 传给第 2 集 |
| `ep2_workbench` | 第 2 集 · 短剧工作台 | `film.workbench` | **阶段三：分集短剧工作台（第 2 集）**。通过 `in-cast` 接收第 1 集设定库（Series Bible），自动复用主角与常驻场景设定图，完成第 2 集拆解、审核、新增设定与批量拍摄 |
| `bgm` | 全剧悬疑氛围配乐 | `asset.music` | 生成全剧背景纯音乐配乐（`generateMusicInstrumental: true`），输出到成片时间线 |
| `timeline` | 短剧成片时间线 | `output.timeline` | **阶段四：成片时间线**。按集序汇聚第 1 集与第 2 集短剧工作台产出的镜头视频流与全剧配乐，自动添加转场、混音并导出最终 MP4 成片 |
| `note` | 影视全流程说明 | `note.text` | 四阶段全流程操作说明便签 |

### 连线表

| `from` | `fromPort` | `to` | `toPort` | 数据类型 | 说明 |
|---|---|---|---|---|---|
| `outline` | `out` | `screenplay` | `in` | `text` | 故事大纲输入剧本工作台 |
| `screenplay` | `out-all` | `ep1_script` | `in` | `texts` | 分集剧本数组输入第 1 集选择器 |
| `screenplay` | `out-all` | `ep2_script` | `in` | `texts` | 分集剧本数组输入第 2 集选择器 |
| `ep1_script` | `out` | `ep1_workbench` | `in` | `text` | 第 1 集剧本输入第 1 集短剧工作台 |
| `ep2_script` | `out` | `ep2_workbench` | `in` | `text` | 第 2 集剧本输入第 2 集短剧工作台 |
| `ep1_workbench` | `out-cast` | `ep2_workbench` | `in-cast` | `filmCast` | 跨集角色/场景设定库（Series Bible）串联共享 |
| `ep1_workbench` | `out-videos` | `timeline` | `in-videos` | `videos` | 第 1 集镜头视频序列接入成片时间线 |
| `ep2_workbench` | `out-videos` | `timeline` | `in-videos` | `videos` | 第 2 集镜头视频序列接入成片时间线 |
| `bgm` | `out` | `timeline` | `in-music` | `voice` | 全剧配乐接入成片时间线音乐轨 |

---

## 阶段一：`asset.screenplay`（创建剧本 · 剧本工作台）

### 端口定义
- **输入端口**：
  - `in`（`text`）：上游故事大纲、世界观或分集梗概
- **输出端口**：
  - `out`（`text`）：当前选中的单篇剧本文本
  - `out-all`（`texts`）：当生成的剧本正文包含 `>= 2` 个显式分集标题（如 `第1集：...`、`第2集：...` 或 `EPISODE 1: ...`）时，自动按集切分为多条剧本文本输出，且每一集自动附带顶部的 `【人物小传】` 设定区块

### 关键参数
- `generateInstruction`：编剧生成指令（可指定集数、题材、情绪节奏、每集场次数）
- `systemPrompt`：可选自定义编剧系统提示词（留空使用内置标准影视/短剧编剧规范）
- `text`：当前剧本全文内容（可直接预填或由模型生成后落盘写回）
- `generateModel` / `generateProviderInstanceId`：编剧使用的文本大模型

---

## 阶段二：`text.select`（剧本分集）

### 端口定义
- **输入端口**：
  - `in`（`texts`）：接收上游 `asset.screenplay` 的 `out-all` 分集剧本数组（若上游仅传入 1 条含多集标记或多场戏的文本，节点内部也会自动展开分集）
- **输出端口**：
  - `out`（`text`）：选中的那一集完整剧本文本（含人物小传与本集所有场次）

### 关键参数
- `selectedTextId`：分集选择器，推荐使用符号化集号 `"ep:1"`、`"ep:2"`、`"ep:3"`（1-based 集号）。使用 `"ep:N"` 时，即使上游 `asset.screenplay` 重新生成了新时间戳的分集条目，选择器也会自动匹配最新一轮的第 N 集并保持 `"ep:N"` 配置不变。

---

## 阶段三：`film.workbench`（分集短剧工作台）

### 端口定义
- **输入端口**：
  - `in`（`text`）：本集剧本文本（来自对应集的 `text.select`）
  - `in-cast`（`filmCast`，可多连）：全剧/上一集共享设定库（Series Bible）。接入后：① 分镜拆解自动将常驻角色/场景标准名称注入提示词并继承外观描述；③ 角色场景设定直接复用已有设定图不重画，仅生成本集新出场实体；若角色名写 `角色名#造型`（如 `林夏#晚礼服`），会自动将主角色设定图作为 `@1` 参考图生成换装设定图
  - `in-image`（`image`，可多连）：现成参考图（标题与实体同名则直接作为该实体设定图，其余作为全局画风参考）
- **输出端口**：
  - `out`（`storyboard`）：结构化分镜文档（含已回填的每镜关键帧/视频路径）
  - `out-cast`（`filmCast`）：本集角色与场景设定库（包含合并继承自上游 `in-cast` 的设定，可继续连给下一集 `film.workbench` 的 `in-cast` 或单镜头节点 `film.shot` 的 `in-cast`）
  - `out-sheets`（`images`）：全部角色/场景设定图
  - `out-images`（`images`）：本集全部镜头的关键帧图片序列
  - `out-videos`（`videos`）：本集全部镜头的成片视频序列（连入 `output.timeline` 的 `in-videos`）
  - `out-text`（`text`）：可读的 Markdown 分镜表与审核报告

### 阶段与运行控制参数
- `filmWorkbenchStage`：当前工作台激活页签（`"breakdown"` | `"review"` | `"cast"` | `"render"`）
- `filmWorkbenchRunMode`：运行模式
  - `""` 或 `"all"`：一键运行全流程（依次执行 ①分镜拆解 → ②导演审核 → ③角色场景设定 → ④批量拍摄）
  - `"breakdown"`：仅执行 ① 分镜拆解
  - `"review"`：仅执行 ② 导演审核
  - `"cast"`：仅执行 ③ 角色场景设定
  - `"render"`：仅执行 ④ 批量拍摄
- `filmCastOnlyId`：在设定阶段仅生成/重跑指定实体 id（如 `"E01"`），跑完自动清空
- `filmRenderOnlyShotId`：在拍摄阶段仅生成/重跑指定镜头 id（如 `"S01-01"`），跑完自动清空

### 分镜、审核、设定与拍摄参数
- `filmAspectRatio`：画幅（默认 `"9:16"`，横屏可改 `"16:9"`、`"2.39:1"`）
- `filmTargetShots`：每集目标镜头数（`0` = 随本集剧情自动决定）
- `filmShotDurationSec`：默认单镜时长秒数（`0` = 随剧情自动分配，或设 `4` / `8` / `10` / `15`）
- `filmStyle`：全局视觉风格（如 `"写实电影感、冷暖对比、35mm 胶片颗粒"`）
- `filmReviewMaxRounds`：最多自动修订轮数（预设 `2`；`0` = 只审不改）
- `filmCastKinds`：要生成设定图的实体类别，默认 `"character,location"`（可加 `"prop"`）
- `filmRenderMode`：`"keyframe"`（只拍关键帧）或 `"video"`（关键帧 + 图生视频，默认 `"video"`）
- `filmShotRange`：镜头范围过滤（空串 = 全部，支持 `"1-5,8"`）
- `filmRenderConcurrency`：最大并发场次数（`1`~`8`，默认 `3`）
- `filmRenderContinuity`：同场次相邻镜头连续性参考（默认 `true`）
- `filmRenderQc`：是否开启关键帧视觉质检（默认 `true`）
- `filmRenderMaxRetries`：质检不通过自动重拍次数（默认 `1`）
- **模型字段（切勿混用）**：
  - `filmTextModel` / `filmTextProviderInstanceId`（别名 `textModel`）：① 分镜拆解与 ② 导演审核使用的**文本大模型**（如 `deepseek/deepseek-v4-pro`）
  - `generateModel` / `generateProviderInstanceId`（别名 `imageModel`）：③ 角色场景设定图与 ④ 镜头关键帧使用的**图片生成模型**（⚠️ 切勿将文本模型填在此字段！）
  - `filmVideoModel` / `filmVideoProviderInstanceId`（别名 `videoModel`）：④ 批量拍摄图生视频使用的**视频生成模型**（如 `veo3.1-fast`）
  - `filmQcModel` / `filmQcProviderInstanceId`（别名 `qcModel`）：④ 关键帧视觉质检使用的多模态文本模型

---

## 阶段四：`output.timeline`（短剧成片时间线）

### 端口定义
- **输入端口**：
  - `in-videos`（`videos`，可多连）：接收各集 `film.workbench` 的 `out-videos` 镜头视频序列（按节点标题中的集号 `第 1 集`、`第 2 集` 及连线顺序自动排列各集全部镜头）
  - `in-images`（`images`，可多连）：当短剧工作台使用 `"keyframe"` 动态漫模式时，也可将 `out-images` 接入 `in-images` 合成动态漫成片
  - `in-voice` / `in-voices`（`voice` / `voices`）：可选旁白或配音轨
  - `in-music`（`voice`）：接收 `asset.music` 的背景配乐轨
  - `in-text`（`text`）：可选外挂字幕文本
- **输出端口**：
  - `out`（`video`）：合成导出的最终 MP4 成片视频

### 排序与混音机制
- **多集镜头自动排序**：`output.timeline` 根据上游 `film.workbench` 节点标题中的集号（如 `第 1 集 · 短剧工作台`、`第 2 集 · 短剧工作台`）及连线顺序，自动保持第 1 集全部镜头在前、第 2 集全部镜头在后的完整叙事顺序。
- **配乐与转场合成**：接入 `in-music` 的背景配乐会自动与镜头音轨混音（具备人声自动闪避），并导出高清 MP4 成片；双击节点可进入多轨时间线精剪台进行逐帧微调。

---

## 结构化分镜 JSON（`params.filmStoryboard`）

schema 固定为 `aiart.film-storyboard@1`，主要结构：

- `entities[]`：`{ id, kind: "character" | "location" | "prop", name, description, locked? }`
  - 外观描述统一收敛在 `entities[].description`，镜头提示词里直接写实体名称，拍摄时按出场顺序自动绑定 `@1`、`@2` 参考图。
- `shots[]`：`{ id, sceneId, shotSize, angle, cameraMove, durationSec, characters, location, props, action, dialogue, imagePrompt, videoPrompt, locked?, status?, review?, media? }`
  - 将某镜的 `locked` 设为 `true` 后，重新拆解或导演审核都不会覆盖该镜。
