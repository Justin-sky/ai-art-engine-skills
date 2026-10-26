---
name: wf-roguelike-survivor-pack
description: 用「肉鸽割草与弹幕生存套件」工作流一键生成《吸血鬼幸存者》《土豆兄弟》类俯视角肉鸽游戏的主角与怪潮移动序列帧、弹幕投射物/经验水晶/遗物 3×3 透明图标包、爽快割草拾取音效，并配合本地算子自动提取 2D 物理碰撞多边形（CollisionPolygon2D）与生成刷怪波次数值配表（waves.csv）。
workflow: roguelike-survivor-pack
workflow-version: 1.0.0
argument-hint: "[肉鸽题材（暗黑哥特/赛博机甲/修仙御剑/魔法学院）/主角设定/怪物潮类型/弹幕武器与掉落物清单]"
when-to-use: 用户需要制作肉鸽割草（Survivor-like）、弹幕射击、双摇杆射击或地牢生存游戏的全套角色、怪物潮、弹幕道具图标、碰撞体与波次配表时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_tile_pbr_process, game_audio_variants, game_data_table_generate, game_bundle_export
---

# 肉鸽割草与弹幕生存套件 · 工业管线 SOP（roguelike-survivor-pack v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，10 节点 / 6 连线）

```
幸存者主角基准立绘 (asset.image: heroRef, 1:1 原生透明背景)
  └─out→in──→ 主角移动循环序列图 (frame.animGen: heroSheet, 2×2 原生透明序列图)
                 └─out→in──→ 主角精灵切帧与GIF (anim.2d: heroAnim)
怪物潮杂兵追击序列图 (frame.animGen: mobSheet, 2×2 原生透明序列图)
  └─out→in──→ 怪物潮精灵切帧与GIF (anim.2d: mobAnim)
弹幕投射物与掉落物名单 (text.plain: lootNames, 9 枚经验水晶/弹幕/遗物/宝箱)
  ├─→ 弹幕与掉落宝石·3×3整版 (asset.image: lootSheet)
  └─→ 弹幕与掉落物切格透明打包 (image.iconPack: lootPack ← lootSheet)
爽快割草受击与宝石拾取音效 (asset.sfx: juicyHitSfx)
```

---

## 二、肉鸽割草游戏四大工程化铁律

1. **铁律 1 · 同屏百怪高辨识度剪影与原生透明直出**：
   - 肉鸽割草游戏同屏常有上百只怪物与密集弹幕，`heroRef`、`heroSheet`、`mobSheet` 必须采用加粗外轮廓、高明度对比与斜 45° 俯视（`animViewAngle: "iso"`），统一保持 `generateTransparentBg: true` 与 `animKeyColor: ""`。

2. **铁律 2 · 自动提取轻量化 2D 碰撞多边形与攻防判定框（`extractPhysicsHull: true`）**：
   - 同屏大量实体要求每个碰撞体顶点数 `<= 8`；
   - 对主角与怪物首帧调用 `game_tile_pbr_process({ assetId, extractPhysicsHull: true, maxHullVertices: 8, physicsPivot: "bottom_center" })`，自动产出 Godot `CollisionPolygon2D` / Unity `PolygonCollider2D` 顶点数组与 `hurtbox` / `hitbox`！

3. **铁律 3 · 高频受击/拾取音效 0ms 瞬发与防疲劳变体矩阵**：
   - 割草游戏中每秒触发十余次受击与拾取经验宝石音效，对 `juicyHitSfx` 调用 `game_audio_variants({ assetId, mode: "sfx_variants", count: 6, pitchSpreadSemitones: 2.4, trimSilence: true })`，获得清脆不刺耳的随机音高变体池。

4. **铁律 4 · 自动生成刷怪波次表与武器属性配表（`game_data_table_generate`）**：
   - 调用 `game_data_table_generate` 生成 `characters.csv`（主角与各阶怪物属性）、`skills.csv`（超武冷却与伤害倍率）、`waves.csv`（第 1~20 波刷怪间隔、数量与精英 Boss）及 `localization.csv`。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "roguelike-survivor-pack", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户的题材（如修仙御剑割草、赛博义体突围、萌宠幸存者）调用 `graph_edit` 修改各节点提示词与 `lootNames` 名单。
3. 调用 `task_run({ assetId })` 并发生成主角动画、怪潮动画、9 枚透明掉落物图标与割草音效。
4. 调用 `game_tile_pbr_process({ extractPhysicsHull: true })`、`game_audio_variants` 与 `game_data_table_generate` 完成物理、音效变体与数值表交付。

## 四、端口与参数速查

详见 `references/ports.md`。
