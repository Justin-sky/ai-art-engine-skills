---
name: wf-game-audio-suite
description: 用「全品类游戏音频矩阵」工作流一键生成探索态与战斗态纯器乐 BGM、攻击挥砍/命中受击/UI确认音效与角色战斗语音，并通过本地 FFmpeg 算子（game_audio_variants）自动将 BGM 处理为相位连续无缝循环音轨、将 SFX 裁切首部静音实现 0ms 瞬发响应并派生多组防听觉疲劳音高变体。
workflow: game-audio-suite
workflow-version: 1.0.0
argument-hint: "[游戏类型与题材（奇幻RPG/赛博射击/像素肉鸽/休闲三消/恐怖解谜）/BGM情绪与BPM/所需SFX列表/角色语音台词]"
when-to-use: 用户需要制作游戏背景音乐（BGM 无缝循环）、战斗/脚步/技能/UI 音效（SFX 变体矩阵）或角色语音包时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_audio_variants, game_bundle_export
---

# 全品类游戏音频矩阵 · 工业管线 SOP（game-audio-suite v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，7 节点）

```
探索态/主城环境 BGM (asset.music: bgmExplore) ──→ game_audio_variants(mode: "bgm_loop") → 零接缝无限循环 BGM
高燃 Boss 战斗态 BGM (asset.music: bgmBattle) ──→ game_audio_variants(mode: "bgm_loop") → 零接缝无限循环 BGM
近战挥砍/武器攻击音效 (asset.sfx: sfxAttack) ───→ game_audio_variants(mode: "sfx_variants", count: 4) → 0ms瞬发+4变体
重击命中/护甲受击音效 (asset.sfx: sfxHit) ──────→ game_audio_variants(mode: "sfx_variants", count: 4) → 0ms瞬发+4变体
UI 按钮确认/卡牌出牌音效 (asset.sfx: sfxUiClick) ─→ game_audio_variants(mode: "sfx_variants", count: 4) → 0ms瞬发+4变体
角色必杀技/受击战斗语音 (asset.voice: battleVoice)
```

---

## 二、游戏音频工程化两大黄金铁律

1. **铁律 1 · BGM 必须纯器乐（Instrumental）且经过相位连续循环闭合（`mode: "bgm_loop"`）**：
   - `bgmExplore` 与 `bgmBattle` 的提示词必须声明 `instrumental loopable structure, steady BPM, no vocals`；
   - 生成完成后调用 `game_audio_variants({ assetId, mode: "bgm_loop", crossfadeSec: 2.5 })`，将音轨转化为首尾在原始 `t=2.5s` 处严格连续的无限循环文件（`_loop.mp3`）。

2. **铁律 2 · SFX 必须 0ms 瞬发响应且具备防听觉疲劳随机变体（`mode: "sfx_variants"`）**：
   - AI 直出的音效常带 `50~200ms` 首部静音，导致玩家按下攻击键后声音滞后；且同一音效连续播放 3 次以上会产生强烈机械感；
   - 生成完成后调用 `game_audio_variants({ assetId, mode: "sfx_variants", count: 4, trimSilence: true })`，自动切除首尾静音并派生 `v01 ~ v04` 四组微音高（Semitone）、语速与高频明暗度变体，供引擎 `AudioStreamRandomizer` 或 Unity 随机 AudioClip 池直接挂载！

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "game-audio-suite", mode: "seed" })` 在当前打开的画布上实例化音频矩阵节点。
2. 根据用户的游戏风格调整各 BGM、SFX 与角色语音节点的提示词（或并联扩展脚步声、跳跃声、宝箱开启声等节点）。
3. 调用 `task_run({ assetId })` 并发生成全部原始音频。
4. 对生成的 BGM 调用 `game_audio_variants({ mode: "bgm_loop" })`，对高频 SFX 调用 `game_audio_variants({ mode: "sfx_variants", count: 4 })`。

## 四、端口与参数速查

详见 `references/ports.md`。
