---
name: wf-short-drama
description: 用「影视全流程」工作流完成从「创建剧本（asset.screenplay）→ 剧本分集（text.select）→ 分集短剧工作台（film.workbench，跨集共享角色场景设定库 Series Bible）→ 成片时间线（output.timeline）」的完整多集短剧/漫剧/影视制作闭环。当用户要从故事大纲或剧本出发制作短剧、漫剧、分集影视成片，需要创建剧本、按集拆分、跨集角色场景一致性、分镜审核拍摄与成片时间线合成时加载本技能；若只需单镜手动拆出精修可配合 wf-short-drama9。
workflow: short-drama
workflow-version: 2.0.0
---

# 影视全流程 · 创建剧本 → 剧本分集 → 分集短剧工作台 → 成片时间线（short-drama，9 节点 / 9 连线）

> **全链路架构**：本工作流完整覆盖影视与短剧工业化生产的四大核心阶段 —— **① 创建剧本（`asset.screenplay`）** → **② 剧本分集（`text.select`）** → **③ 分集短剧工作台（`film.workbench`）** → **④ 成片时间线（`output.timeline`）**，支持多集连载角色与场景设定库（Series Bible）跨集自动继承与一键成片合成。

## 拓扑总览与产出物

```
故事大纲与世界观 (text.plain: outline)
  │ out → in
  ▼
创建剧本 · 剧本工作台 (asset.screenplay: screenplay)
  │ out-all (texts：自动按「第1集 / 第2集」切分各集剧本，并每集附带共享【人物小传】)
  ├───────────────────────────────┐
  ▼ in                            ▼ in
剧本分集 · 第 1 集               剧本分集 · 第 2 集
(text.select: ep1_script)        (text.select: ep2_script)
[selectedTextId = "ep:1"]        [selectedTextId = "ep:2"]
  │ out → in                      │ out → in
  ▼                               ▼
第 1 集 · 短剧工作台             第 2 集 · 短剧工作台
(film.workbench: ep1_workbench)  (film.workbench: ep2_workbench)
  │  ├─ ① 分镜拆解 (storyboard)   ▲  ├─ ① 分镜拆解 (自动继承第1集常驻角色/场景标准名)
  │  ├─ ② 导演审核 (自动修订)     │  ├─ ② 导演审核 (自动修订)
  │  ├─ ③ 角色场景设定 ──out-cast─┴──in-cast─ ③ 角色场景设定 (复用第1集设定图，仅画新实体/换装)
  │  └─ ④ 批量拍摄 (关键帧+视频)     └─ ④ 批量拍摄 (关键帧+视频)
  │ out-videos                    │ out-videos
  └──────────────┬────────────────┘
                 ▼ in-videos                      全剧悬疑氛围配乐 (asset.music: bgm)
         短剧成片时间线 (output.timeline: timeline) ◄──in-music── out
                 │
                 └─ out (video：多集镜头自动拼接 + 转场 + 配乐混音导出的最终 MP4 成片)
```

### 四大阶段核心产出

1. **阶段一 · 创建剧本（`asset.screenplay`）**：
   - 接收上游 `text.plain` 的故事大纲或世界观，按标准影视格式创作包含 `剧本名：...`、`【人物小传】` 以及显式分集标题（`第1集：...`、`第2集：...`）的完整多集剧本。
   - 双击节点可进入全屏**剧本工作台**（支持结构化场次编辑与 **3D 剧本运镜预演**）。
   - 节点运行后自动识别正文中的 `第1集`、`第2集` 标记切分为多集剧本条目，并将顶部的 `【人物小传】` 自动拼接到每一集剧本开头，从 `out-all`（`texts` 端口）输出分集数组。
