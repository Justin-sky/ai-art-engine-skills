---
name: wf-novel-serial
description: 用「数值驱动长篇小说连载」工作流引导用户完成开书参数配置（流派/总章数/单章字数/主角与反派双轴/底牌阶梯），先行创建完整的数值经济系统（game.economy）锁定全书境界曲线与底牌门禁，再驱动创建小说节点（asset.novel）一章一章递进生成长篇小说，维护四层记忆金字塔与人物/伏笔账本，并支持同步衍生分集剧本（asset.screenplay）或同名IP游戏系统策划案（asset.gameSystem）。
workflow: novel-serial
workflow-version: 1.0.0
argument-hint: "[题材流派/小说书名/规划总章数/单章字数/主角成长轴与核心金手指/开篇冲突]"
when-to-use: 当用户想要创作小说、长篇连载网文、互动故事，或者希望用严谨的数值/境界/经济系统全程指导小说每一章的剧情推进、防止战力崩坏与吃书，以及需要将小说同步改编为剧本或游戏策划案时加载本技能。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_data_table_generate
---

# 数值驱动长篇小说连载 · 逐章生成 SOP（novel-serial v1.0.0）

> **核心设计哲学**：拒绝「单次糊弄式摘要生成」与「中后期战力崩坏/吃书」。本技能通过 **`game.economy`（小说数值与经济系统） ➔ `asset.novel`（长篇小说逐章连载工作台）** 的双核闭环，先建立全书的**数值与境界宪法**，再根据当前章号游标 `(novelCurrentChapter / novelTotalChapters)` 实时切出每一章的**战力天花板、反派施压值、准入底牌与封存底牌门禁**，结合 **L1~L4 四层记忆金字塔** 一章一章递进创作。

---

## 一、种子拓扑总览（6 节点 / 6 连线）

```
开书立项与世界观总纲 (text.plain: brief)
  ├─ out ──► in ──► 小说数值与经济系统 (game.economy: economy)
  │                   │  ├─ Machinations 资源/筹码动态平衡拓扑 + 蒙特卡洛 P10/P50/P90 模拟
  │                   │  └─ 15 模块数值工作台（境界阶梯 / 乘区公式 / 战力上限曲线 / 奇遇掉落表 / 底牌门禁）
  │                   │ out-text (实时输出全书数值配额包 machinations_economy_bundle)
  │                   ├─────────────────────────────────────────┐
  │                   ▼ in                                      ▼ in
  └─ out ──► in ──► 长篇小说逐章连载工作台 (asset.novel: novel)  IP游戏系统策划案 (asset.gameSystem: gameSystem)
                      │  ├─ 实时解包第 N/Total 章数值配额切片      ▲ in (提取小说IP人物/境界/法宝)
                      │  ├─ L1 世界观宪法 + L2 状态账本/人物生死/伏笔表
                      │  ├─ L3 滚动前情摘要 + L4 上一章末尾 550 字原文锚点
                      │  └─ 生成后自动存入章节目录树并推进游标 Ch.N ➔ Ch.N+1
                      ├─ out ───────────────────────────────────┘
                      ▼ out ──► in
                    分集短剧剧本改编 (asset.screenplay: screenplay)
```

---

## 二、四阶段标准执行规程（MCP 路径）

### 阶段一：开书立项参数选择（Interactive Parameter Selection）

当用户提出写小说需求时，若用户尚未指定完整参数，先用精炼清晰的列表向用户确认（或提供默认推荐值供用户一键确认）以下 **6 项核心立项参数**：

1. **题材流派（`novelGenre`）**：
   - `xianxia_cultivation`（修仙长生 · 境界突破与法宝神通）
   - `urban_reversal`（都市逆袭 · 财富权势与商业打脸）
   - `mystery_detective`（悬疑刑侦 · 关键证据链与真凶博弈）
   - `ancient_romance`（古言权谋 · 家族声望与朝堂宿命）
   - `scifi_cyberpunk`（科幻星际 · 义体算力与巨企危机）
   - `lovecraft_rules`（诡秘怪谈 · 污染理智值与禁忌封印物）
   - `historical_dynasty`（历史争霸 · 粮草兵力与天下格局）
   - `custom_literary`（自由文学自定义阶梯 · 自定义阶段名与成长/阻力双轴）
