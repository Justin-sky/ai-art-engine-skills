---
name: wf-anim2d-gif
description: 用「2D 帧动画」工作流把一张动作序列图（sprite sheet）或一段动作视频切分成透明原地稳像的逐帧 PNG 并合成为 GIF 动图。当用户要角色行走/奔跑/待机/攻击等 2D 帧动画、想把视频抽帧转为序列图/GIF、或已有一张分格序列图要拆帧时加载本技能。
workflow: anim2d-gif
workflow-version: 1.1.0
---

# 2D 帧动画（anim2d-gif）

## 这条工作流产出什么

`动作序列图（in）或动作视频（in-video） → 2D帧动画节点（自动抠底 + 原地稳像） → 逐帧透明 PNG + GIF 动图`

- **逐帧 PNG** 从节点的 `out`（`帧`，当前选中帧）与 `out-all`（`全部帧`，逐帧序列）两个端口输出。
- **GIF 动图** 从 `out-gif`（`GIF`）端口输出，并且**落盘为工程资产**（相对路径写在节点的 `animGifRelativePath`）。
- **双击「2D帧动画」节点**可进入**全屏帧动画工作台**（实时切格预览、洋葱皮残影、逐帧条带坏帧开关、视频一键抽帧拼合、导出序列图/GIF，或一键进入生成内图）。

## 什么时候用 / 什么时候别用

用：
- 用户要**像素 / 赛璐璐 / HD-2D 角色或特效的逐帧循环动画**（行走、奔跑、待机、跳跃、攻击、受击、放技能、火焰/爆炸/斩击等）。
- 手里已有一张 sprite sheet 要**切格 + 去背 + 脚底稳像 + 合成 GIF**。
- 手里有一段动作视频，想**均匀抽帧转为 2D 序列图 + 逐帧 PNG + GIF**（解决纯提示词生序列图姿态不可控问题）。

别用（用兄弟工作流）：

| 用户其实想要 | 该用 |
|---|---|
| 一张图直接变成一段镜头视频 | `storyboard-video`（分镜出片） |
| 剧本/分镜驱动的整条买量或短剧视频 | `game-ua-video`（游戏买量）、`short-drama9`（短剧分镜·9格直出） |
| 角色多视角设定立绘 | `character-sheet`（角色设定） |
| 游戏图标整版切格打包 | `game-icons`（游戏图标包，走 `image.process` + `image.iconPack`） |

## 上游输入与 4 种可控生成路径

1. **直接给序列图（`in` 端口）**：把已有的 rows×cols 序列图接到 `anim.2d` 的 `in` 端口（`image`，`multiple: false`），或运行上游预设的 `asset.image` 节点生成 1×4 序列图。
2. **直接给动作视频（`in-video` 端口，最稳！）**：把 `asset.video` 接到 `anim.2d` 的 `in-video` 端口（或在生成内图里接 `frame.animGen` 的 `in-video`）：
   - **`video-sheet`（视频直转，零 AI 幻觉）**：在 `[animVideoStartRatio, animVideoEndRatio]` 区间均匀抽取 `rows × cols` 帧，自动抠底去白边并按脚底/质心原地稳像，直接拼合成标准序列图再切帧合成 GIF。
   - **`video-restyle`（视频姿态 + AI 换皮重绘）**：从视频抽取姿态网格表，连同角色设定图一起喂给图片模型，让 AI 严格锁死视频每帧的肢体动作重绘为目标角色。
3. **多张散帧拼合（`frames-sheet`）**：在内图 `frame.animGen` 的 `in` 端口接入多张独立帧（例如 `video.framePull` 逐帧拉片挑出的关键帧），自动对齐拼合为标准序列图。
4. **单图 AI 序列图（`ai-sheet`）**：单张角色参考图 + 动作预设（`animPresetId`）+ 视角锁定（`animViewAngle`）+ 画风锁定（`animStyleLock`）生成序列图。

## 关键约束与参数要点

1. **行列数与实际分格必须一致**：`animRows` × `animCols`（均 clamp 在 1~6）决定切格网格；若用视频直转（`in-video`），则决定从视频均匀抽取多少帧（`rows × cols` 帧）。
2. **自动抠底与容差（`animKeyColor` + `animKeyThreshold`）**：
   - `'auto'`（推荐默认）：自动采样每格四角背景色并键控抠除 + 边缘反混合去白边，适用 AI 常见的纯色底/灰底/立绘底；
   - `'black'` / `'white'`：黑底（发光特效最稳）或白底键控透明；
   - `''`：保留原背景不抠除（带完整场景的帧动画用 `''`）。
   - `animKeyThreshold`（5~180，默认 `60`）控制抠底容差。
3. **帧间原地稳像（`animStabilize`）**：
   - `'ground'`（角色动作推荐默认）：自动检测每帧主体 Alpha 包围盒，将脚底基线对齐且水平居中，彻底消除 AI 序列图或跑步视频中角色忽高忽低、左右漂移的抖动；
   - `'center'`（飞行物 / 特效推荐）：按每帧主体质心居中对齐；
   - `'off'`：保持原始切格位置。
4. **坏帧剔除与循环模式**：
   - `animDisabledCells`（如 `["1-3"]`）可跳过崩坏帧，不参与 `out-all` 与 GIF 合成；
   - `animPlayMode` 支持 `'loop'`（正向循环）与 `'pingpong'`（往返溜溜球循环，非常适合呼吸待机、蓄力、摇摆动作）。
5. **GIF 帧率与尺寸规则**：
   - `animGifFps`（0~24，预设 `12`）：**>0 才合成 GIF**；`0` 只切帧不出 GIF。
   - **GIF 长边上限为 512px**（等比缩放，只缩不放），**逐帧 PNG 保持原图完整分辨率**。

## 参数去哪查

`anim.2d`、`frame.animGen` 以及同属 2D 节点族的 `image.align`、`stage.2d` 的全部端口与参数速查见 `references/ports.md`。

