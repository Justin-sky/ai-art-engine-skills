---
name: wf-anim2d-gif
description: 用「2D 帧动画」工作流生成产品级 2D 游戏角色序列帧雪碧图（Sprite Sheet）与循环透明 GIF：角色立绘（charRef）与序列图（frameGen）直接生成原生透明 PNG（generateTransparentBg: true），后续切帧与 GIF 合成不加入二次抠图操作（animKeyColor: ""），内置躯干主轴 X 防抖、跨帧身高归一化与 PixelOver 级后处理，并可按需扩展待机/奔跑/攻击/特效全套游戏动作包。
workflow: anim2d-gif
workflow-version: 2.3.0
argument-hint: "[2D角色设定/画风（赛璐璐cel/像素风pixel/HD-2D/纯特效vfx）/视角（side横版/iso等轴测/front正面/top俯视）/动作需求]"
when-to-use: 用户需要制作 2D 游戏角色序列帧动画、Sprite Sheet 雪碧图、透明背景 GIF 动图、待机/行走/奔跑/攻击/受击/刀光特效游戏动作资产包或 Spine 2D 骨骼动作时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status
---

# 2D 帧动画 · 产品级序列帧与透明 GIF 工业管线 SOP（anim2d-gif v2.3）

## 一、默认种子拓扑与交付物（Seed Skeleton，4 节点 / 2 连线）

```
角色立绘基准·锚点 (asset.image: charRef, 1:1 原生透明背景 generateTransparentBg: true + 正交平视视角)
  └─out→in──→ 生成帧动画序列图 (frame.animGen: frameGen, 原生透明序列图 generateTransparentBg: true + animKeyColor: "" + 视角/画风双锁)
                 └─out→in──→ 2D帧动画 (anim.2d: anim, 零二次抠图 animKeyColor: "" + 躯干主轴稳像 + 身高归一化 → out-gif / out-all)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "anim2d-gif", mode: "seed" })` 物化后，画布上的节点 `id` 与上方 `key` 完全一致（`charRef`、`frameGen`、`anim`、`note`），零冗余废节点，可直接用于 `graph_edit` 与 `task_run`。

---

## 二、产品级质量铁律（消除二次抠图挖穿盔甲、切格腰斩与躯干抽动）

1. **铁律 1 · 网格行列与画幅比例必须同构（严禁在 `1:1` 画布里排 `1×4`）**：
   - 每个单格的物理长宽比必须接近 `1:1`，与 `charRef`（`1:1`）保持一致，防止生图模型擅自改行列导致切格把角色拦腰截断：
     - **4 帧动作（轻量待机 / 4 步循环 / 表情差分）**：设 `animRows: 2, animCols: 2, generateAspectRatio: "1:1"`；
     - **6 帧动作（跳跃 / 挥砍 / 6 表情）**：设 `animRows: 2, animCols: 3, generateAspectRatio: "4:3"`；
     - **8 帧动作（细腻奔跑 / 连击 / 8 方向站姿）**：设 `animRows: 2, animCols: 4, generateAspectRatio: "16:9"`；
     - **9 / 16 帧矩阵（4 方向行走 `walk4Dir` / 地形九宫格 `autotile3x3`）**：设 `animRows: 3, animCols: 3` 或 `4×4`，`generateAspectRatio: "1:1"`。
   - **`frameGen` 与下游 `anim` 的 `animRows` / `animCols` 必须严格相等**。

2. **铁律 2 · 原生透明直出与严禁二次抠图（保护银白盔甲/高光/服饰细节）**：
   - **角色立绘（`charRef`）与序列图（`frameGen`）直接生成原生透明图**：统一保持 `generateTransparentBg: true`，提示词写明 `原生透明背景（RGBA 透明通道），无地面阴影且严禁绘制棋盘格纹理`；
   - **后续严禁加入抠图操作**：`frameGen` 与下游 `anim`（`anim.2d`）的 `animKeyColor` 必须统一设为空字符串 `""`（不开启键控抠图），并且**严禁在画布中插入 `image.cutout` / `image.matte` / `image.process(cutout)` 节点**，防止二次抠图误把角色身上的银白盔甲、眼白或浅色服饰挖穿成透明孔洞！
   - `animViewAngle`: `"side"`（2D 横版正侧视）| `"iso"`（2.5D 斜 45° 等轴测）| `"front"`（正面平视）| `"top"`（90° 俯视）；
   - `animStyleLock`: `"cel"`（Spine 赛璐璐立绘风）| `"pixel"`（16/32-bit 正统像素风）| `"hd2d"`（香草社厚涂风）| `"vfx"`（无人物纯刀光/爆炸特效）。

