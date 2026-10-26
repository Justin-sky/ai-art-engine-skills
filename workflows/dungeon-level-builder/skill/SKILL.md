---
name: wf-dungeon-level-builder
description: 用「2D地牢与平台关卡设计套件」工作流一键生成关卡地形瓦片集、9 枚关卡可交互物件（宝箱/传送门/地刺陷阱/祭坛/机关拉杆）透明切图包、关卡探索背景音乐，并配合 game_level_map_generate 与 game_systems_graph_generate 自动生成可玩 Tiled (.tmj) 关卡地图（支持 BSP 地牢、有机洞穴、横版平台跳跃、肉鸽竞技场）及 Boss AI 状态机。
workflow: dungeon-level-builder
workflow-version: 1.0.0
argument-hint: "[关卡类型（BSP地牢/银河城洞穴/横版平台跳跃/割草竞技场）/美术主题/交互机关清单]"
when-to-use: 用户需要设计 2D 游戏关卡地图、生成 Tiled (.tmj) 关卡文件、制作地牢/平台跳跃场景交互物件（宝箱/传送门/机关）或配置敌人 AI 状态机时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_tile_pbr_process, game_audio_variants, game_level_map_generate, game_systems_graph_generate, game_bundle_export
---

# 2D地牢与平台关卡设计套件 · 工业管线 SOP（dungeon-level-builder v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，6 节点 / 3 连线）

```
关卡无缝地形瓦片集 (asset.image: tilesetImg, 1:1 地形瓦片表)
关卡可交互物件清单 (text.plain: propNames, 9 枚宝箱/传送门/陷阱/祭坛/拉杆)
  ├─→ 关卡交互物件·3×3整版 (asset.image: propSheet)
  └─→ 关卡物件切格透明打包 (image.iconPack: propPack ← propSheet)
地牢关卡探索循环配乐 (asset.music: dungeonBgm)
```

---

## 二、游戏关卡设计三大工程化铁律

1. **铁律 1 · 四大关卡拓扑算法按品类精准选型（`game_level_map_generate`）**：
   - **RPG / 冒险地牢**：使用 `algorithm: "bsp_dungeon"`，自动通过二叉空间分割生成出生房、普通刷怪房、宝藏房、商店房、Boss 房与 L 型连通走廊；
   - **银河恶魔城 / 挖矿沙盒**：使用 `algorithm: "cellular_cave"`，通过元胞自动机 + Flood-fill 连通域修剪生成无孤岛有机洞穴；
   - **横版动作 / 平台跳跃**：使用 `algorithm: "platformer_course"`，自动排布地面断崖、高低踏板、巡逻怪、金币弧线与关底终点；
   - **肉鸽割草 / 射击竞技场**：使用 `algorithm: "arena_waves"`，生成带对称掩体柱、中央祭坛与外圈刷怪环的竞技场。

2. **铁律 2 · 标准 Tiled (`.tmj`) 双图层 + 实体对象层直出**：
   - `game_level_map_generate` 自动计算每个固体的 4-bit Wang-16 自动瓦片掩码，并在 `.tmj` 中输出 `Ground`、`WallsAndPlatforms` 与 `Entities`（包含 `player_spawn`、`boss_spawn`、`treasure_chest`、`shop_npc`、`trap`、`exit_portal` 像素坐标），可直接拖入 Godot 4 / Unity / Phaser 3 运行。

3. **铁律 3 · 关卡任务链与 Boss AI 状态机同步落盘（`game_systems_graph_generate`）**：
   - 调用 `game_systems_graph_generate` 为关卡配套生成敌人/Boss 有限状态机（`idle` → `chase` → `attack` → `enrage`）及关卡主线/支线任务配置。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "dungeon-level-builder", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户的关卡题材修改 `tilesetImg`、`propNames`、`propSheet` 与 `dungeonBgm` 参数并调用 `task_run` 并发生成。
3. 调用 `game_level_map_generate` 导出 `.tmj` 关卡地图与 ASCII 预览图。
4. 调用 `game_systems_graph_generate` 生成配套的 Boss AI 状态机与任务树。

## 四、端口与参数速查

详见 `references/ports.md`。
