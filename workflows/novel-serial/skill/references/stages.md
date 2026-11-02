# 阶段表、端口与参数速查（novel-serial v2.0.0）

以下字段名、端口 id 与参数名均严格对齐应用源码 `src/shared/graph/`，可直接用于 `graph_edit`。

## 一、节点拓扑（10 节点 / 21 连线）

| key | 节点标题 | `typeId` | 阶段与职责 |
|---|---|---|---|
| `brief` | 开书立项与世界观总纲 | `text.plain` | **阶段一：立项选参**。存放用户选定的题材流派、规划总章数、单章字数、成长/施压双轴、初期准入底牌与终局封存底牌、核心人物档案及开篇冲突弧光 |
| `economy` | 经济系统 | `game.economy` | **阶段二（A）：经济循环建模**。专注 Machinations 资源水龙头/黑洞动态平衡、P10/P50/P90 蒙特卡洛仿真、奇遇/抽卡软硬保底与反通胀防线，从 `out-text` 输出经济契约 |
| `numerical` | 数值系统 | `game.numerical` | **阶段二（B）：数值定标建模**。接收 `brief (in)` 与 `economy (in-economy)`，专注全书境界阶梯、战力上限曲线、反派施压曲线、乘区边际审计、战斗 TTK 与底牌解锁门禁，从 `out-text` 输出数值契约 |
| `outline` | 小说大纲 | `novel.outline` | **阶段三：全书大纲统筹**。接收 `brief (in)`、`economy (in-economy)` 与 `numerical (in-numerical)`，生成世界观宪法、分卷架构、人物名录与分章大纲，从 `out` 广播至所有章节节点的 `in-outline` |
| `chapter1` | 小说·第1章 | `asset.novel` | **阶段四：第 1 章精写**。接收 `outline (in-outline)`、`economy (in-economy)`、`numerical (in-numerical)`，按第 1 章数值配额锁精写开篇正文，从 `out` 传给第 2 章 |
| `chapter2` | 小说·第2章 | `asset.novel` | **阶段四：第 2 章递进**。`in` 接收 `chapter1 (out)` 继承前文与章末 550 字尾锚，同时接收 `in-outline`、`in-economy`、`in-numerical` 精写第 2 章 |
| `chapter3` | 小说·第3章 | `asset.novel` | **阶段四：第 3 章递进与汇总**。`in` 接收 `chapter2 (out)`，同时接收 `in-outline`、`in-economy`、`in-numerical` 精写第 3 章；其 `out-all`（全部）端口自动汇总第 1~3 章完整正文 |
| `screenplay` | 分集短剧剧本改编（可选下游） | `asset.screenplay` | **阶段五（分支 A）：剧本衍生**。接收最后一章 `chapter3 (out-all)` 的第 1~3 章全量正文，改编为多场次影视/漫剧剧本 |
| `gameSystem` | IP游戏系统策划案（可选下游） | `asset.gameSystem` | **阶段五（分支 B）：游戏策划衍生**。通过 `in` 接收 `chapter3 (out-all)` 小说全量正文，通过 `in-economy` 接收 `economy (out-text)`，通过 `in-numerical` 接收 `numerical (out-text)`，生成六维游戏系统策划案 |
| `note` | 数值驱动长篇小说连载指南 | `note.text` | 五阶段操作流程与全屏工作台入口说明便签 |

### 连线表

| `from` | `fromPort` | `to` | `toPort` | 数据类型 | 说明 |
|---|---|---|---|---|---|
| `brief` | `out` | `economy` | `in` | `text` | 开书立项总纲输入经济系统 |
| `brief` | `out` | `numerical` | `in` | `text` | 开书立项总纲输入数值系统 |
| `economy` | `out-text` | `numerical` | `in-economy` | `text` | 经济系统资源产出/消耗速率输入数值系统，用于对齐成长节奏与消耗曲线 |
| `brief` | `out` | `outline` | `in` | `text` | 开书立项总纲输入小说大纲 |
| `economy` | `out-text` | `outline` | `in-economy` | `text` | 经济系统契约输入小说大纲 |
| `numerical` | `out-text` | `outline` | `in-numerical` | `text` | 数值系统境界阶梯与底牌门禁输入小说大纲 |
| `outline` | `out` | `chapter1` | `in-outline` | `text` | 小说大纲接入第 1 章的「小说大纲」专属端口 |
| `economy` | `out-text` | `chapter1` | `in-economy` | `text` | 经济系统接入第 1 章的「经济系统」专属端口 |
| `numerical` | `out-text` | `chapter1` | `in-numerical` | `text` | 数值系统接入第 1 章的「数值系统」专属端口 |
| `chapter1` | `out` | `chapter2` | `in` | `text` | 第 1 章「下一章」接入第 2 章「上一章」，传递前序章节历史与 550 字章末尾锚 |
| `outline` | `out` | `chapter2` | `in-outline` | `text` | 小说大纲接入第 2 章「小说大纲」端口 |
| `economy` | `out-text` | `chapter2` | `in-economy` | `text` | 经济系统接入第 2 章「经济系统」端口 |
| `numerical` | `out-text` | `chapter2` | `in-numerical` | `text` | 数值系统接入第 2 章「数值系统」端口 |
| `chapter2` | `out` | `chapter3` | `in` | `text` | 第 2 章「下一章」接入第 3 章「上一章」 |
| `outline` | `out` | `chapter3` | `in-outline` | `text` | 小说大纲接入第 3 章「小说大纲」端口 |
| `economy` | `out-text` | `chapter3` | `in-economy` | `text` | 经济系统接入第 3 章「经济系统」端口 |
| `numerical` | `out-text` | `chapter3` | `in-numerical` | `text` | 数值系统接入第 3 章「数值系统」端口 |
| `chapter3` | `out-all` | `screenplay` | `in` | `text` | 第 3 章「全部」端口输出第 1~3 章汇总正文，送入剧本节点 |
| `chapter3` | `out-all` | `gameSystem` | `in` | `text` | 第 3 章「全部」端口输出第 1~3 章汇总正文，送入策划案生成「文本」端口 |
| `economy` | `out-text` | `gameSystem` | `in-economy` | `text` | 经济系统契约送入策划案生成「经济系统」端口 |
| `numerical` | `out-text` | `gameSystem` | `in-numerical` | `text` | 数值系统契约送入策划案生成「数值系统」端口 |