3. **铁律 3 · `anim.2d` 稳像、PixelOver 级后处理与打击感律动规范**：
   - 横版/等轴测角色动作设 `animStabilize: "ground"`（底层自动执行**躯干主轴水平质心对齐** + **跨格中位身高尺度归一化** + **脚底基线锁定**，挥剑/踢腿时身体绝不反向抽动）；
   - 飞行物、纯特效（VFX）或俯视角旋转设 `animStabilize: "center"`；
   - **PixelOver 级精灵后处理滤镜**：像素风（`pixel`）推荐开启 `animPixelSnap: 2`（或 `4`，真像素晶格规整消除 Mixels 毛边）、`animPaletteLock: 32`（32色定色防闪烁）与 `animOutline: "black"`（1px 锐利黑边）；冲刺/连招可开启 `animMotionTrail: "cyan"`（幻影拖尾）；横版/RPG 角色可开启 `animGroundShadow: true`（脚底贴地椭圆阴影）与 `animLoopBlend: true`（首尾循环无缝闭合）；
   - **打击感顿帧律动（Hit-Stop Timing）**：攻击动作可配置 `animCellHolds`（如蓄力帧 `1.5x`、挥砍帧 `0.5x`、命中定格帧 `2.5x`），使导出的 GIF 与 `.atlas.json` 具备真正的街机打击节奏；
   - 往返呼吸/蓄力动作可设 `animPlayMode: "pingpong"`（首尾自动无缝折返闭环），步态循环设 `animPlayMode: "loop"`，`animGifFps` 设为 `8 ~ 12`。

---

## 三、三大产品级生成通道（按场景择优编排）

### 通道 A：黄金网格 AI 直出（默认种子，`animGenMode: "ai-sheet"`）
- **适用**：快速产出 4 帧（`2×2`）或 8 帧（`2×4`）角色待机、行走、攻击、表情差分、4/8 方向转身表、纯刀光特效。
- **扩展多动作并行包（Idle + Run + Attack + Slash VFX）**：
  保留唯一 `charRef` 锚点，用 `graph_edit` 新增并行分支：
  - `frameGen_idle`（`animPresetId: "idle"`, `2×2`, `animPlayMode: "pingpong"`）→ `anim_idle`（`animGifFps: 8`）
  - `frameGen_run`（`animPresetId: "run"`, `2×4`, `generateAspectRatio: "16:9"`）→ `anim_run`（`animGifFps: 12`）
  - `frameGen_atk`（`animPresetId: "attack"`, `2×3`, `generateAspectRatio: "4:3"`）→ `anim_atk`（`animGifFps: 12`）
  全部从 `charRef:out` 接入各自 `frameGen_*:in`，一次并发跑齐整套动作资产。

### 通道 B：I2V 图生视频时序抽帧管线（高流畅度推荐，`animGenMode: "video-sheet"`）
- **为什么用它**：图像模型单次直出 8~16 格容易出现手脚顺拐或帧间跳变；而视频大模型具备天然的时序连续运动先验！
- **拓扑升级方式**：
  在 `charRef` 与 `frameGen` 之间用 `graph_edit` 插入一个 `asset.video` 节点（`key: "motionVid"`）：
  1. `motionVid` 参数：`generateFrameMode: "first"`（将 `charRef:out` 连入 `motionVid:in-first-frame`），`generateAspectRatio: "1:1"`，`generateAudio: false`，`generateInstruction: "固定正交侧视机位（Camera Locked），纯白无阴影干净背景（#FFFFFF），角色在原地完成流畅的【奔跑/挥剑连击】循环动作，身体不移出画面，动作干净利落"`；
  2. 将 `motionVid:out` 连入 `frameGen:in-video`，将 `frameGen` 的 `animGenMode` 设为 `"video-sheet"`（或同时保留 `charRef:out → frameGen:in` 并设 `animGenMode: "video-restyle"` 做 AI 姿态重绘），`animRows: 2, animCols: 4`，`animKeyColor: "auto"`，`animStabilize: "ground"`；
  3. `frameGen` 会自动从视频均匀抽取 8 帧、泛洪抠底、按躯干主轴与中位身高稳像拼成标准序列图，再由 `anim.2d` 切出透明 PNG 序列帧与丝滑 GIF！

### 通道 C：2D 骨骼绑定与 Spine 工业管线（零像素闪烁，`stage.2d`）
- **适用**：要求多帧 100% 零件像素零变化、支持导出 Spine 2D JSON 骨骼工程或丝滑骨骼插值动画。
- **拓扑升级方式**：
  `charRef`（`asset.image`）→ `align`（`image.align`，精灵去底规整）→ `stage`（`stage.2d`，自动关节检测 + 骨骼蒙皮 + 预设动作采样，从 `out-frames` / `out-sheet` 输出序列帧，或直接接入 `anim.2d:in` 合成 GIF）。

---

## 四、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "anim2d-gif", mode: "seed" })` 在当前画布加载种子节点（若在已有画布叠加则用 `mode: "merge"`）。
2. 根据用户所需的画风、视角、帧数与通道（通道 A 黄金网格 / 通道 B 视频时序抽帧 / 通道 C 骨骼绑定），用 `graph_edit` 更新 `charRef`、`frameGen`、`anim` 的 `params` 并按需增删节点。
3. 调用 `task_run({ assetId })` 执行生成，通过 `task_status` 跟踪进度，完成后汇报序列图、逐帧透明 PNG 与循环 GIF 动图产物（提示用户双击 `anim.2d` 可进入全屏帧动画工作台：支持前后双色洋葱皮 `🧅`、攻防双判定框 `HURTBOX/HITBOX` `🎯`、2D 法线贴图与实时动态点光源测试 `💡`、打击感顿帧节奏预设 `⚔️/🦘/💥`、跨帧调色板防闪锁定、恶魔城式残影拖尾、前后邻帧插值坏帧修复 `✨` 以及导出配套 `*-normal.png` + `.atlas.json` 引擎资产包）。

## 五、端口与参数速查

详见 `references/ports.md`。