2. **连载规模（`novelTotalChapters` & `novelTargetWords`）**：
   - 全书规划总章数：`80`（中篇精品）/ `200`（标准长篇）/ `600`（大长篇连载）/ `1200+`（超长篇巨制），连载中途可随时动态调整；
   - 单章目标字数：默认 `3200` 字/章（支持 `1500` ~ `10000` 字）。
3. **主角成长轴 vs 反派施压轴**：
   - 如 `novelCustomAxisProtagonist: "修为灵力上限"` vs `novelCustomAxisAntagonist: "宗门强敌与天劫威胁"`（若使用预设流派则自动内置，也可自定义覆盖）。
4. **底牌与筹码解锁阶梯（防战力崩坏核心）**：
   - **前期准入底牌**（第 1 卷允许动用的低阶功法/信物/线索）；
   - **终局封存底牌**（严禁在前中期章节提前泄露的大招/终极真相，由合规体检器实时监控）。
5. **核心角色名录与第 1 章开篇冲突**：
   - 主角、核心盟友、首卷反派的姓名与动机，以及第 1 章开篇的第一幕强冲突钩子。
6. **连载推进节奏**：
   - **逐章精写确认模式（推荐默认）**：每生成完 1 章，向用户汇报本章字数、数值合规得分、人物/伏笔状态更新与章末悬念钩子，用户确认或微调后再生成下一章；
   - **连续批次连载模式**：用户指定「一次连写 $K$ 章（如连写 3 章）」，AI 按 `Ch.1 ➔ Ch.2 ➔ Ch.3` 顺序逐章闭环调用运行。

---

### 阶段二：创建并运行小说内部数值经济系统（`game.economy`）

1. **铺设工作流种子骨架**：
   - 调用 `workflow_use_installed({ id: "novel-serial", mode: "seed" })` 在当前画布加载种子拓扑。
   - *按需裁剪（Prune Optional Downstream）*：若用户当前只需专心写小说、暂不需要剧本或游戏策划案，保留或调用 `graph_edit` 删去 `screenplay` / `gameSystem` 均可；默认保留时只需先运行 `economy` 和 `novel`。
2. **写入开书总纲并配置数值经济系统**：
   - 调用 `graph_edit` 将用户选定的立项参数、世界观、阶段阶梯、初期准入底牌与终局封存底牌写入 `brief`（`text.plain`）的 `params.text`；
   - 同时配置 `economy`（`game.economy`）的 `params`：
     - `gameEconomyPreset`：网文/爽文逆袭首选 `"short_drama_reversal_loop"`；修仙炼器/刷宝可选 `"rpg_forge"` 或 `"roguelike_run"`；抽卡/秘境气运可选 `"gacha_pity"`；势力争霸可选 `"slg_city"`；
     - `generateInstruction`：写明本小说的成长轴名称、全书总章数、每卷境界上限与货币/资源产出消耗规则。
3. **先行运行 `economy` 节点（必做！）**：
   - 调用 `task_run({ assetId, nodeIds: ["economy"] })` 并用 `task_status` 等待完成（本地确定性蒙特卡洛与数值工作台合成，仅需 1~2 秒）。
   - `economy` 运行后会产出完整的 **15 模块数值工作台**（含全书分卷规划、每章战力上限公式 `powerCeiling` vs 反派压力 `villainPressure`、准入底牌表 `allowedTrumpCards`、封存底牌表 `lockedEndgameTrumps`、奇遇掉落表），并通过 `out-text ➔ in` 持续指导 `novel` 节点！

---

### 阶段三：在数值系统持续指导下「一章一章递进生成」（`asset.novel`）

**⚠️ 严禁试图在一个章节里写完全书梗概！必须严格按 `novelCurrentChapter = 1, 2, 3...` 一章一章生成：**

1. **初始化第 1 章参数与四层记忆种子**：
   - 调用 `graph_edit` 更新 `novel`（`asset.novel`）的 `params`：
     - `novelGenre`：用户选定的流派 ID；
     - `novelTotalChapters`：规划总章数（如 `600`）；
     - `novelCurrentChapter`: `1`（从第 1 章开始）；
     - `novelTargetWords`: 单章目标字数（如 `3200`）；
     - `novelAutoAdvanceChapter`: `true`（每章生成完自动将游标 `+1` 推进到下一章）；
     - `novelWorldBible`（L1 恒定世界观宪法）：写入流派铁律、境界阶梯、力量代价与核心设定；
     - `novelStateLedger`（L2 动态状态账本）：写入第 1 章初始状态（主角当前境界、持有准入底牌、封存底牌清单）；
     - `text`: `""`（清空底稿以触发全新章节生成）；
     - `generateInstruction`：针对第 1 章的四拍戏剧节拍指令（**第一拍：困境/铺垫蓄势 → 第二拍：奇遇/线索破局 → 第三拍：反派施压拉满 → 第四拍：准入底牌反击立威 + 章末高能断章钩子**）。
