# 端口与参数速查（anim2d-gif 及 2D 节点族）

字段名与端口 id 取自应用源码，可直接写进 `graph_edit` 或 `workflow_commit` 的 `params`。**参数名拼错会被当作"未声明参数"丢弃**，并在物化的 warnings 里列出 —— 先看 warnings。

## anim.2d（2D帧动画，核心）

双击卡片可进入**全屏帧动画工作台**（或点击「进入生成内图」下潜到含 `frame.animGen` 的子图）。

| 端口 | 界面标签 | 方向 | 类型 | 说明 |
|---|---|---|---|---|
| `in` | 输入 | in | `image`（`multiple: false`） | 序列图（sprite sheet），**只接一条** |
| `in-video` | 视频 | in | `video`（`multiple: false`） | 动作视频（当 `in` 未接序列图时，直接从视频均匀抽帧 + 抠底稳像拼成序列图再切格） |
| `out` | 帧 | out | `image` | 当前选中帧 PNG |
| `out-all` | 全部帧 | out | `images` | 本次切出的全部有效帧 PNG（已剔除 `animDisabledCells`） |
| `out-gif` | GIF | out | `image` | 运行后合成的 GIF 动图（`animGifFps > 0` 时产出），并落盘为工程资产 |

声明参数（`defaultParams()` 声明，物化与导出时保留）：

| 参数 | 取值 / 默认 | 说明 |
|---|---|---|
| `animRows` | `1`~`6`，默认 `1` | 切格行数；超出范围会被 clamp |
| `animCols` | `1`~`6`，默认 `4` | 切格列数 |
| `animPresetId` | `idle` / `walk` / `run` / `jump` / `attack` / `hurt` / `skill` / `cast` / `death` / `victory` / `dash` / `climb` / `vfx_fire` / `vfx_slash` / `vfx_explosion` / `vfx_magic` / `vfx_lightning` / `vfx_smoke`，默认 `walk` | 角色动作或视觉特效预设 id |
| `animInstruction` | 字符串，默认 `''` | 自定义动作描述（为空时回退 preset 的 prompt） |
| `animGifFps` | `0`（关闭）~ `24`，节点默认 `0`（预设 `12`） | **>0 才合成 GIF**；非数字 / ≤0 → 关闭 |
| `animKeyColor` | `''` / `'auto'` / `'black'` / `'white'`，默认 `''`（预设 `'auto'`） | 背景键控透明：`auto` 自动采样每格四角背景色并去白边；`black` 黑底；`white` 白底 |
| `animKeyThreshold` | `5`~`180`，默认 `60` | 背景键控容差阈值 |
| `animEdgeInset` | `'auto'` 或 `0`~`24`（px），默认 `'auto'` | 切格边缘内缩像素，避开分格线与邻格渗色 |
| `animStabilize` | `'off'` / `'ground'` / `'center'`，默认 `'off'`（预设 `'ground'`） | 帧间原地稳像：`ground` 脚底基线对齐（防跑步/行走上下左右漂移）、`center` 主体质心居中 |
| `animPlayMode` | `'loop'` / `'pingpong'`，默认 `'loop'` | 循环模式：`loop` 正向循环、`pingpong` 往返循环（导出 GIF 同步生效） |
| `animDisabledCells` | 字符串数组（如 `["1-2", "1-4"]`），默认 `[]` | 跳过/剔除的坏帧格子坐标（`1-based` 行-列） |
| `animGenMode` | `'auto'` / `'video-sheet'` / `'video-restyle'` / `'frames-sheet'` / `'ai-sheet'` | 视频输入或内图生成模式 |
| `animVideoStartRatio` | `0`~`0.95`，默认 `0` | 视频抽帧起始区间比例 |
| `animVideoEndRatio` | `0.05`~`1`，默认 `1` | 视频抽帧结束区间比例（须 ≥ `startRatio + 0.05`） |
| `animAssetId` | 资产 id，默认 `''` | 内图资产 id（本工程引用，导出市场包时自动剥离） |
| `animGraphVersion` | 数字 | 内图结构版本 |
| `text` | 字符串 | 备注文本 |
| `mediaOutputDir` | 字符串 | 帧 / GIF 的落盘目录 |

运行时写回字段（不要手工伪造）：`animGridImage`、`animGifRelativePath`、`animGifFrameCount`、`animGifWidth`、`animGifHeight`。

