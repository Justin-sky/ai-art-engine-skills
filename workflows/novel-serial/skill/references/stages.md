# 阶段表、端口与参数速查（novel-serial v1.0.0）

以下字段名、端口 id 与参数名均严格对齐应用源码 `src/shared/graph/`，可直接用于 `graph_edit`。

## 一、节点拓扑（6 节点 / 6 连线）

| key | 节点标题 | `typeId` | 阶段与职责 |
|---|---|---|---|
| `brief` | 开书立项与世界观总纲 | `text.plain` | **阶段一：立项选参**。存放用户选定的题材流派、规划总章数、单章字数、成长/施压双轴、初期准入底牌与终局封存底牌、核心人物档案及第 1 章开篇钩子（`params.text`） |
| `economy` | 小说数值与经济系统 | `game.economy` | **阶段二：数值经济建模**。根据开书总纲生成 Machinations 资源循环图与 15 模块数值工作台（境界阶梯、战力膨胀上限曲线、反派施压曲线、奇遇掉落概率、底牌解锁门禁），从 `out-text` 输出完整数值包持续指导小说生成 |
| `novel` | 长篇小说逐章连载工作台 | `asset.novel` | **阶段三：一章一章递进连载**。接收 `brief` 与 `economy (out-text)`，按 `(novelCurrentChapter, novelTotalChapters)` 实时切出单章数值配额锁，维护 L1~L4 四层记忆金字塔，逐章生成 3000+ 字正文并沉淀至 `generatedTexts` 目录树 |
| `screenplay` | 分集短剧剧本改编（可选下游） | `asset.screenplay` | **阶段四（分支 A）：剧本衍生**。接收 `novel (out)` 的单章或合并多章正文，改编为带 `【人物小传】` 与场次动作的影视/漫剧剧本 |
| `gameSystem` | IP游戏系统策划案（可选下游） | `asset.gameSystem` | **阶段四（分支 B）：游戏策划衍生**。同时接收 `economy (out-text)` 数值体系与 `novel (out)` 小说 IP，生成含 ASCII 线框图与 9-slice 规范的六维游戏系统策划案 |
| `note` | 数值驱动长篇小说逐章连载指南 | `note.text` | 四阶段操作流程与全屏工作台入口说明便签 |

### 连线表

| `from` | `fromPort` | `to` | `toPort` | 数据类型 | 说明 |
|---|---|---|---|---|---|
| `brief` | `out` | `economy` | `in` | `text` | 开书立项总纲输入经济系统，驱动自动合成对应流派的数值工作台 |
| `brief` | `out` | `novel` | `in` | `text` | 开书立项总纲输入小说工作台 |
| `economy` | `out-text` | `novel` | `in` | `text` | **核心连线**：经济系统输出完整数值配额包，实时锁死小说每章境界上限与可用底牌 |
| `novel` | `out` | `screenplay` | `in` | `text` | 小说当前选中章节（或多章合并输出）送入剧本工作台改编分集剧本 |
| `economy` | `out-text` | `gameSystem` | `in` | `text` | 经济系统乘区公式与掉落保底规则送入游戏系统策划案 |
| `novel` | `out` | `gameSystem` | `in` | `text` | 小说世界观境界、角色卡池与法宝道具送入游戏系统策划案 |

---

## 二、`game.economy`（小说数值与经济系统）端口与参数

### 端口定义
- **输入端口**：
  - `in`（`text`，可多连）：接收上游 `text.plain` 的世界观与数值规则描述（或自定义 Machinations JSON）
- **输出端口**：
  - `out`（`image`）：高对比度 Machinations 拓扑图 + 蒙特卡洛 P10/P50/P90 时序置信带曲线图
  - `out-text`（`text`）：完整 `machinations_economy_bundle` JSON 载荷（内含 `numericalWorkbench` 15 模块数值工作台、`narrativeProgressionBrief` 长线叙事数值简报、`gameDataTables` 角色/技能/道具配表）

### 关键参数
- `gameEconomyPreset`：内置经济拓扑模板 ID
  - `"short_drama_reversal_loop"`：**网文/爽文逆袭首选**（憋屈蓄势池 ➔ 奇遇暴击门 ➔ 打脸爽感释放 ➔ 境界晋升）
  - `"rpg_forge"`：修仙炼器 / 装备强化养成循环
  - `"roguelike_run"`：秘境闯关 / 轮回构筑循环
  - `"gacha_pity"`：气运抽卡 / 机缘软硬保底循环
  - `"slg_city"`：历史争霸 / 宗门势力城建资源循环
  - `"idle_clicker"`：挂机修真 / 灵气指数膨胀循环
- `gameEconomySteps`：单局模拟步数（默认 `30`，范围 `1`~`500`）
- `gameEconomyMonteCarloRuns`：蒙特卡洛并发模拟轮数（默认 `60`，范围 `1`~`500`）
- `gameEconomySeed`：确定性随机种子（默认 `2026`）
- `generateInstruction`：补充说明全书境界阶段划分、战力膨胀倍率与核心资源名称

---

## 三、`asset.novel`（长篇小说逐章连载工作台）端口与参数

### 端口定义
- **输入端口**：
  - `in`（`text`，可多连）：接收上游 `game.economy` 的 `out-text` 数值配额包及 `text.plain` 的故事大纲。节点执行时会自动调用 `unwrapEconomyBundleForNarrative` 按 `(novelCurrentChapter, novelTotalChapters)` 解算出本章专属的境界锁、战力上限、准入底牌与封存底牌禁令
