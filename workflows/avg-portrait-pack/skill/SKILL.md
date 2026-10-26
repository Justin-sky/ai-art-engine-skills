---
name: wf-avg-portrait-pack
description: 用「AVG/RPG 立绘与表情差分包」工作流生成视觉小说（Galgame / Ren'Py）、叙事解谜与 JRPG 对话系统的原生透明角色基准立绘、6格喜怒哀乐惊羞表情差分、4格说话口型与眨眼差分（Lip-Sync）、16:9 剧情舞台背景 CG 与角色台词配音。
workflow: avg-portrait-pack
workflow-version: 1.0.0
argument-hint: "[角色人设/外貌服饰/画风（日系二次元/韩系厚涂/国风古韵/暗黑哥特）/场景背景CG主题/代表台词]"
when-to-use: 用户需要制作 AVG 视觉小说、文字冒险、恋爱养成、互动叙事或 RPG 对话框立绘、角色喜怒哀乐表情差分、说话张嘴眨眼口型差分或配套背景 CG 时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_bundle_export
---

# AVG/RPG 立绘与表情差分包 · 工业管线 SOP（avg-portrait-pack v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，8 节点 / 4 连线）

```
角色基准半身立绘 (asset.image: charPortrait, 3:4 原生透明背景 generateTransparentBg: true)
  ├─out→in──→ 6格表情差分表 (frame.animGen: exprSheet, 2×3 expr6 喜怒哀乐惊羞)
  │              └─out→in──→ 表情差分切帧与对齐输出 (anim.2d: exprPack → 6 张同基准线透明表情立绘)
  └─out→in──→ 4格说话口型与眨眼差分 (frame.animGen: lipSheet, 2×2 lipSync4 闭口/半张/大张/眨眼)
                 └─out→in──→ 口型眨眼动态精灵 (anim.2d: lipAnim → 4 张口型帧 + 说话动画 GIF)
配套剧情舞台背景CG (asset.image: storyBg, 16:9 无人场景背景)
角色代表台词配音 (asset.voice: charVoice, 角色专属音色台词)
```

---

## 二、AVG 立绘与表情差分三大核心规范

1. **规范 1 · 原生透明直出与头肩像素级锁死**：
   - `charPortrait`、`exprSheet`、`lipSheet` 均开启 `generateTransparentBg: true`，`animKeyColor: ""`，严禁插入二次抠图节点；
   - `exprSheet` 与 `lipSheet` 必须连入 `charPortrait:out` 作为视觉锚点，提示词强调 **`除五官表情/口型变化外，发型、头饰、肩线与服装褶皱位置保持 100% 一致`**；
   - `exprPack` 与 `lipAnim`（`anim.2d`）开启 `animStabilize: "ground"`，自动将切出的每张差分图对齐到同一底边与水平质心，确保在 Ren'Py / Unity / Godot 对话框中切换表情时立绘绝不跳动。

2. **规范 2 · 服装差分扩展（Outfit Variants）**：
   - 若用户需要同一角色的多套服装差分（如日常便服 / 战斗铠甲 / 礼服 / 战损状态），可用 `graph_edit` 新增 `asset.image` 节点并将 `charPortrait:out` 连入其 `in-image` 端口，锁定面部五官与身材比例仅替换服装。

3. **规范 3 · 多状态背景 CG 扩展（昼 / 暮 / 夜场景差分）**：
   - 可将 `storyBg:out` 连入新增的 `asset.image:in-image`，一键派生同一场景的「清晨阳光」「黄昏夕照」「深夜烛火」三态背景差分。

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "avg-portrait-pack", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户设定的角色外貌、性格、场景与台词，调用 `graph_edit` 更新 `charPortrait`、`exprSheet`、`lipSheet`、`storyBg`、`charVoice` 的参数。
3. 调用 `task_run({ assetId })` 并发生成全套透明立绘、6 格表情差分、4 格口型差分、背景 CG 与角色语音。

## 四、端口与参数速查

详见 `references/ports.md`。
