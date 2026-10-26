---
name: wf-game-vfx-sheet
description: 用「2D 游戏特效序列帧」工作流生成动作/RPG/卡牌/肉鸽/弹幕游戏的原生透明刀光剑气、魔法爆炸、受击火花、持续法阵光环序列图（VFX Flipbook Sprite Sheet）与循环透明 GIF，并同步生成配套技能打击音效与防疲劳音效变体。
workflow: game-vfx-sheet
workflow-version: 1.0.0
argument-hint: "[特效类型（刀光vfxSlash/爆炸vfxExplosion/光环vfxAura/弹幕/受击火花）/元素属性（雷电/烈焰/冰霜/暗影/圣光）/帧数（4帧/8帧/9帧）]"
when-to-use: 用户需要制作 2D 游戏特效序列帧、刀光剑气、技能爆炸 Flipbook、魔法光环、受击火花透明雪碧图或配套打击音效时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_audio_variants, game_bundle_export
---

# 2D 游戏特效序列帧 · 产品级 VFX Flipbook 工业管线 SOP（game-vfx-sheet v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，4 节点 / 1 连线）

```
生成VFX特效序列图 (frame.animGen: vfxSheet, 2×4 原生透明背景 generateTransparentBg: true + animStyleLock: "vfx")
  └─out→in──→ 特效切格与GIF合成 (anim.2d: vfxAnim, animKeyColor: "" + animStabilize: "center" → 透明单帧序列 + GIF)
配套技能打击音效 (asset.sfx: vfxSfx → 可接 game_audio_variants 生成防听觉疲劳随机音高变体组)
```

---

## 二、游戏 VFX 特效序列图三大黄金铁律

1. **铁律 1 · 纯特效无人物约束与原生透明直出**：
   - `vfxSheet`（`frame.animGen`）必须保持 `generateTransparentBg: true`、`animKeyColor: ""`、`animStyleLock: "vfx"`；
   - 提示词必须显式声明 **`纯视觉特效（VFX Only），画面中严禁出现任何人物、角色肢体或武器柄实体，原生透明背景（RGBA 透明通道），严禁绘制棋盘格纹理`**；
   - 下游 `vfxAnim`（`anim.2d`）同样保持 `animKeyColor: ""` 零二次抠图，避免高光与半透明粒子辉光被误抠。

2. **铁律 2 · 完整特效生命周期设计（起手 → 爆发 → 消散）**：
   - **瞬发型特效（刀光 `vfxSlash` / 爆炸 `vfxExplosion` / 受击火花）**：
     - `#1~#2`：能量聚拢 / 闪光起手（Anticipation）；
     - `#3~#4`：全盛爆发 / 最大张力帧（Climax Impact，可配合 `animCellHolds` 设顿帧 `1.5x`）；
     - `#5~#8`：冲击波扩散、余烬与电弧粒子逐帧衰减消散（Dissipation），确保特效结尾干净不突兀截断。
   - **持续型特效（法阵 / 护盾 / 火焰光环 `vfxAura`）**：
     - 设 `animPlayMode: "loop"` 且开启 `animLoopBlend: true`，首尾帧形态平滑闭环。

3. **铁律 3 · 质心稳像与音效防疲劳**：
   - 特效节点 `vfxSheet` 与 `vfxAnim` 的 `animStabilize` 均设为 `"center"`（锁定爆发中心点不漂移）；
   - 当 `vfxSfx` 生成完成后，可调用 `game_audio_variants({ assetId, mode: "sfx_variants", count: 4, trimSilence: true })` 裁除首部静音并产出 4 组微音高随机变体，满足高频连击音效需求。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "game-vfx-sheet", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户需求（如烈焰刀光、雷电落雷、冰霜碎裂、治疗法阵）调用 `graph_edit` 更新 `vfxSheet`、`vfxAnim`、`vfxSfx` 的参数（若需多套特效可并联新增 `vfxSheet_2 → vfxAnim_2`）。
3. 调用 `task_run({ assetId })` 并发渲染特效序列图、切格 GIF 与配套打击音效；若需音效随机变体则追加调用 `game_audio_variants`。

## 四、端口与参数速查

详见 `references/ports.md`。