2. **执行第 $k$ 章生成**：
   - 调用 `task_run({ assetId, nodeIds: ["novel"] })`，用 `task_status` 等待本章生成完毕。
   - **底层自动完成的闭环机制**：
     - `novel` 节点执行时自动读取上游 `economy` 的 `out-text` 载荷，按 `(novelCurrentChapter, novelTotalChapters)` 精确解算**本章数值配额切片**（锁死本章境界、战力上限、准入底牌与封存底牌禁令）；
     - 生成完成后，自动将第 $k$ 章完整正文追加进 `params.generatedTexts` 多章目录树，自动截取章末最后 550 字存入 `novelChapterHistoryJson`（L4 原文尾锚，供第 $k+1$ 章第一段无缝咬合），自动更新 `novelStateLedger`（L2 状态账本）与 `novelRollingSummary`（L3 滚动前情提要），并将 `novelCurrentChapter` 自动推进为 $k + 1$。
3. **章节交付汇报与推进下一章（第 $k+1$ 章）**：
   - 调用 `graph_read({ assetId, includeParams: true })` 读取 `novel` 节点的最新状态，向用户汇报：
     - ✅ **已生成章节标题与字数**（如 `第 1 / 600 章《青铜剑匣》· 3,420 字`）；
     - ⚖️ **本章数值配额与状态结算**（当前境界、本章动用的准入底牌、有无越界泄露封存底牌）；
     - 🪝 **章末断章悬念钩子**（展示最后一段悬念，并给出第 $k+1$ 章的剧情预告）。
   - **当继续生成第 $k+1$ 章时**：
     - 若需微调人物生死状态（如某反派在第 $k$ 章已伏诛退场，将其在 `novelCharacterRosterJson` 中标为 `"eliminated"` 防止后续章节复活）或回收某条到期伏笔（`novelForeshadowingJson`），用 `graph_edit` 更新 `novel` 的 `generateInstruction`（写入第 $k+1$ 章具体冲突指令，并确保 `params.text` 设为空串 `""` 以便生成新章节而非润色旧章），再次调用 `task_run({ assetId, nodeIds: ["novel"] })` 即可！
     - **动态调改数值系统**：若剧情发展到新卷需要调整经济通胀率、新增神兵底牌或修改境界跨度，随时用 `graph_edit` 更新 `economy` 节点并重跑 `economy`，新的数值体系会立即作用于后续所有章节。

---

### 阶段四：全屏工作台精修与下游剧本/游戏策划案衍生

- **双击进入全屏工作台**：
  - 双击画布上的 `novel` 节点进入**长篇小说全屏工作台**：左侧管理多章节目录树、动态调整总章数（80~5000 章）、勾选多章合并输出；中间沉浸式正文编辑与局部框选爽点扩写/底牌合规替换；右侧三页签查看**「⚖️ 本章数值配额与合规体检医生」「🗺️ 分卷多章大纲推演与自动连载」「🧠 四层记忆金字塔与人物生死/伏笔追踪表」**。
  - 双击 `economy` 节点进入 **Machinations 经济拓扑与 15 模块数值工作台**，可视化调优战力曲线与掉落概率。
- **一键衍生分集剧本（`screenplay`）或游戏策划案（`gameSystem`）**：
  - 需要将当前章节（或勾选合并的多章）改编为短剧剧本时，运行 `task_run({ assetId, nodeIds: ["screenplay"] })`，随后可进一步接入 `film.workbench` 拍摄短剧成片；
  - 需要将小说 IP 与 `economy` 数值体系转化为同名游戏系统策划案（GDD）时，运行 `task_run({ assetId, nodeIds: ["gameSystem"] })`，自动提取小说人物卡池、境界阶梯、法宝道具与经济乘区公式，产出可直接接入 `ui.split` 的工业级策划案。

---

## 三、节点端口与参数速查

完整的 6 节点拓扑、端口定义、`asset.novel` 八大流派枚举、四层记忆金字塔字段与 `game.economy` 联动参数速查见 `references/stages.md`。
