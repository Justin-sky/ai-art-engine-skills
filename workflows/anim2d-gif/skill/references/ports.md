# 节点拓扑与端口速查（anim2d-gif v2.3）

## 默认种子节点清单（共 4 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `charRef` | `asset.image` | 角色立绘基准（锚点，默认 `generateTransparentBg: true` 原生透明立绘） |
| `frameGen` | `frame.animGen` | 生成帧动画序列图（默认 `generateTransparentBg: true`, `animKeyColor: ""` 原生透明序列图） |
| `anim` | `anim.2d` | 2D帧动画（默认 `animKeyColor: ""` 零二次抠图，切格+稳像+合成透明 GIF） |
| `note` | `note.text` | 产品级 2D 帧动画工业管线说明 |

## 默认连线关系（共 2 条连线）

```
charRef:out → frameGen:in
frameGen:out → anim:in
```

## 2D 帧动画核心算子端口与参数全表

### 1. `frame.animGen`（生成帧动画序列图）
- **输入端口**：
  - `in`（`image`，多路输入）：接收角色基准立绘（`ai-sheet` / `video-restyle` 模式）或多张散帧（`frames-sheet` 模式）；
  - `in-video`（`video`）：接收动作参考视频或 I2V 生成的原地动作视频（`video-sheet` / `video-restyle` 模式）；
  - `in-text`（`text`）：接收外部动作描述文本。
- **输出端口**：
  - `out`（`image`）：生成的 `rows × cols` 标准序列图（Sprite Sheet）；
  - `out-all`（`images`）：图库历史序列图列表。
- **关键参数（`params`）**：
  - `animGenMode`: `"auto"` | `"ai-sheet"`（AI 角色生序列图）| `"video-sheet"`（视频均匀抽帧+抠底稳像直拼序列图，零生图成本）| `"video-restyle"`（视频抽帧姿态表+角色图引导 AI 严格重绘）| `"frames-sheet"`（多张散帧稳像拼表）；
  - `animRows` / `animCols`: 行数与列数（推荐黄金配比：`2×2` 配 `1:1` 画幅、`2×3` 配 `4:3` 画幅、`2×4` 配 `16:9` 画幅、`3×3` / `4×4` 配 `1:1` 画幅）；
  - `animPresetId`: 内置 20 种专业预设：
    - 移动类：`idle`（待机呼吸）、`walk`（行走循环）、`run`（奔跑循环）、`jump`（起跳腾空落地）、`dash`（高速冲刺闪避）；
    - 战斗类：`attack`（近战挥砍）、`combo`（多段连击）、`hurt`（受击硬直）、`skill`（必杀技爆发）、`death`（力竭倒地）；
    - 特效类（无人物）：`vfxSlash`（刀光剑气）、`vfxExplosion`（爆炸冲击波）、`vfxAura`（能量光环循环）；
    - 朝向与矩阵：`dir4`（4 方向正交站姿 `1×4`）、`dir8`（8 方向等距站姿 `2×4`）、`walk4Dir`（俯视角 RPG 4 方向行走矩阵 `4×4`）；
    - 表情与口型：`expr6`（AVG 6 格表情差分 `2×3`）、`lipSync4`（对话口型与眨眼 4 格差分 `2×2`）；
    - 地块瓦片：`tileSeamless`（无缝地砖 `2×2`）、`autotile3x3`（九宫格地形瓦片 `3×3`）；
  - `generateTransparentBg`: `true`（默认开启，直接生成原生 RGBA 透明序列图，无需下游二次抠图）；
  - `animViewAngle`: `"side"`（2D 横版正侧视）| `"iso"`（2.5D 斜 45° 等轴测）| `"front"`（正面平视）| `"top"`（90° 俯视）；
  - `animStyleLock`: `"cel"`（赛璐璐立绘）| `"pixel"`（16/32-bit 像素艺术）| `"hd2d"`（HD-2D 厚涂）| `"vfx"`（纯特效）；
  - `animKeyColor`: 默认设 `""`（不加纯色背景提示词、不执行抠图，直接依赖 `generateTransparentBg: true` 原生透明序列图；仅在 `video-sheet` 视频抽帧模式按需设 `"auto"`）；
  - `animStabilize`: `"ground"`（躯干主轴水平居中 + 跨帧身高归一化 + 脚底压基线）| `"center"`（质心居中）| `"off"`；
  - `animVideoStartRatio` / `animVideoEndRatio`: 视频抽帧起止区间占比（`0 ~ 1`）。

