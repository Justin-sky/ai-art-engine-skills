---
name: wf-character-sheet
description: 用「角色设定」工作流从角色设定圣经（bio）出发，先生成正面全身基准立绘（front）作为唯一视觉锚点（通过 in-image 锁死五官骨相、发色与服装结构），再并发衍生侧面正交立绘（side）、背面立绘与披挂拆解（back）、四格面部表情表（expr）与标志性战斗动态立绘（action），并支持动态扩展 360° 环绕视频或 3D 模型生成。
workflow: character-sheet
workflow-version: 2.1.0
argument-hint: "[角色姓名/性格/体型外貌/服装道具/美术风格]"
when-to-use: 用户需要设计游戏/动漫/漫剧角色原画设定集、正侧背三视图、表情包设定表或锁死角色一致性锚点时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status
---

# 角色设定全案 · 主锚点锁死多视图与衍生资产 SOP（character-sheet v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，7 节点 / 9 连线）

```
角色设定圣经 (text.plain: bio)
  ├─out→in-text──→ ① 正面基准立绘·锚点 (asset.image: front, 3:4 标准 A-Pose)
  │                  ├─out→in-image──→ ② 侧面正交立绘 (asset.image: side, 3:4)
  │                  ├─out→in-image──→ ③ 背面立绘与披挂拆解 (asset.image: back, 3:4)
  │                  ├─out→in-image──→ ④ 四格面部表情表 (asset.image: expr, 1:1 2×2 表情特写)
  │                  └─out→in-image──→ ⑤ 标志性战斗动态立绘 (asset.image: action, 3:4)
  └─out→in-text──────┴─────────────────┴──────────────────────┴────────────────────────┘
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "character-sheet", mode: "seed" })` 后，画布节点 `id` 即为 `bio`、`front`、`side`、`back`、`expr`、`action`、`note`，无需猜测随机 UUID。

## 为什么必须采用「主锚点辐射拓扑」

多视图设定若各自只吃文本独立生成，模型会画出完全不同的人。本工作流强制以 **`front`（正面基准立绘）** 为唯一视觉源头，将其 `out` 连入 `side`、`back`、`expr`、`action` 的 `in-image` 端口：
- **`side`（侧面正交立绘）** & **`back`（背面立绘与披挂拆解）**：严格继承 `front` 的头身比、发色与服装剪裁，交代侧面轮廓厚度与背部披风/挂件结构；
- **`expr`（四格面部表情表）**：锁定 `front` 的脸型、瞳色与发型，在 1:1 画布绘制 2×2 四种核心情绪微表情；
- **`action`（标志性战斗动态立绘）**：继承全套穿搭与专属武器，展现极具张力的招牌动作。

## 提示词覆写规范（Agent 必读）

1. **`bio`（`text.plain`）**：结构化撰写「角色姓名/年龄体态 + 面部骨相与发型发色 + 主色/辅色/点缀色比例 + 上装/下装/鞋履/配饰材质 + 标志性随身武器 + 渲染画风 + 绝对禁止项（纯色干净底、无文字水印）」。
2. **分步省流策略（推荐）**：若用户希望先确认正面立绘再衍生其余视图，可先调用 `task_run({ assetId, nodeIds: ["front"] })`，用户满意后再调用 `task_run({ assetId })` 并发跑完所有衍生节点。

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

Agent 应根据用户的实际用途通过 `graph_edit` 灵活增删节点，绝不僵化照搬固定 5 图：

- **按需裁剪（Prune Down）**：
  - 若用户只需「立绘 + 表情包」（如对话小说/AVG 头像），用 `node_delete` 删除 `side`、`back`、`action`；
  - 若用户只需「正侧背标准三视图」供 3D 建模参考，删除 `expr` 与 `action`。
- **扩展动态展示与语音（Scale Up）**：
  - **角色 360° 环绕动态展示视频**：`node_upsert` 新增 `turntableVid`（`typeId: "asset.video"`, `params: { generateInstruction: "角色原地 360 度平滑环绕展示，呼吸感自然，服装材质光影流转", generateAspectRatio: "9:16" }`），连接 `front:out → turntableVid:in-image`；
  - **角色招牌台词语音 + 开口说话**：新增 `charVoice`（`asset.voice`）+ `lipSync`（`video.lipSync`），让角色立绘开口说出招牌台词；
  - **一键转 3D 角色白模/带贴图模型**：新增 `model3d`（`typeId: "asset.model3d"`），连接 `front:out → model3d:in-image`，直接将立绘转化为 GLB 3D 资产；
  - **衔接 2D 序列帧动画**：用 `workflow_use_installed({ id: "anim2d-gif", assetId, mode: "merge" })` 将 2D 帧动画管线合入当前画布，把 `front:out` 接给 `frame.animGen:in`。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "character-sheet", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 写入 `bio` 的角色设定正文，定制 `front`、`side`、`back`、`expr`、`action` 的提示词与画幅比例，并按需增删衍生算子。
3. `task_run({ assetId })` 执行生成并用 `task_status` 跟踪完成。

## 端口与参数速查

详见 `references/ports.md`。
