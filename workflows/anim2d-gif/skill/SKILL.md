---
name: wf-anim2d-gif
description: 用「2D 帧动画」工作流生成游戏级 2D 角色序列帧雪碧图（Sprite Sheet）与去底循环透明 GIF：角色基准立绘（charRef）→ 可控序列图生成（frameGen: frame.animGen，支持 ai-sheet / video-sheet / video-restyle / frames-sheet 四种模式）→ 2D 帧动画切格抠底稳像（anim: anim.2d），并可按需动态扩展待机/奔跑/攻击多动作并行管线。
workflow: anim2d-gif
workflow-version: 2.1.0
argument-hint: "[2D角色设定/画风（像素风/赛璐璐/Q版脊椎动画）/动作需求]"
when-to-use: 用户需要制作 2D 游戏角色序列帧动画、Sprite Sheet 雪碧图、透明背景 GIF 动图或待机/行走/奔跑/攻击游戏动作资产包时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status
---

# 2D 帧动画 · 可控序列图生成与透明 GIF 工业管线 SOP（anim2d-gif v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，5 节点 / 3 连线）

```
角色立绘基准·锚点 (asset.image: charRef, 1:1 纯色干净底)
  ├─out→in────────→ 生成帧动画序列图 (frame.animGen: frameGen, 四模式可控序列图)
  │                    └─out→in──→ 2D帧动画 (anim.2d: anim, 切帧 + 键控抠底 + 脚底稳像 → out-gif 透明动图)
  └─out→in-image──→ 备用直出序列图 (asset.image: sheet)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "anim2d-gif", mode: "seed" })` 物化后，画布上的节点 `id` 与上方 `key` 完全一致（`charRef`、`frameGen`、`sheet`、`anim`、`note`），可直接用于 `graph_edit` 与 `task_run`。

## 核心算子能力拆解

1. **`charRef`（`asset.image` · 角色立绘基准锚点）**：
   - 生成**纯色高对比干净背景、无地面阴影、正侧面全身标准站姿**的 2D 角色立绘，锁定头身比、描边粗细与服装色板。
2. **`frameGen`（`frame.animGen` · 生成帧动画序列图）**：
   - 支持 4 种可控模式（`params.animGenMode`）：
     - `"ai-sheet"`（默认）：基于 `in` 角色基准图 + 动作预设（`animPresetId`: `"idle" | "walk" | "run" | "attack" | "jump"`）生成 `animRows × animCols` 连续动作序列图；
     - `"video-sheet"`：从 `in-video` 动作参考视频在 `[animVideoStartRatio, animVideoEndRatio]` 区间均匀抽帧、去白边并按 `animStabilize`（`"ground"` 脚底对齐 / `"center"` 质心居中）稳像拼合序列图；
     - `"video-restyle"`：将参考视频抽帧拼成姿态网格 + `in` 角色图交由 AI 严格按姿态重绘；
     - `"frames-sheet"`：将多张散帧对齐拼合为标准序列图。
3. **`anim`（`anim.2d` · 2D 帧动画切格、抠底与稳像）**：
   - 按 `animRows × animCols` 切分序列图，自动采样四角背景色键控透明（`animKeyColor: "auto" | "black" | "white"`），按 `animStabilize: "ground"` 对齐每帧脚底防止动画左右漂移；
   - 端口输出：`out`（单帧 PNG）、`out-all`（全部帧 PNG 列表，可直连游戏引擎）、`out-gif`（合成循环 GIF 动图，`animGifFps` 控制帧率，`animPlayMode: "loop" | "pingpong"`）。

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

`workflow.json` 仅提供单动作最小种子骨架，Agent 应根据用户需求通过 `graph_edit` 自由伸缩：

- **裁剪精简（单条快速序列帧）**：若无需 `sheet`（备用直出序列图），可用 `{ op: "node_delete", nodeId: "sheet" }` 删掉以节省一次生图调用。
- **横向扩展（一套角色同时出 Idle + Run + Attack 三套动作 GIF）**：
  保留唯一 `charRef` 锚点，用 `graph_edit` 的 `node_upsert` 新增 `frameGen_run`（`typeId: "frame.animGen"`, `animPresetId: "run"`）→ `anim_run`（`typeId: "anim.2d"`, `animGifFps: 12`）以及 `frameGen_atk` → `anim_atk`，全部从 `charRef:out` 接入 `in`，一次并发跑出全套游戏动作包。
- **视频姿态驱动（Video-to-Sprite）**：若用户提供了实拍动作视频或想复刻某段动作，新增 `refVid`（`asset.video`）连入 `frameGen:in-video`，将 `frameGen` 的 `animGenMode` 设为 `"video-restyle"` 或 `"video-sheet"`。
- **汇入成片时间线**：若用户想把多组 2D 角色 GIF 做成带音效的角色技能展示短片，可新增 `timeline`（`output.timeline`）与 `sfx`（`asset.sfx`），将 `anim:out-gif` 连入 `timeline:in-image`。

## 标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "anim2d-gif", mode: "seed" })` 在当前画布加载种子节点（若在已有画布叠加则用 `mode: "merge"`）。
2. 根据用户需要的动作套数，用 `graph_edit` 更新 `charRef`、`frameGen`、`anim` 的 `params`（如 `animRows`、`animCols`、`animPresetId`、`animGifFps`、`generateInstruction`），并按需增删动作分支。
3. 调用 `task_run({ assetId })` 执行生成，通过 `task_status` 跟踪进度，完成后汇报序列图与透明 GIF 产物。

## 端口与参数速查

详见 `references/ports.md`。
