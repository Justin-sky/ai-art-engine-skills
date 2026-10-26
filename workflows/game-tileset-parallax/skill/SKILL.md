---
name: wf-game-tileset-parallax
description: 用「2D 瓦片地图与视差背景」工作流生成平台跳跃/银河城/俯视角RPG/肉鸽/模拟经营游戏的 3×3 Auto-Tile 九宫格地形瓦片图集、100% 零接缝平铺地表材质、2D动态光照法线贴图（Normal Map）以及远中近多层横向无缝视差滚轴背景（Parallax Background）。
workflow: game-tileset-parallax
workflow-version: 1.0.0
argument-hint: "[关卡主题（奇幻森林/地下城地牢/赛博朋克都市/废土沙漠/冰雪遗迹）/视角（横版平台side/俯视角RPG top/等轴测iso）/画风（像素风/手绘厚涂/赛璐璐）]"
when-to-use: 用户需要制作 2D 游戏地图瓦片（Tileset / Tilemap）、Auto-Tile 九宫格地形、无缝平铺贴图、2D 光照法线贴图或横版多层视差滚轴背景时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_tile_pbr_process, game_bundle_export
---

# 2D 瓦片地图与多层视差背景 · 关卡场景资产工业管线 SOP（game-tileset-parallax v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，6 节点 / 1 连线）

```
九宫格地形瓦片图集 (frame.animGen: autoTileSheet, 3×3 autotile3x3 原生透明边缘)
  └─out→in──→ 3×3 瓦片切格器 (image.gridSplit: tileSplitter → 9 张标准地形块)
无缝平铺地表基础纹理 (asset.image: seamlessGround → 配合 game_tile_pbr_process 实现零接缝平铺 + 派生 Normal/Roughness/AO)
视差远景层 (asset.image: skyFarBg, 16:9 远景天空山脉)
视差中景层 (asset.image: midForestBg, 16:9 原生透明背景中景层，可叠加在远景之上横向无限卷轴)
```

---

## 二、关卡瓦片与视差背景核心工程规范

1. **规范 1 · 九宫格 Auto-Tile 瓦片图集（`autoTileSheet` → `tileSplitter`）**：
   - `autoTileSheet` 使用 `frame.animGen` 的 `animPresetId: "autotile3x3"`，`animRows: 3, animCols: 3, generateAspectRatio: "1:1"`；
   - **横版平台跳跃（Platformer / Metroidvania）**：设 `animViewAngle: "side"`，产出顶面平台草皮/石砖、左右垂直岩壁、中心泥土填充与底部悬崖边缘；
   - **俯视角 RPG / 肉鸽 / 模拟经营（Top-Down RPG）**：设 `animViewAngle: "top"`，产出草地—水域、石路—泥地或地牢墙体—地板的九宫过渡瓦片；
   - `animStabilize` 必须保持 `"off"`，防止瓦片被角色稳像逻辑缩放位移而破坏网格对齐！

2. **规范 2 · 本地 C1 零接缝平铺融合与 PBR 法线派生（`game_tile_pbr_process`）**：
   - AI 模型直出的纹理在四边拼合时往往有细微色差接缝；
   - 在 `seamlessGround` 节点生成完成后，**必须调用 MCP 工具 `game_tile_pbr_process`**：
     ```json
     {
       "assetId": "<seamlessGround产出资产ID>",
       "makeSeamless": true,
       "blendRatio": 0.18,
       "derivePbr": true,
       "pbrMaps": ["normal", "roughness", "ao", "orm"],
       "autoTileLayout": "grid3x3"
     }
     ```
   - 该工具会在本地完成正弦对偶无缝边缘闭合（使平铺接缝色差降为 0），并自动生成 Unity URP 2D / Godot `CanvasTexture` 动态点光源所需的切线空间法线贴图（`_normal.png`）与 `_autotile.json` 掩码配置！

3. **规范 3 · 多层视差背景水平无限滚动（Horizontal Seamless Parallax）**：
   - 远景层（`skyFarBg`）为实底 16:9，中景层（`midForestBg`）及可扩展的近景层（`foreLayer`）必须开启 `generateTransparentBg: true`；
   - 对视差层调用 `game_tile_pbr_process({ assetId, makeSeamless: true, wrapX: true, wrapY: false })`（仅开启水平方向 `wrapX: true`，关闭垂直方向 `wrapY: false`），即可让远景与中景在游戏相机水平移动时实现 100% 零接缝无限循环卷轴！

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "game-tileset-parallax", mode: "seed" })` 在当前打开的画布上创建种子节点。
2. 根据用户的关卡主题（如赛博霓虹街区、深渊地下城、像素田园牧场）调用 `graph_edit` 修改各节点提示词与视角参数。
3. 调用 `task_run({ assetId })` 跑通图集切分与视差背景生成。
4. 对生成的地表贴图与视差层调用 `game_tile_pbr_process` 完成无缝闭合、法线贴图派生与 Auto-Tile 清单落盘。

## 四、端口与参数速查

详见 `references/ports.md`。
