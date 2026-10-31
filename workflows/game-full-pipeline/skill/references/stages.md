# 节点、端口、参数与可玩 HTML 构建速查（game-full-pipeline）

## 1. 种子画布节点与连线拓扑

调用 `workflow_use_installed({ id: "game-full-pipeline", mode: "seed" })` 后，画布上会生成以下 12 个节点与 9 条连线：

| 节点 `key` (`id`) | `typeId` | 标题 | 核心职责 |
|---|---|---|---|
| `brief` | `text.plain` | 游戏立项与参数总纲 | 存放用户确认的品类、2D/3D视角、美术与UI画风预设、核心循环、数值目标与多屏 UI 规划 |
| `economy` | `game.economy` | 经济系统与全套数值体系 | 运行 Machinations 资源动力学与蒙特卡洛模拟，产出置信带曲线图（`out`）与完整经济数值包（`out-text`） |
| `gameSystem` | `asset.gameSystem` | 全套游戏系统策划案 (GDD) | 接收 `brief` 与 `economy` (`out-text`)，产出涵盖核心循环、战斗/经济公式、配表结构与逐屏 UI 规范的完整策划案 |
| `uiSplit` | `ui.split` | 全游戏 UI 界面拆分 | 将 `gameSystem` 策划案拆解为最多 12 屏双轨 UI 提示词（带字 `prompt` + 无字底图 `cleanPrompt`） |
| `uiGen` | `ui.gen` | 全套 UI 界面生成 | 双击进入内图逐屏产出带字精修图与空字底图，回外层 Cook 后输出整套 UI 界面图（`out` / `out-selected`） |
| `uiLayerSplit` | `image.layerSplit` | UI 图层分离与九宫格导出 | 将选中的 UI 界面拆解为底板、面板容器、装饰与多态按钮，支持 9-Slice 九宫格拉伸与分层 PSD 导出 |
| `heroSprite` | `asset.image` | 主角核心精灵设定 | 生成 1:1 原生透明背景的主角基准精灵图 |
| `enemySprite` | `asset.image` | 怪物与Boss精灵设定 | 生成 1:1 原生透明背景的怪物/Boss 精灵图 |
| `levelMap` | `game.levelMap` | 可玩关卡地图生成 (.tmj) | 程序化生成 2D 关卡地图预览图（`out`）与标准 Tiled JSON（`out-text`） |
| `bgmDsp` | `game.audioDsp` | 无缝循环 BGM 工坊 | 生成或处理首尾等功率交叉淡化的无缝循环游戏 BGM WAV |
| `sfxDsp` | `game.audioDsp` | 防疲劳战斗音效矩阵 | 生成或处理 0ms 瞬发修剪 + 4× 变调防听觉疲劳的战斗音效 WAV |
| `note` | `note.text` | 全栈游戏设计与可玩HTML指南 | 工作流全流程操作说明 |

### 标准连线表

| `from` | `fromPort` | `to` | `toPort` | 数据流说明 |
|---|---|---|---|---|
| `brief` | `out` | `economy` | `in` | 立项参数注入经济与数值模型 |
| `brief` | `out` | `gameSystem` | `in` | 立项参数注入系统策划案 |
| `economy` | `out-text` | `gameSystem` | `in` | 完整经济数值体系与配表护栏实时注入系统策划案 |
| `gameSystem` | `out` | `uiSplit` | `in` | 系统策划案正文送入 UI 界面拆分器 |
| `uiSplit` | `out` | `uiGen` | `in` | 逐屏双轨 UI 提示词数组送入 UI 界面生成器 |
| `uiGen` | `out-selected` | `uiLayerSplit` | `in` | 选中的 UI 界面图送入 UI 图层分离器导出 PSD 与九宫格 |
| `brief` | `out` | `heroSprite` | `in-text` | 立项角色设定驱动主角精灵生成 |
| `brief` | `out` | `enemySprite` | `in-text` | 立项怪物设定驱动敌人精灵生成 |
| `brief` | `out` | `levelMap` | `in` | 立项关卡设定驱动可玩地图生成 |

---

## 2. 核心节点端口与参数契约

### 2.1 `game.economy`（经济系统与全套数值体系）

| 端口 | 方向 | 类型 | 说明 |
|---|---|---|---|
| `in` | in | `text`（multiple） | 上游游戏设定与数值目标描述 |
| `out` | out | `image` | Machinations 拓扑图与 P10/P50/P90 置信带曲线图 |
| `out-text` | out | `text` | 完整经济系统 JSON 载荷（含 `workbenchReport`、`qaReportMarkdown`、`dataTableBundle`） |

核心参数：
- `gameEconomyPreset`（预设模板枚举）：
  - `survivor_xp_loop`（肉鸽割草经验与波次循环）
  - `roguelike_mana_deck`（杀戮尖塔式法力与牌库构筑）
  - `hearthstone_mana_pack`（炉石式法力水晶与卡包经济）
  - `onmyoji_speed_orbs`（回合制行动条与鬼火资源池）
  - `balatro_ante_jokers`（小丑牌筹码×倍率指数闯关）
  - `arpg_equipment_economy`（ARPG 装备刷宝与词条洗练回收）
  - `gacha_pity_banner`（商业抽卡大小保底概率门禁）
  - `idle_exponential_tycoon`（放置大亨指数产出与转生重置）
  - `tower_defense_waves`（塔防波次击杀赏金与建塔升级）
  - `crafting_chain`（生存建造采集与多级合成链）
  - `energy_stamina_timer`（体力自然恢复与付费购买漏斗）
  - `roguelike_run` / `simple_faucet_sink` / `short_drama_reversal_loop`