---

## frame.animGen（生成帧动画序列图，内图或独立节点）

| 端口 | 界面标签 | 方向 | 类型 | 说明 |
|---|---|---|---|---|
| `in` | 输入 | in | `image`（`multiple: true`） | 角色参考图（单张）或散帧图序列（多张，用于 `frames-sheet`） |
| `in-video` | 视频 | in | `video`（`multiple: false`） | 动作参考视频（用于 `video-sheet` 直转或 `video-restyle` 姿态锁定 AI 重绘） |
| `out` | 序列图 | out | `image` | 当前选中序列图（sprite sheet） |
| `out-all` | 全部序列图 | out | `images` | 历次生成的序列图历史 |

关键参数：`animRows`（1~6）、`animCols`（1~6）、`animPresetId`、`animKeyColor`（`''` / `'auto'` / `'black'` / `'white'`）、`animViewAngle`（`''` / `'side'` / `'iso'` / `'front'` / `'top'`）、`animStyleLock`（`''` / `'cel'` / `'pixel'` / `'hd2d'` / `'vfx'`）、`animGenMode`（`'auto'` / `'video-sheet'` / `'video-restyle'` / `'frames-sheet'` / `'ai-sheet'`）、`animVideoStartRatio`（0~0.95）、`animVideoEndRatio`（0.05~1）、`animStabilize`（`'off'` / `'ground'` / `'center'`）。

---

## 同族 2D 节点端口速查（image.align / stage.2d）

### `image.align`（精灵对齐）
- **端口**：`in`（`image`）→ `out`（`对齐图`，`image`，单输出端口，每次运行原地更新对齐结果）。
- **参数**（写在 `params.imageAlign`）：`canvasWidth`、`canvasHeight`、`anchor`（`'ground'` / `'center'`）、`contentHeightRatio`（0.1~1）、`groundRatio`（0~0.5）、`fitWithinWidth`（boolean）、`offsetX` / `offsetY`（px）、`flipX`（水平镜像）、`bgKey`（`''` / `'auto'` / `'black'` / `'white'`）、`bgThreshold`（5~180）、`defringe`（去白边）、`tileSeamless`（瓦片四边无缝平铺）、`tileBlendRatio`（0.05~0.35）、`nineSlice`（九宫格边距 px）。

### `stage.2d`（2D 舞台）
- **端口**：`in`（`image`，`multiple: true`）→ `out`（`舞台图`，单帧合成 PNG）、`out-frames`（`动作帧`，`images`，`stage2dAnimFps > 0` 且带动作时产出逐帧 PNG 序列）、`out-sheet`（`序列图`，`image`，拼版 sprite sheet PNG）。
- **参数**：`stage2dScene`（含图层 `blendMode` / `opacity` / `tint` / `parallaxX` / `parallaxY` / `parentId` / `pivotX` / `pivotY` 及 `stage2dAction` 动作轨）、`stage2dAnimFps`（0 或 6~24）。

---

## asset.image（动作序列图，上游）

| 端口 | 方向 | 类型 | 说明 |
|---|---|---|---|
| `in-text` | in | `text`（multiple） | 上游正文/提示词 |
| `in-image` | in | `image`（multiple） | 参考图（角色一致性、画风参考） |
| `out` | out | `image` | 当前选中图 |
| `out-all` | out | `images` | 全部历史 |

---

## 常见错误

| 现象 | 原因与排查 |
|---|---|
| `GRAPH_PROCESS_NO_INPUT` | `anim.2d` 的 `in` 与 `in-video` 都没接到有效媒体，且内图 `animAssetId` 也无序列图；先跑上游或接入视频/图片 |
| GIF 没产出，只有帧 PNG | `animGifFps = 0`（默认关闭）／有效帧数不足 2（检查 `animDisabledCells` 是否禁用了过多帧） |
| 角色走路/跑步在原地忽高忽低或左右乱晃 | 将 `animStabilize` 设为 `'ground'`（脚底对齐）并开启 `animKeyColor: 'auto'` |
| 切出来的帧四边有白边或背景色残留 | 将 `animKeyColor` 设为 `'auto'`，并适当调高 `animKeyThreshold`（如 60~90）或调大 `animEdgeInset` |
