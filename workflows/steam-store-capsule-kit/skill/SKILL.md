---
name: wf-steam-store-capsule-kit
description: 用「Steam商店胶囊图与宣传片」工作流一键生成独立游戏上架 Steam / Epic / TapTap / Itch.io 所需的横版主视觉胶囊图（16:9）、竖版游戏库封面（3:4）、原生透明通道游戏标题 Logo、商店动态宣传片镜头（Trailer）与史诗预告片配乐（BGM），并通过时间轴节点合成为可直传商店后台的宣传成片。
workflow: steam-store-capsule-kit
workflow-version: 1.0.0
argument-hint: "[游戏名称与美术题材/主角与核心视觉意象/Logo风格/宣传片镜头与配乐基调]"
when-to-use: 用户需要制作 Steam 商店页胶囊图（Header/Library Capsule）、游戏透明 Logo、独立游戏发售预告片（Reveal/Launch Trailer）或宣发物料包时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, timeline_tracks, timeline_compose, game_bundle_export
---

# Steam商店胶囊图与宣传片 · 工业管线 SOP（steam-store-capsule-kit v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，7 节点 / 3 连线）

```
Steam横版主视觉胶囊图 (asset.image: headerCapsule, 16:9 预留标题负空间)
  └─out→in──→ 商店页动态宣传片镜头 (asset.video: trailerVid, 16:9 首帧驱动)
                 └──────────→ 商店宣传片音画合成时间轴 (output.timeline: storeTimeline)
宣传片史诗交响配乐 (asset.music: trailerBgm, 纯器乐) ──┘
Steam竖版游戏库封面 (asset.image: verticalCapsule, 3:4 竖幅构图)
原生透明通道游戏标题Logo (asset.image: gameLogo, 16:9 原生透明背景)
```

---

## 二、游戏商店页宣发视觉三大铁律

1. **铁律 1 · 胶囊图构图预留标题负空间（Negative Space for Title）**：
   - Steam 横版胶囊图（Header/Main Capsule）与竖版库封面（Library Capsule）切忌由 AI 直接在原画中乱画畸变文字；
   - `headerCapsule` 必须在左侧或右侧 1/3 预留干净负空间，`verticalCapsule` 在顶部 1/4 预留负空间，并显式声明“不要生成任何文字”。

2. **铁律 2 · 游戏标题徽标 Logo 原生透明直出（`generateTransparentBg: true`）**：
   - `gameLogo` 节点强制开启 `generateTransparentBg: true`，直接产出边缘锐利的 RGBA 透明通道徽标，方便叠加到不同尺寸的商店胶囊图及预告片片尾定帧上。

3. **铁律 3 · 主视觉首帧驱动宣传片 + 史诗配乐时间轴合成**：
   - 将 `headerCapsule` 的输出连入 `trailerVid` 的首帧输入（`fromPort: "out", toPort: "in"`），保证商店预告片色调、主角造型与商店封面 100% 统一；
   - 将 `trailerVid` 与 `trailerBgm` 汇入 `storeTimeline`，即可一键导出音画同步的商店宣传视频。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "steam-store-capsule-kit", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户的游戏题材修改 `headerCapsule`、`verticalCapsule`、`gameLogo`、`trailerVid` 与 `trailerBgm` 的提示词。
3. 先并发运行 `headerCapsule`、`verticalCapsule`、`gameLogo` 与 `trailerBgm`；待 `headerCapsule` 完成后运行 `trailerVid`，最后在 `storeTimeline` 合成导出。
4. 如需连同游戏内资产一并归档交付，可调用 `game_bundle_export` 生成包含宣发物料与引擎导入脚本的完整资产包。

## 四、端口与参数速查

详见 `references/ports.md`。
