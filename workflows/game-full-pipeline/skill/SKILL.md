---
name: wf-game-full-pipeline
description: 用「全栈游戏设计与可玩实现」工作流引导用户完成游戏参数选择（品类/2D或3D视角/美术与UI画风/核心循环/数值规模），先创建完整经济系统与全套数值配表（game.economy + game_data_table_generate + game_systems_graph_generate），再驱动生成全套游戏系统策划案（asset.gameSystem）、拆解并生成多屏UI界面与PSD图层（ui.split → ui.gen → image.layerSplit），最后直接把策划案、全套数值配表与UI界面架构编译为可一键试玩的单文件HTML游戏（gameplay_prepare_project → 编写 src/** → gameplay_build + game_playable_preview_compile）。
workflow: game-full-pipeline
workflow-version: 1.0.0
allowed-tools: ask_user, workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_data_table_generate, game_systems_graph_generate, game_level_map_generate, game_playable_preview_compile, gameplay_prepare_project, gameplay_build, gameplay_job_status
---

# 全栈游戏设计与可玩实现（game-full-pipeline）

## 什么时候加载本技能

当用户提出以下任一需求时，**必须优先加载本技能**：
- 「帮我设计一款完整的游戏，包含所有策划案、UI界面、经济系统、全套数值，并直接生成可玩的 HTML 游戏」
- 「我想做个肉鸽/卡牌/RPG/塔防/AVG游戏，先让我选参数，再出全套 GDD 策划案、数值表、UI 界面和 HTML 试玩版」
- 「从零立项一款游戏，把经济数值系统、多屏 UI 和可运行 HTML 原型一次做全」

---

## 核心设计哲学：参数驱动 → 数值先行 → 策划与 UI 联动 → 代码级内联生成可玩 HTML

传统 AI 生成小游戏往往与策划案、数值表、UI 设计“两张皮”脱节（策划案写一套数值，HTML 游戏里硬编码另一套随机数字）。本技能强制执行**全栈同源闭环**：
1. **交互式选参立项（Parameter Wizard）**：开工前通过 `ask_user` 一次性确认核心品类、2D/3D 维度视角、美术与 UI 风格预设、经济数值复杂度与可玩 HTML 交付模式。
2. **经济系统与全套数值先行（`game.economy` + 结构化配表落盘）**：先运行 `game.economy` 完成 Machinations 资源动力学与蒙特卡洛（Monte Carlo）平衡模拟，并调用 `game_data_table_generate` 与 `game_systems_graph_generate` 产出全套 CSV/JSON 数值配表（角色、技能、卡牌、道具、波次、对白、技能树 DAG、Boss AI 状态机）。
3. **全套系统策划案生成（`asset.gameSystem`）**：`game.economy` 的 `out-text` 直接连入 `asset.gameSystem` 的 `in` 端口，自动解包上游数值与经济护栏，产出严丝合缝的全套系统策划案（GDD）。
4. **多屏 UI 界面拆分与生成（`ui.split` → `ui.gen` → `image.layerSplit`）**：从策划案自动拆出 6~12 屏 UI 界面规范（带字精修轨 + 空字底图轨），统一锁定 UI 材质风格并支持拆分为分层 PSD 与九宫格切图。
5. **直接生成同源 HTML 可玩游戏实现（Playable HTML Build）**：将第 2 步产出的**真实数值配表**、第 3 步的**核心玩法规则**与第 4 步的**多屏 UI 界面结构（主界面/战斗HUD/升级三选一/背包商店/结算页）**直接写进 `gameplay_prepare_project` 脚手架的 `src/**` 代码中，经 `gameplay_build` 打包并通过隐藏窗口 `smoke` 试玩门禁校验，直接向用户交付可一键点开畅玩的单文件 HTML 游戏！

---

## 四阶段标准作业程序（SOP）

### 阶段 ①：交互式立项向导（必须在建图或运行前先调用 `ask_user`）

当用户尚未明确全部核心参数时，**先调用一次 `ask_user`（3~4 个精炼问题）**收集游戏设计参数：

1. **核心玩法品类与视角维度**：
   - `俯视角动作肉鸽割草（Survivor / 2D Top-Down）`（对应 `gameSystemArchetype: "roguelike_survivor"`，`gameEconomyPreset: "survivor_xp_loop"`）
   - `回合制卡牌构筑（Deckbuilder DBG / 2D）`（对应 `gameSystemArchetype: "card_dbg_combat"`，`gameEconomyPreset: "roguelike_mana_deck"`）
   - `3D 动作冒险 / 地牢探索（Three.js 3D）`（对应 `gameSystemArchetype: "rpg_progression"`，`gameEconomyPreset: "arpg_equipment_economy"`，`gameplay_prepare_project` 选 `mode: "3d"`）
   - `横版平台跳跃 / 银河城（Platformer 2D Side-View）`（对应 `gameSystemArchetype: "rpg_progression"`，`gameLevelAlgorithm: "platformer_course"`）
   - `策略塔防 / 阵营战棋（Tower Defense & SLG）`（对应 `gameSystemArchetype: "slg_faction_intrigue"`，`gameEconomyPreset: "tower_defense_waves"`）
   - `悬疑/恋爱多分支视觉小说（AVG / Visual Novel）`（对应 `gameSystemArchetype: "avg_mystery_branch"`）
2. **美术与全套 UI 视觉风格预设（`uiGenStylePreset`）**：
   - `星轨白金幻想（celestial_star_gold，原神/星铁风）`
   - `怪盗红黑潮酷（persona_rebel_red，P5高对比漫画风）`
   - `极简战术终端（minimal_ark_tech，明日方舟机能风）`
   - `赛博全息霓虹（cyber_holo_grid，赛博朋克科幻风）`
   - `国风水墨仙侠（ink_paper_cel，东方修仙古风）`
   - `暗黑哥特羊皮纸（dark_iron_parchment，暗黑魂系地牢风）`
3. **经济系统与数值深度**：
   - `标准商业级数值（角色成长曲线 + 局内货币产出/商店回收 + 8~12 张卡牌/技能 + 10~15 波刷怪配表）`（推荐）
   - `深度硬核商业数值（额外包含抽卡保底概率模型 gacha_pity_banner + 完整技能树 DAG + 装备合成配方 + Boss AI 状态机）`
4. **UI 画幅与可玩 HTML 交付方式**：
   - `16:9 横屏 PC/主机端布局 + 完整多界面可玩 HTML 游戏（含主菜单/HUD/升级商店弹窗/结算）`（推荐）
   - `9:16 竖屏移动端布局 + 完整多界面可玩 HTML 游戏`

> 若用户在首句提示词中已给出了明确题材或参数，直接保留用户设定，仅为缺失维度补齐推荐默认值。

---

### 阶段 ②：铺设全栈工作流画布并先行构建经济与全套数值体系

1. **物化种子工作流**：
   调用 `workflow_use_installed({ id: "game-full-pipeline", mode: "seed" })` 在当前画布铺设 12 节点全栈拓扑：
   - `brief` (`text.plain`) ➔ `economy` (`game.economy`) + `gameSystem` (`asset.gameSystem`) + `heroSprite` (`asset.image`) + `enemySprite` (`asset.image`) + `levelMap` (`game.levelMap`)
   - `economy` (`out-text`) ➔ `gameSystem` (`in`)
   - `gameSystem` (`out`) ➔ `uiSplit` (`in`) ➔ `uiGen` (`in`) ➔ `uiLayerSplit` (`in`)
   - 配套音频节点：`bgmDsp` (`game.audioDsp`) 与 `sfxDsp` (`game.audioDsp`)

2. **写入立项总纲与经济模型参数（`graph_edit`）**：
   - 将用户选定的全部参数写入 `brief` (`text.plain`) 的 `params.text`。
   - 根据品类设置 `economy` (`game.economy`) 的 `gameEconomyPreset`（如 `survivor_xp_loop`、`roguelike_mana_deck`、`balatro_ante_jokers`、`arpg_equipment_economy`、`gacha_pity_banner`、`tower_defense_waves` 等）、`gameEconomySteps` 与 `gameEconomyMonteCarloRuns`。
   - 设置 `gameSystem` (`asset.gameSystem`) 的 `gameSystemArchetype` 与 `gameSystemTargetAspect`（`"16:9"` 或 `"9:16"`）。
   - 设置 `uiGen` (`ui.gen`) 的 `uiGenStylePreset` 与 `generateAspectRatio`。

3. **执行经济系统节点并生成全套工程级数值配表**：
   - 调用 `task_run({ assetId, nodeIds: ["economy", "levelMap", "bgmDsp", "sfxDsp"] })` 并用 `task_status` 等待完成。
   - 调用 `graph_read({ assetId, includeParams: true })` 读取 `economy` 产出的 `gameEconomyReport` 与数值诊断结论。
   - **调用 `game_data_table_generate`** 将全套结构化数值配表（`characters` 角色与怪物属性、`skills` 技能倍率与冷却、`items` 装备与遗物价格/掉落、`cards` 卡牌费用与攻防、`waves` 刷怪波次曲线、`dialogues` 剧情对白树）落盘为标准 CSV + JSON + 多语言 `localization.csv`（位于 `Assets/GameData/<tableName>/`）。
   - **调用 `game_systems_graph_generate`** 生成无环校验通过的天赋技能树（`skillTree`）、合成配方（`craftingRecipes`）、任务链（`quests`）与 Boss AI 有限状态机（`aiBehaviors` + Mermaid 状态图）。

---

### 阶段 ③：生成全套游戏系统策划案（GDD）与多屏 UI 界面

1. **运行全套策划案与 UI 拆分**：
   - 调用 `task_run({ assetId, nodeIds: ["gameSystem", "uiSplit"] })`（若用户同时需要生成画布上的主角/怪物精灵参考图，可把 `"heroSprite"`, `"enemySprite"` 一并加入 `nodeIds`）。
   - `gameSystem` (`asset.gameSystem`) 会自动读取上游 `brief` 与 `economy` (`out-text`) 的完整数值体系，生成包含**核心循环、3C 手感参数、伤害/经济公式、全系统模块设计、全量配表映射及 6~12 屏 UI 界面交互详单**的系统策划案。
   - `uiSplit` (`ui.split`) 自动将策划案拆解为结构化多屏 UI 清单（`uiScreens`：每屏包含 `id`、`title`、带字 `prompt` 与无字底图 `cleanPrompt`）。

2. **多屏 UI 界面生成与图层分离说明**：
   - 用 `graph_read({ assetId, includeParams: true })` 读取 `uiSplit` 拆出的界面列表并展示给用户。
   - 告知用户：双击画布上的 `uiGen`（「全套 UI 界面生成」）节点即可进入内图，逐屏生成高保真双轨 UI 界面图（带字精修版 + 空字底图版），回外层点 Cook 后即可流入下游 `uiLayerSplit`（「UI 图层分离与九宫格导出」）拆出 PSD 分层、9-Slice 九宫格拉伸框与多态按钮。

---

### 阶段 ④：直接生成可玩 HTML 游戏实现（全套数值 + 多屏 UI 完整落地）

在完成策划案、经济数值体系与 UI 拆分后，**不要停留在文档层面，必须立即为用户直接生成可玩的 HTML 游戏实现**：

#### 1. 准备 Node + esbuild 可玩游戏工程脚手架
调用 `gameplay_prepare_project({ mode: "2d" | "3d", title: "<游戏名称>" })`（2D 品类传 `"2d"`，3D 品类传 `"3d"`），获取 `projectRelativeDir`。

#### 2. 编写与策划案、数值表、UI 架构 100% 同源的 `src/**` 代码
使用文件写入工具编写工程代码，严格遵循以下**全栈同源规范**：
- **`src/core/rng.js` 与 `src/core/palette.js`**：
  - 严禁使用 `Math.random()`，全部使用 `core/rng.js` 的确定性种子随机函数。
  - 在 `core/palette.js` 中精确还原用户所选 `uiGenStylePreset` 的主题色板（如 `celestial_star_gold` 的深空靛蓝 `#0f172a`、星轨鎏金 `#f59e0b`、奥术青 `#38bdf8`）。
- **`src/game/dataTables.js`（内联全套真实数值与经济系统）**：
  - 将阶段 ② `economy` (`game.economy`) 与 `game_data_table_generate` 产出的**完整数值表原样内联为常量配置**：
    - 主角基础属性（HP、MP、ATK、DEF、SPD、暴击率、暴击伤害）与等级经验公式；
    - 全部技能 / 卡牌 / 武器的冷却、蓝耗/费用、伤害倍率、穿透/范围词条；
    - 全部普通怪、精英怪、阶段 Boss 的血量成长系数、移速、攻击间隔与掉落金币/经验权重；
    - 商店物品/遗物价格递增公式、刷新费用（Sink）、抽卡/三选一稀有度权重与保底计数器（Pity Gate）。
- **`src/assets/**`（程序化几何/贴图与 WebAudio 音效合成）**：
  - 每个美术与音效资产都通过 `defineAsset({ id, kind, build })` 注册，玩法侧通过 `buildAsset(id, ctx)` 获取。
  - 2D 角色/怪物/弹幕/地形贴图用 Canvas 2D 精心绘制（多层渐变、装甲高光、动态呼吸/行走帧、受击闪白与血条）；3D 模式用 Three.js `BufferGeometry` + Canvas 纹理 `MeshStandardMaterial` 组合建模。
  - 音效与 BGM 用 WebAudio（`OscillatorNode` + `BiquadFilterNode` + 增益包络或噪声合成）实现攻击、受击、暴击、拾取经验、升级、商店购买、按钮点击与背景音乐律动，禁止静音。
- **`src/game/uiScreens.js`（还原 `uiSplit` 拆出的全套游戏 UI 界面）**：
  - 严格按照阶段 ③ `gameSystem` 与 `uiSplit` 规划的多屏 UI 架构，在 HTML/Canvas 中实现完整可交互的多屏 UI 状态机：
    1. **主标题与开局选角/选难度屏（Title / Lobby Screen）**
    2. **局内战斗 HUD（Battle HUD）**：实时显示生命/护盾/经验进度条、波次/关卡倒计时、金币与核心资源池、底部技能/卡牌快捷栏（带 CD 遮罩）、右上小地图或 DPS 统计
    3. **升级三选一 / 技能树 / 卡牌构筑弹窗（Upgrade / Deck Modal）**：展示词条品质边框、数值加成详情与重随（Reroll）按钮
    4. **波间商店 / 遗物背包 / 装备合成面板（Shop & Inventory Screen）**：真实执行金币扣除、属性实时叠加与物品栏展示
    5. **战斗结算与数值复盘屏（Victory / GameOver Summary）**：展示生存时长、击杀数、总造成伤害、经济收支统计与「再来一局（R 键）」按钮
- **`index.template.html`**：
  - 保持内联经典脚本（保留 `/*APP_JS*/` 占位符，绝不加 `type="module"` 或外链 URL），确保打包后的 `dist/single.html` 在任何浏览器双击即可独立运行。

#### 3. 打包构建与 Smoke 试玩门禁自检（强制闭环）
1. 调用 `gameplay_build({ projectRelativeDir })` 触发宿主后台执行 `npm install` + `node build.mjs` 生成单文件 `dist/single.html`。
2. 调用 `gameplay_job_status({ jobId })` 轮询直到 `status === "done"` 或 `"error"`。
3. **检查试玩门禁报告 `smoke`**：
   - 若构建报错或 `smoke.ok === false`（存在未捕获异常、黑屏、零帧渲染等），**必须立即根据 `smoke.errors` 修改 `src/**` 代码并重新调用 `gameplay_build`**，绝不能把跑不起来的半成品交给用户！
   - 当 `status === "done"` 且 `smoke.ok === true` 时，应用会自动在资产库登记 `gamePlay` 资产并在对话流生成带**「试玩」**按钮的卡片。
4. **（可选双保险）同步编译画布精灵微缩沙盒**：
   - 若画布上的 `heroSprite`、`enemySprite` 已生图完成，可额外调用一次 `game_playable_preview_compile({ title, mode })` 将 AI 生成的角色/怪物精灵图内联编译到 `Assets/GamePlayable/` 下，供用户对比体验。

---

## 阶段总结交付模板

全部阶段执行完毕后，向用户输出一份清晰的全栈游戏交付总览：
1. **游戏立项与核心循环概览**（题材、品类、视角、UI 风格预设）。
2. **经济系统与全套数值配表清单**（`economy` 蒙特卡洛平衡结论 + 已落盘至 `Assets/GameData/` 的 CSV/JSON 配表路径与核心公式）。
3. **全套系统策划案（GDD）与多屏 UI 界面清单**（列出 `gameSystem` 节点与 `uiSplit` 拆出的 6~12 屏 UI 名称，提示用户可双击 `uiGen` 进入内图生成全套 UI 视觉稿并由 `uiLayerSplit` 导出分层 PSD）。
4. **HTML 可玩游戏试玩指引**（说明已将策划案、全套数值配表与多屏 UI 完整实现为单文件 HTML 游戏并通过 `smoke` 试玩门禁，提示用户直接点击对话卡片上的**「试玩」**按钮在浏览器中畅玩，并列出操控键位如 `WASD` 移动、`鼠标/空格` 释放技能、`B/Tab` 打开商店背包、`R` 重开）。

## 节点、端口与参数速查

完整节点端口、参数枚举与 MCP 工具契约见 `references/stages.md`。