---

## 二、`game.economy`（经济系统）与 `game.numerical`（数值系统）端口与参数

### 1. `game.economy`（经济系统）
- **输入端口**：
  - `in`（`text`，可多连）：接收世界观与资源规则描述
- **输出端口**：
  - `out`（`image`）：Machinations 资源循环图 + 蒙特卡洛 P10/P50/P90 时序置信带预览图
  - `out-text`（`text`，端口标签「经济系统」）：输出精炼的经济系统契约（货币水龙头/黑洞吞吐、奇遇/抽卡保底分布、通胀防线）
- **双击全屏工作台**：进入 **Machinations 经济仿真工作台**（拓扑画布、蒙特卡洛 P10/P50/P90 时序与相空间图、抽卡保底与通胀审计）

### 2. `game.numerical`（数值系统）
- **输入端口**：
  - `in`（`text`，可多连）：接收世界观与立项规则描述
  - `in-economy`（`text`，端口标签「经济系统」）：接收上游 `game.economy` 的 `out-text` 经济仿真结果
- **输出端口**：
  - `out`（`image`）：数值曲线与战力包络可视化预览图
  - `out-text`（`text`，端口标签「数值系统」）：输出精炼的数值系统契约（境界阶梯、战力上限曲线 `powerCeiling`、反派施压曲线 `villainPressure`、乘区边际收益、战斗 TTK、准入/封存底牌门禁、标准配表）
- **双击全屏工作台**：进入 **独立数值系统工作台（NumericalStudio）**，包含四大专精页签：
  1. `📈 叙事与养成曲线`（境界阶梯、每章战力上限 vs 反派施压曲线、准入/封存底牌门禁）
  2. `⚔️ 战斗与乘区精算`（多乘区伤害/属性边际收益审计、遭遇战 TTK 矩阵、Roguelike 单局战力包络）
  3. `📋 游戏打表导出`（角色属性表、技能倍率表、装备/法宝掉落表 CSV/JSON 一键导出）
  4. `📜 数值契约预览`（实时查看通过 `out-text` 向下游输出的 `numerical_system_contract_v1` JSON）

---

## 三、`novel.outline`（小说大纲）与 `asset.novel`（创建小说）端口与参数

### 1. `novel.outline`（小说大纲）端口
- **输入端口**：
  - `in`（`text`，可多连）：接收上游 `text.plain` 的立项设定
  - `in-economy`（`text`，端口标签「经济系统」）：接收 `game.economy` 的 `out-text`
  - `in-numerical`（`text`，端口标签「数值系统」）：接收 `game.numerical` 的 `out-text`
- **输出端口**：
  - `out`（`text`，端口标签「小说大纲」）：输出结构化全书大纲（世界观宪法、分卷规划、人物生死档案与分章大纲），连接至各章节节点的 `in-outline`

### 2. `asset.novel`（创建小说 · 单章节点）端口
- **输入端口**：
  - `in`（`text`，端口标签「上一章」）：仅连接上一章 `asset.novel` 的 `out`（下一章），用于链式继承前序章节正文、状态账本与章末 550 字尾锚
  - `in-outline`（`text`，端口标签「小说大纲」）：连接 `novel.outline` 的 `out`
  - `in-economy`（`text`，端口标签「经济系统」）：连接 `game.economy` 的 `out-text`
  - `in-numerical`（`text`，端口标签「数值系统」）：连接 `game.numerical` 的 `out-text`
- **输出端口**：
  - `out`（`text`，端口标签「下一章」）：输出当前章节正文（及连载历史载荷），连接至下一章 `asset.novel` 的 `in`（上一章）
  - `out-all`（`text`，端口标签「全部」）：输出**第 1 章至当前章节的全部累计章节正文汇总**，连接至下游 `asset.screenplay` 或 `asset.gameSystem`

### 3. 流派与连载规模核心参数
- `novelGenre`：小说题材流派 ID（`"xianxia_cultivation"` | `"urban_reversal"` | `"mystery_detective"` | `"ancient_romance"` | `"scifi_cyberpunk"` | `"lovecraft_rules"` | `"historical_dynasty"` | `"custom_literary"`）
- `novelTotalChapters`：全书规划总章数（默认 `600` 或 `800`，范围 `1`~`5000`）
- `novelCurrentChapter`：本节点对应的章号（如 `chapter1` 为 `1`，`chapter2` 为 `2`，`chapter3` 为 `3`）
- `novelTargetWords`：单章目标字数（默认 `3200`）
- `novelAutoAdvanceChapter`：多节点串联模式下建议设为 `false`（每个节点固定负责对应章号，也可在双击全屏工作台中点击「➕ 续写下一章」自动在画布创建并连好下一章节点）