2. **阶段二 · 剧本分集（`text.select`）**：
   - `ep1_script`（`selectedTextId: "ep:1"`）与 `ep2_script`（`selectedTextId: "ep:2"`）接入 `screenplay` 的 `out-all` 端口，各自精准提取第 1 集与第 2 集剧本（即使上游剧本重新生成，按 `"ep:1"` / `"ep:2"` 仍稳定命中最新一轮的对应集数）。
   - 需要扩展到第 3 集、第 N 集时，只需新增 `text.select`（设 `selectedTextId: "ep:3"`）并接入对应的 `film.workbench` 节点。
3. **阶段三 · 分集的短剧工作台（`film.workbench`）**：
   - 每一集配备独立的 `film.workbench`（`ep1_workbench`、`ep2_workbench`），双击即可进入该集专属的全屏四合一工作台，依次完成 **① 分镜拆解 → ② 导演审核自动修订 → ③ 角色与场景设定图 → ④ 批量关键帧与视频拍摄（含视觉质检重拍）**。
   - **跨集全剧圣经（Series Bible）串联**：`ep1_workbench` 的 `out-cast` 连入 `ep2_workbench` 的 `in-cast` 端口。第 2 集拆解分镜时自动沿用第 1 集的主角与常驻场景标准名称，生成设定图时直接复用第 1 集已生成的角色/场景设定图，只为第 2 集新登场的场景或换装造型（如 `林夏#夜行装`）生成新设定图。
4. **阶段四 · 成片时间线（`output.timeline`）**：
   - `timeline` 节点通过 `in-videos` 按集序接收 `ep1_workbench` 与 `ep2_workbench` 的 `out-videos` 镜头视频序列，并通过 `in-music` 接收 `bgm`（`asset.music`）生成的全剧氛围配乐。
   - 运行后自动按「第 1 集全部镜头 → 第 2 集全部镜头」顺序排列视觉轨道、添加平滑转场、混音背景配乐并渲染导出最终 MP4 成片；双击 `timeline` 节点可进入多轨时间线精剪台微调。

## 怎么操作（MCP 路径）

1. **安装或复用工作流创建画布资产**：
   - 调用 `workflow_list_installed` 确认已安装 `short-drama`；
   - 调用 `workflow_use_installed({ id: "short-drama", name?, generateAspectRatio?, textModel?, imageModel?, videoModel? })` 创建画布资产并获取 `assetId`（竖屏短剧默认 `"9:16"`，横屏影视可传 `"16:9"`；传入的 `textModel` 会同时注入 `asset.screenplay` 与各集 `film.workbench`，`imageModel` 与 `videoModel` 会注入各集 `film.workbench`）。
2. **读取节点并写入故事大纲或剧本**：
   - `graph_read({ assetId, includeParams: true })` 查看节点列表（`outline`、`screenplay`、`ep1_script`、`ep2_script`、`ep1_workbench`、`ep2_workbench`、`bgm`、`timeline`）。
   - **方式 A（从大纲自动写多集剧本）**：用 `graph_edit` 修改 `outline`（`text.plain`）的 `params.text` 为用户的故事大纲与角色设定，**同时将 `screenplay` 的预设示例 `params.text` 清空为 `""`**，在 `screenplay`（`asset.screenplay`）的 `params.generateInstruction` 中补充集数与风格要求，并修改 `bgm`（`asset.music`）的配乐提示词。
   - **方式 B（用户已提供完整多集剧本）**：直接用 `graph_edit` 把含 `【人物小传】` 与 `第1集：...`、`第2集：...` 的完整剧本写入 `screenplay` 的 `params.text`。
   - **⚠️ 模型参数区分（切勿混淆）**：
     - `asset.screenplay` 的文本大模型字段是 `generateModel`（及 `generateProviderInstanceId`）；
     - `film.workbench` 是多模态复合节点：**文本大模型**必须填在 `filmTextModel`（或别名 `textModel`），**图片生成模型**填在 `generateModel`（或别名 `imageModel`），**视频生成模型**填在 `filmVideoModel`（或别名 `videoModel`）。**严禁把 `deepseek` 等文本模型填进 `film.workbench` 的 `generateModel`！**
