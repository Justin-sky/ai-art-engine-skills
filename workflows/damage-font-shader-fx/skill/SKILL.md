---
name: wf-damage-font-shader-fx
description: 用「暴击美术跳字与Shader特效套件」工作流一键生成 4×4 战斗暴击跳字美术字图集（0-9、+-!%CM）、导出标准 AngelCode BMFont (.fnt) 位图字体文件、生成无缝溶解噪声贴图与暴击重击音效，并配合 game_shader_particle_generate 一键生成跨引擎（Godot 4 .gdshader / Unity .shader / WebGL GLSL）8 大核心游戏 Shader（受击闪白、余烬溶解、像素外描边、水面波纹、复古CRT、全息故障、植被摇曳、传奇流光）与 8 大运行时 GPU 粒子预设。
workflow: damage-font-shader-fx
workflow-version: 1.0.0
argument-hint: "[跳字美术风格（烈焰金属/冰晶霓虹/像素街机/暗黑血刃）/所需Shader与粒子特效类型]"
when-to-use: 用户需要制作游戏战斗伤害跳字（暴击数字位图字体 BMFont）、游戏 Shader（受击闪白/死亡溶解/高亮描边/水面/CRT滤镜）或 GPU 粒子发射器参数配置时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_tile_pbr_process, game_audio_variants, game_shader_particle_generate, game_systems_graph_generate, game_bundle_export
---

# 暴击美术跳字与Shader特效套件 · 工业管线 SOP（damage-font-shader-fx v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，6 节点 / 3 连线）

```
暴击跳字16字符表 (text.plain: glyphNames, 0-9 + +-!%CM)
  ├─→ 暴击跳字艺术字体·4×4整版 (asset.image: fontSheet, 1:1 16格字符表)
  └─→ 暴击跳字切格透明打包 (image.iconPack: fontPack ← fontSheet)
Shader余烬溶解无缝噪声遮罩图 (asset.image: dissolveNoise, 1:1 灰度Perlin噪声)
刀刀烈火暴击重击音效 (asset.sfx: critHitSfx)
```

---

## 二、游戏打击感表现层三大工程化铁律

1. **铁律 1 · 艺术暴击跳字必须同步产出 AngelCode BMFont (`.fnt`) 度量文件**：
   - 仅有数字图片无法直接在引擎 `Label` 中按字符串渲染 `-9999!`；
   - 调用 `game_systems_graph_generate({ systemName: "combat_ui", bitmapFont: { fontName: "damage_crit_font", rows: 4, cols: 4, charset: "0123456789+-!%CM" } })`，自动导出标准 `.fnt` 与 JSON 度量表。

2. **铁律 2 · 高频状态表现优先使用跨引擎实时 Shader（`game_shader_particle_generate`）**：
   - 受击闪白（`hit_flash`）、怪物死亡余烬溶解（`dissolve_burn`）、交互高亮描边（`pixel_outline`）与传奇卡牌扫光（`rarity_sheen`）不应為每个角色重复画序列帧，而应调用 `game_shader_particle_generate` 一键导出 Godot 4 `.gdshader`、Unity `.shader` 与 WebGL `.frag.glsl`。

3. **铁律 3 · 溶解遮罩贴图必须无缝四方连续（`makeSeamless: true`）**：
   - 对 `dissolveNoise` 节点产物调用 `game_tile_pbr_process({ assetId, makeSeamless: true })`，确保任意 UV 滚动或缩放采样时无接缝断裂。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "damage-font-shader-fx", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户的游戏风格调整 `fontSheet` 提示词并调用 `task_run` 并发生成跳字图集、溶解噪声贴图与暴击音效。
3. 调用 `game_shader_particle_generate` 导出全套跨引擎 Shader 与 GPU 粒子发射器配置。
4. 调用 `game_systems_graph_generate` 导出 `damage_crit_font.fnt` 位图字体描述文件。

## 四、端口与参数速查

详见 `references/ports.md`。