- **输出端口**：
  - `out`（`text`）：当前选中的章节正文（或在全屏工作台中勾选合并的多章正文）
  - `out-all`（`texts`）：已生成的全部章节数组（按章号升序排列，可接入下游 `text.select` 按章提取）

### 1. 流派与连载规模参数
- `novelGenre`：小说题材流派与叙事引擎 ID（8 选 1）
  - `"xianxia_cultivation"`：修仙长生（凡人引气 ➔ 筑基 ➔ 金丹 ➔ 元婴 ➔ 化神 ➔ 渡劫）
  - `"urban_reversal"`：都市逆袭（微末蛰伏 ➔ 初露锋芒 ➔ 商圈立威 ➔ 豪门震动 ➔ 巅峰执棋）
  - `"mystery_detective"`：悬疑刑侦（案发迷雾 ➔ 现场勘查 ➔ 嫌犯博弈 ➔ 诡计拆解 ➔ 真相收网）
  - `"ancient_romance"`：古言权谋（闺阁隐忍 ➔ 崭露头角 ➔ 府院交锋 ➔ 朝堂布局 ➔ 母仪天下）
  - `"scifi_cyberpunk"`：科幻星际（底层义体 ➔ 黑客觉醒 ➔ 荒坂突围 ➔ 星舰跃迁 ➔ 文明升维）
  - `"lovecraft_rules"`：诡秘怪谈（初触怪谈 ➔ 规则摸索 ➔ 污染侵蚀 ➔ 序列晋升 ➔ 旧日直面）
  - `"historical_dynasty"`：历史争霸（寒门起势 ➔ 屯田练兵 ➔ 割据一方 ➔ 逐鹿中原 ➔ 一统山河）
  - `"custom_literary"`：自定义文学阶梯（配合下方 4 个自定义字段）
- `novelCustomStagesText`：自定义阶段阶梯（逗号分隔，自动均匀映射至全书总章数）
- `novelCustomAxisProtagonist`：自定义主角成长轴名称（如 `"修为灵力"`、`"破案进度"`、`"家族权势"`）
- `novelCustomAxisAntagonist`：自定义反派施压轴名称（如 `"天劫强敌威胁"`、`"真凶毁证压力"`）
- `novelCustomUnlockTerm`：自定义底牌/解锁物统称（如 `"法宝神通"`、`"关键物证"`、`"商业底牌"`）
- `novelTotalChapters`：全书规划总章数（默认 `600` 或 `800`，范围 `1`~`5000`，支持连载中途动态修改）
- `novelCurrentChapter`：当前生成章号游标（1-based，如 `1` 表示生成第 1 章）
- `novelTargetWords`：单章目标字数（默认 `3200`，范围 `1000`~`15000`）
- `novelAutoAdvanceChapter`：布尔值（默认 `true`），生成完第 $N$ 章后是否自动将 `novelCurrentChapter` 推进至 $N+1$

### 2. L1~L4 四层记忆金字塔与防崩坏账本参数
- `novelWorldBible`（**L1 恒定世界观与力量宪法**）：跨全书不变的境界铁律、代价规则与门派/势力设定
- `novelStateLedger`（**L2 动态状态账本**）：每章生成后自动刷新，记录最新结算章号、主角当前境界、当前可用准入底牌、严禁提前动用的封存底牌与待承接章末钩子
- `novelRollingSummary`（**L3 滚动前情摘要**）：已生成章节的滚动浓缩摘要，防止长篇连载遗忘前文主线
- `novelChapterHistoryJson`（**L4 章节历史与 550 字原文尾锚 JSON**）：按章号键存储 `{ chapter, title, wordCount, summary, tailAnchor, updatedAt }`，生成第 $N$ 章时自动提取第 $N-1$ 章末尾 550 字原文 `tailAnchor`，确保开篇第一段与上一章结尾无缝咬合
- `novelCharacterRosterJson`（**人物生死与仇恨状态名录 JSON 数组**）：
  - 元素结构：`{ id, name, faction: "protagonist" | "ally" | "villain" | "neutral", realmOrRole, status: "active" | "retreated" | "eliminated", relationNote, updatedAtChapter }`
  - 标为 `"eliminated"`（已伏诛/退场）的反派会自动注入系统禁令，严禁后续章节“诈尸”再次登场挑衅
- `novelForeshadowingJson`（**长线伏笔埋设与回收追踪表 JSON 数组**）：
  - 元素结构：`{ id, title, plantedChapter, targetPayoffChapterRange: [startChap, endChap], status: "planted" | "paid_off", payoffHint, paidOffAtChapter? }`
  - 当 `novelCurrentChapter` 进入 `targetPayoffChapterRange` 时，系统自动提醒在本章回收该伏笔

### 3. 逐章连载时的关键操作守则
- **重跑修正某一章（In-place Overwrite）**：若对刚生成的第 $k$ 章不满意，将 `novelCurrentChapter` 设回 `k`，清空 `params.text: ""` 并修改 `generateInstruction` 后重跑 `novel` 节点，底层会原地覆盖 `generatedTexts` 中的第 $k$ 章条目，不会产生重复章节。
- **继续生成下一章（Next Chapter）**：当 `novelAutoAdvanceChapter: true` 时，第 $k$ 章跑完后 `novelCurrentChapter` 已自动变为 $k+1$；只需将 `params.text` 置为 `""` 并更新 `generateInstruction` 为第 $k+1$ 章的剧情指令，调用 `task_run({ assetId, nodeIds: ["novel"] })` 即可生成第 $k+1$ 章。