3. **分阶段递进运行（推荐标准范式，避免单节点全量跑视频超时中断）**：
   - **阶段 3.1（剧本与分集）**：调用 `task_run({ assetId, nodeIds: ["screenplay", "ep1_script", "ep2_script"] })`（若有第 3 集则含 `"ep3_script"`），几秒内产出并确认各集剧本文本。
   - **阶段 3.2（全集分镜拆解 `breakdown`）**：用 `graph_edit` 将各集 `film.workbench` 设为 `filmWorkbenchRunMode: "breakdown"`，调用 `task_run` 运行各集工作台（可并发或串联）。此阶段仅调用文本大模型，每集约 20~40 秒即可落盘结构化分镜表 `filmStoryboard`，用户在画布中可立即查看所有分镜。
   - **阶段 3.3（可选导演审核 `review` 与角色场景设定图 `cast`）**：
     - 若需 AI 导演审片修订，设 `filmWorkbenchRunMode: "review"` 运行；
     - 随后设 `filmWorkbenchRunMode: "cast"`，**按集序依次运行** `ep1_workbench` → `ep2_workbench`（→ `ep3_workbench`），让第 1 集产出的主角/常驻场景设定图通过 `out-cast → in-cast` 自动传递给后续集数复用。
   - **阶段 3.4（批量拍摄 `render` 与成片合成 `timeline`）**：
     - 将各集 `film.workbench` 设为 `filmWorkbenchRunMode: "render"`（若只需快速预览画面可设 `filmRenderMode: "keyframe"`；若需生成动态视频则设 `filmRenderMode: "video"`），逐集运行 `task_run` 并在每集完成后向用户汇报进度；
     - 最后调用 `task_run({ assetId, nodeIds: ["bgm", "timeline"] })` 合成导出最终 MP4 成片。
   - *注：仅当镜头数极少（如每集 2~3 镜）或用户明确要求无人值守长跑时，才直接用 `filmWorkbenchRunMode: "all"` 一次性跑完单集四阶段。*
4. **轮询状态与交付**：
   - 使用 `task_status({ mcpTaskId })` 轮询至当前阶段任务完成，向用户汇报已落盘的分集剧本、分镜表、角色场景设定图、各集分镜关键帧/视频以及 `timeline` 导出的最终成片路径。

## 扩展更多集数与单镜精修

- **扩展第 3 集 / 第 N 集**：
  1. `node_upsert` 新增 `ep3_script`（`typeId: "text.select"`, `params: { selectedTextId: "ep:3" }`）与 `ep3_workbench`（`typeId: "film.workbench"`, 标题须包含 `"第 3 集"` 如 `"第 3 集 · 短剧工作台"` 以便时间线按集序排列，并同步配置 `filmAspectRatio`、`filmCastKinds: "character,location"`、`filmTextModel`、`generateModel`、`filmVideoModel`）；
  2. `edge_connect` 连接：`screenplay (out-all) → ep3_script (in)`、`ep3_script (out) → ep3_workbench (in)`、`ep2_workbench (out-cast) → ep3_workbench (in-cast)`、`ep3_workbench (out-videos) → timeline (in-videos)`。
- **改某一镜并只重拍该镜**：
  1. `graph_read({ assetId, includeParams: true })` 读取对应集 `film.workbench` 的 `params.filmStoryboard`（schema `aiart.film-storyboard@1`）；
  2. 修改目标镜头的提示词/台词并将 `locked` 设为 `true`，用 `graph_edit` 写回，同时可设 `filmWorkbenchRunMode: "render"` 与 `filmRenderOnlyShotId: "<镜头id>"` 单独重拍该镜，随后重跑 `timeline` 更新成片。
- **单镜拆出深度精修**：从任一集 `film.workbench` 的 `out` 与 `out-cast` 引出 `film.shot` 节点，接 `asset.image` / `asset.video` 单独打磨特定高光镜头。

## 参数与端口速查

完整的 9 节点拓扑、端口定义、分集选择器语法与 `film.workbench` / `output.timeline` 参数速查见 `references/stages.md`。