### 2. `anim.2d`（2D 帧动画切格、稳像与 GIF 合成）
- **输入端口**：
  - `in`（`image`）：接入 `frame.animGen:out` 或任意序列图；
  - `in-video`（`video`）：亦可直接连入视频实时抽帧切格。
- **输出端口**：
  - `out`（`image`）：首帧/当前选中单帧透明 PNG；
  - `out-all`（`images`）：切分并稳像后的全部单帧透明 PNG 序列（可直接导入 Unity / Godot / Unreal）；
  - `out-gif`（`image`）：当 `animGifFps > 0` 时合成落盘的循环透明 GIF 动图（可直连 `output.timeline:in-image`）。
- **关键参数（`params`）**：
  - `animRows` / `animCols`: 必须与上游序列图行列一致；
  - `animKeyColor`: 默认设 `""`（不执行二次抠图，直接保留上游透明序列图的 Alpha 通道；仅当上游为不透明视频/实底图时才按需设 `"auto"` | `"white"` | `"black"`）；
  - `animKeyThreshold`: 键控容差阈值（`5 ~ 180`，默认 `60`）；
  - `animEdgeInset`: `"auto"`（自动内缩约 1.5% 削去格线）或 `0 ~ 24` 像素；
  - `animStabilize`: `"ground"` | `"center"` | `"off"`；
  - `animPlayMode`: `"loop"`（正向循环）| `"pingpong"`（往返折返循环）；
  - `animDisabledCells`: 屏蔽跳过的坏帧坐标数组（如 `["2-4"]`）；
  - `animCellHolds`: 逐格停留时长倍率字典（如 `{"1-1": 1.5, "1-2": 0.5, "1-3": 2.5}`，支持挥砍顿帧 Hit-Stop、跳跃滞空等非均匀打击感律动，自动驱动实时预览、GIF89a 逐帧 `delayCs` 与引擎 `.atlas.json` 的逐帧 `duration`）；
  - `animCellOffsets`: 单帧像素级微调与镜像翻转字典（如 `{"1-2": {"dx": 4, "dy": -2, "flipX": false}}`）；
  - `animOutline`: 精灵自动外描边（`""` 关闭 | `"black"` 1px 深黑描边 | `"white"` 纯白描边 | `"gold"` 金色高光描边 | `"cyan"` 青色霓虹描边）；
  - `animPixelSnap`: 真像素晶格规整（`0` 关闭 | `2` / `3` / `4` / `6` px 像素块重采样与硬边缘二值化，彻底消除 AI 伪像素画的混杂像素 Mixels 与抗锯齿毛边）；
  - `animPaletteLock`: 跨帧调色板锁定与限色量化（`0` 关闭 | `128` 跨帧首帧色彩防闪烁 | `64` / `32` / `16` 色正统像素索引调色板）；
  - `animMotionTrail`: 动作残影 / 刀光幻影拖尾（`""` 关闭 | `"cyan"` 恶魔城式霓虹青残影 | `"gold"` 疾风金闪 | `"purple"` 暗影紫魅 | `"crimson"` 狂暴绯红 | `"echo"` 原色残像）；
  - `animLoopBlend`: `boolean`（首尾循环无缝平滑闭合：对循环尾帧向首帧做质心与核心区柔化收敛，消除 `Frame N → Frame 1` 跳帧顿挫）；
  - `animGroundShadow`: `boolean`（是否在角色脚底基线自动合成半透明接触椭圆阴影，采用底层 Destination-Over 合成不污染角色像素）；
  - `animGifFps`: GIF 基准帧率（`1 ~ 24`，默认建议 `8 ~ 12`；`0` 表示只切 PNG 帧不合成 GIF）。

### 3. 关联协同算子
- `asset.video`（用于通道 B 视频抽帧精灵）：设 `generateFrameMode: "first"` 时输入口为 `in-first-frame`（接 `charRef:out`），输出 `out`（接 `frameGen:in-video`）。
- `image.align` → `stage.2d`（用于通道 C 2D 骨骼动画）：`image.align:out` 接 `stage.2d:in`；`stage.2d` 输出 `out`（舞台单帧）、`out-frames`（骨骼动作帧序列）、`out-sheet`（烘焙后的雪碧图，可接 `anim.2d:in`）。