- `gameEconomySteps`：单次模拟步数（10~200，默认 `50`）
- `gameEconomyMonteCarloRuns`：蒙特卡洛随机采样次数（5~200，默认 `60`）
- `gameEconomySeed`：确定性随机种子（默认 `2026`）

### 2.2 `asset.gameSystem`（全套游戏系统策划案）

| 端口 | 方向 | 类型 | 说明 |
|---|---|---|---|
| `in` | in | `text`（multiple） | 接收 `brief` 与 `economy` (`out-text`)，自动解包上游数值与经济体系 |
| `out` | out | `text` | 结构化全套游戏系统策划案（GDD）正文 |

核心参数：
- `gameSystemArchetype`（8 大系统策划品类模板）：
  - `roguelike_survivor`（俯视角动作肉鸽与局内构筑）
  - `card_dbg_combat`（回合制卡牌构筑与遗物协同）
  - `rpg_progression`（角色扮演养成、技能树与装备成长）
  - `gacha_equipment_loot`（抽卡养成、圣遗物/词条刷宝与商业化循环）
  - `avg_mystery_branch`（悬疑推理/恋爱多分支叙事与好感度/线索账本）
  - `rule_horror_sanity`（规则怪谈、SAN 值理智惩罚与探索解谜）
  - `slg_faction_intrigue`（SLG 沙盘战略、六角地块行军与科技外交）
  - `universal_full_ui`（全品类通用 8~12 屏完整 UI 系统策划）
- `gameSystemTargetAspect`：`"16:9"`（横屏）| `"9:16"`（竖屏）| `"1:1"`
- `generateInstruction`：策划案生成指令

### 2.3 `ui.split` → `ui.gen` → `image.layerSplit`（多屏 UI 管线）

- **`ui.split`**：
  - 输入 `in` (`text`) 接 `gameSystem` (`out`)；输出 `out` (`texts`) 为最多 12 屏 UI 提示词数组。
  - 每屏自动产出带字 `prompt` 与无字底图 `cleanPrompt`。
- **`ui.gen`**：
  - 输入 `in` (`texts`) 接 `uiSplit` (`out`)；可选 `in-image` (`image`) 接风格参考图。
  - 输出 `out` (`images`) 汇集全部界面图，`out-selected` (`image`) 输出当前选中界面图。
  - 参数 `uiGenStylePreset` 支持 8 大主题风格：
    - `celestial_star_gold`（星轨白金幻想）
    - `persona_rebel_red`（怪盗红黑潮酷）
    - `minimal_ark_tech`（极简战术终端）
    - `cyber_holo_grid`（赛博全息霓虹）
    - `ink_paper_cel`（国风水墨仙侠）
    - `dark_iron_parchment`（暗黑哥特羊皮纸）
    - `royal_gold_enamel`（皇室鎏金珐琅）
    - `arcane_crystal`（奥术水晶法阵）
- **`image.layerSplit`**：
  - 输入 `in` (`image`) 接 `uiGen` (`out-selected`)，拆解为可编辑 PSD 图层、9-Slice 九宫格边框与多态按钮。

---

## 3. 全套数值配表与可玩 HTML 构建工具契约

| MCP 工具名 | 用途 | 产出物路径 |
|---|---|---|
| `game_data_table_generate` | 生成角色属性表（`characters`）、技能表（`skills`）、道具遗物表（`items`）、DBG 卡牌表（`cards`）、刷怪波次表（`waves`）、分支对白表（`dialogues`）及多语言翻译表 `localization.csv` | `Assets/GameData/<tableName>/*.csv` 与 `*.json` |
| `game_systems_graph_generate` | 生成无环校验通过的技能树 DAG（`skillTree`）、合成配方（`craftingRecipes`）、任务链（`quests`）、Boss AI 状态机（`aiBehaviors` + Mermaid 图）与 BMFont 位图字体（`.fnt`） | `Assets/GameData/<systemName>/*_systems.json` 与 `*_graph.mmd` |
| `gameplay_prepare_project` | 在工程下初始化纯 Node + esbuild 可玩 HTML 游戏脚手架（`mode: "2d" \| "3d"`, `title`），返回 `projectRelativeDir` | `Cache/GamePlayJobs/<jobDir>/src/**` |
| `gameplay_build` | 宿主后台执行 `npm install` + `node build.mjs` 编译单文件 `dist/single.html`，自动登记 `gamePlay` 资产并触发隐藏窗口 `smoke` 试玩门禁 | `dist/single.html` + 对话流「试玩」卡片 |
| `gameplay_job_status` | 轮询打包进度与 `smoke` 试玩门禁报告（`smoke.ok`、`smoke.errors`、`smoke.metrics`） | 状态 JSON |
| `game_playable_preview_compile` | 将画布生成的角色/怪物精灵与瓦片一键编译为 60fps 零依赖 HTML5 微缩沙盒 | `Assets/GamePlayable/*.html` |
