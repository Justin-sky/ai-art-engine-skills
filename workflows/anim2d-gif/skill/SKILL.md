---
name: wf-anim2d-gif
description: 用「2D 动画套装」工作流从一段角色设定出发，先生成 2D 游戏角色基准锚点图（in-image 锁定头身比、色板与描边风格），再一次性并发产出「待机呼吸循环（Idle）」「奔跑移动循环（Run）」「近战/技能攻击（Attack）」三套标准 3×3 序列帧雪碧图与去底透明 GIF（anim.2d）。
workflow: anim2d-gif
workflow-version: 2.0.0
argument-hint: "[2D角色设定/画风（像素风/赛璐璐/Q版脊椎动画）/动作需求]"
when-to-use: 用户需要制作 2D 游戏角色序列帧动画、Sprite Sheet 雪碧图、透明背景 GIF 动图或一套完整的待机/奔跑/攻击游戏动作资产包时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 2D 动画套装 · 角色锚点锁死三动作序列帧与透明 GIF SOP（anim2d-gif v2.0）

## 拓扑架构与交付物

```
2D角色与动作设定 (text.plain)
  └─out→in-text──→ 2D角色基准锚点图 (asset.image, 纯白底侧面标准站姿)
                     ├─out→in-image──→ ① 待机呼吸动画·Idle (anim.2d, 3×3 序列帧 + 透明GIF, 8fps)
                     ├─out→in-image──→ ② 奔跑循环动画·Run (anim.2d, 3×3 序列帧 + 透明GIF, 10fps)
                     └─out→in-image──→ ③ 技能攻击动画·Attack (anim.2d, 3×3 序列帧 + 透明GIF, 12fps)
```

## 为什么必须先過「2D 角色基准锚点图」

游戏引擎里的同一个角色在切换 `Idle` / `Run` / `Attack` 状态时，头身比、描边粗细、服装色板与朝向必须完全一致。本工作流 v2.0 先由 `charAnchor` 生成一张**纯白底、无地面阴影、标准正侧面游戏精灵基准图**，再通过 `in-image` 喂给三个 `anim.2d` 节点：
- 每个 `anim.2d` 节点在一次运行内自动完成：**九宫格连续动作雪碧图生成 → 网格精准切帧 → 智能背景色抠除 → 合成循环透明 GIF**；
- 每个 `anim.2d` 节点同时输出 `out`（透明 GIF）、`out-frames`（独立切好的单帧 PNG 序列）与 `out-sheet`（原始雪碧图），可直接拖入 Unity / Godot / Cocos 或画布预览。

## 序列帧提示词规范

1. **纯白底无阴影铁律**：在 `charAnchor` 与三个 `anim.2d` 的 `generateInstruction` 中必须强调 `纯白背景（#FFFFFF）、无地面投影、无刀光跨格溢出、每格角色脚底基准线对齐`，确保自动去底后边缘干净无白边。
2. **帧率分级**：待机呼吸建议 `animGifFps: 8`，奔跑循环建议 `animGifFps: 10`，攻击爆发建议 `animGifFps: 12`。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "anim2d-gif", name: "<角色名>·2D动作套装", generateAspectRatio: "1:1" })`。
2. `graph_read({ assetId, includeParams: true })` 获取节点 ID。
3. `graph_edit` 更新 `spec`（角色设定）及 `charAnchor`、`idleAnim`、`runAnim`、`attackAnim` 的具体动作描述。
4. `task_run({ assetId })` 一键并发产出三套完整序列帧与透明 GIF。

## 端口与参数速查

详见 `references/ports.md`。
