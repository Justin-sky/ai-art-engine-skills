---
name: wf-novel-serial
description: 用「数值驱动长篇小说连载」工作流引导用户完成开书参数配置，先行创建经济系统（game.economy）与数值系统（game.numerical）锁定资源循环、全书境界阶梯与底牌门禁，再通过小说大纲（novel.outline）与逐章串联的创建小说节点（asset.novel：第1章➔第2章➔第3章…）递进生成长篇小说，并通过最后一章的「全部（out-all）」端口一键汇总衍生分集剧本（asset.screenplay）或同名IP游戏系统策划案（asset.gameSystem）。
workflow: novel-serial
workflow-version: 2.0.0
argument-hint: "[题材流派/小说书名/规划总章数/单章字数/主角成长轴与核心金手指/开篇冲突]"
when-to-use: 当用户想要创作小说、长篇连载网文、互动故事，或者希望用独立的经济系统与数值系统全程指导小说大纲和每一章的剧情推进、防止战力崩坏与吃书，以及需要将小说同步改编为剧本或游戏策划案时加载本技能。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_data_table_generate
---

# 数值驱动长篇小说连载 · 逐章生成 SOP（novel-serial v2.0.0）

> **核心设计哲学**：拒绝「单次糊弄式摘要生成」与「中后期战力崩坏/吃书」。本技能采用 **`game.economy`（经济系统） + `game.numerical`（数值系统） ➔ `novel.outline`（小说大纲） ➔ 串联式 `asset.novel`（小说·第1章 ➔ 第2章 ➔ 第3章…）** 的工业级管线：
> 1. **经济与数值双核分治**：`game.economy` 专注 Machinations 资源水龙头/黑洞动态平衡、通胀预警与奇遇保底模拟；`game.numerical` 专注境界阶梯、战力上限曲线、反派施压曲线、乘区审计与底牌门禁定标；
> 2. **四输入端口精准路由**：每个 `asset.novel` 章节节点拥有 4 个语义化输入端口——`in`（上一章）、`in-outline`（小说大纲）、`in-economy`（经济系统）、`in-numerical`（数值系统）；
> 3. **双输出端口汇总流转**：每个 `asset.novel` 章节节点的 `out`（下一章）向后传递本章正文与章末 550 字尾锚；`out-all`（全部）自动汇总从第 1 章到当前章的完整正文，直接送入下游剧本改编或游戏策划案生成。

---

## 一、种子拓扑总览（10 节点 / 21 连线）

```
开书立项与世界观总纲 (text.plain: brief)
  ├─ out ──► in ──────────► 经济系统 (game.economy: economy)
  │                           │ out-text (Machinations 资源/货币/保底契约)
  │                           ├────────────────────┬────────────────────┬────────────────────┐
  │                           ▼ in-economy         │                    │                    │
  ├─ out ──► in ──────────► 数值系统 (game.numerical: numerical)        │                    │
  │                           │ out-text (境界阶梯/战力曲线/乘区/底牌门禁)│                    │
  │                           ├────────────────────┼────────────────────┼────────────────────┤
  │                           ▼ in-numerical       ▼ in-economy         │                    │
  └─ out ──► in ──────────► 小说大纲 (novel.outline: outline) ◄─────────┘                    │
                              │ out (全书世界观宪法 + 分卷规划 + 分章大纲)                    │
                              ├────────────────────┬────────────────────┐                    │
                              ▼ in-outline         ▼ in-outline         ▼ in-outline         │
                            小说·第1章 ──out──► in 小说·第2章 ──out──► in 小说·第3章          │
                            (chapter1)           (chapter2)           (chapter3)             │
                            ▲ in-economy         ▲ in-economy         ▲ in-economy ◄─────────┤
                            ▲ in-numerical       ▲ in-numerical       ▲ in-numerical         │
                                                                        │                    │
                                                                        ├─ out-all (全部) ──►├─► in 分集短剧剧本改编 (asset.screenplay: screenplay)
                                                                        └─ out-all (全部) ──►└─► in IP游戏系统策划案 (asset.gameSystem: gameSystem)
                                                                                                   ▲ in-economy (连自 economy.out-text)
                                                                                                   ▲ in-numerical (连自 numerical.out-text)
```

---

## 二、五阶段标准执行规程（MCP 路径）

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
   - 全书规划总章数：`80`（中篇精品）/ `200`（标准长篇）/ `600`（大长篇连载）/ `1200+`（超长篇巨制）；
   - 单章目标字数：默认 `3200` 字/章（支持 `1500` ~ `10000` 字）。
3. **主角成长轴 vs 反派施压轴**：
   - 如 `novelCustomAxisProtagonist: "修为灵力上限"` vs `novelCustomAxisAntagonist: "宗门强敌与天劫威胁"`。
4. **底牌与筹码解锁阶梯（防战力崩坏核心）**：
   - **前期准入底牌**（第 1 卷允许动用的低阶功法/信物/线索）；
   - **终局封存底牌**（严禁在前中期章节提前泄露的大招/终极真相，由合规体检器实时监控）。
5. **核心角色名录与前 3 章开篇冲突**：
   - 主角、核心盟友、首卷反派的姓名与动机，以及前 3 章的起承转合冲突弧光。
6. **连载章节数**：
   - 种子默认铺设 3 个串联章节节点（`chapter1` ➔ `chapter2` ➔ `chapter3`），可按用户需求用 `graph_edit` 动态增删章节节点。

---

### 阶段二：先行建模「经济系统」与「数值系统」（`game.economy` + `game.numerical`）

1. **铺设工作流种子骨架**：
   - 调用 `workflow_use_installed({ id: "novel-serial", mode: "seed" })` 在当前画布加载种子拓扑。
   - *按需裁剪（Prune Optional Downstream）*：若用户当前只需写小说、暂不需要剧本或游戏策划案，可调用 `graph_edit` 删去 `screenplay` / `gameSystem`；若保留，先运行上游节点即可。
2. **写入开书总纲并配置经济与数值双系统**：
   - 调用 `graph_edit` 将用户选定的立项参数写入 `brief`（`text.plain`）的 `params.text`；
   - 配置 `economy`（`game.economy`）：设置 `gameEconomyPreset`（网文逆袭首选 `"short_drama_reversal_loop"`，炼器养成可选 `"rpg_forge"`，秘境轮回可选 `"roguelike_run"`，抽卡机缘可选 `"gacha_pity"`，势力争霸可选 `"slg_city"`）；
   - 配置 `numerical`（`game.numerical`）：通过 `in-economy` 接收 `economy` 的 `out-text` 载荷，定标全书境界阶梯、战力上限曲线、反派施压曲线、乘区公式与底牌门禁。
3. **依序运行 `economy` 与 `numerical`（必做！）**：
   - 先调用 `task_run({ assetId, nodeIds: ["economy"] })` 并等待完成；
   - 再调用 `task_run({ assetId, nodeIds: ["numerical"] })` 并等待完成。

---

### 阶段三：生成全书「小说大纲」（`novel.outline`）

1. **配置并运行 `outline`（`novel.outline`）**：
   - `outline` 节点同时连接 `brief (out ➔ in)`、`economy (out-text ➔ in-economy)` 与 `numerical (out-text ➔ in-numerical)`；
   - 调用 `task_run({ assetId, nodeIds: ["outline"] })` 生成全书世界观宪法、分卷架构、核心人物名录与分章大纲；
   - `outline` 的 `out` 端口通过连线接入每一个章节节点的 `in-outline`（小说大纲）端口，确保所有章节共享统一的全书大纲。

---

### 阶段四：逐章串联生成小说正文（`chapter1 ➔ chapter2 ➔ chapter3`）

1. **四端口各司其职**：
   - **第 1 章（`chapter1`）**：连接 `outline (out ➔ in-outline)`、`economy (out-text ➔ in-economy)`、`numerical (out-text ➔ in-numerical)`，`novelCurrentChapter: 1`；
   - **第 $k$ 章（`chapter2`, `chapter3`…）**：
     - `in`（上一章）：连接第 $k-1$ 章的 `out`（下一章），自动继承前序章节正文历史、L2 状态账本、L3 滚动摘要与 L4 上一章末尾 550 字原文尾锚；
     - `in-outline`（小说大纲）：连接 `outline` 的 `out`；
     - `in-economy`（经济系统）：连接 `economy` 的 `out-text`；
     - `in-numerical`（数值系统）：连接 `numerical` 的 `out-text`。
2. **按章序依次执行**：
   - 依次调用 `task_run` 运行 `chapter1` ➔ `chapter2` ➔ `chapter3`（或在生成完每一章后向用户汇报字数、境界合规体检得分与章末悬念钩子）。
   - 如需继续扩写第 4 章、第 5 章，只需调用 `graph_edit` 新增 `asset.novel` 节点（或在画布上点击章节节点的「➕ 续写下一章」按钮），将其 `in` 连自上一章 `out`，并连入 `in-outline`、`in-economy`、`in-numerical` 即可。

---

### 阶段五：通过「全部（`out-all`）」端口汇总衍生剧本与游戏策划案

- **全量章节汇总输出**：
  - 串联链条上任意一章的 `out-all`（全部）端口都会自动输出**第 1 章至该章的完整累计章节正文**（按章号升序合并）。
  - 将最后一章（如 `chapter3`）的 `out-all` 连接到下游节点，即可一次性将全书已写章节全部送入下游！
- **衍生分集剧本（`screenplay`）**：
  - `chapter3 (out-all) ➔ screenplay (in)`：运行 `screenplay` 即可将第 1~3 章完整小说改编为标准多场次短剧/漫剧剧本。
- **衍生同名 IP 游戏系统策划案（`gameSystem`）**：
  - `gameSystem` 节点具备 3 个输入端口：
    - `in`（文本）：连接最后一章的 `out-all`（接收小说世界观、人物卡池与法宝剧情）；
    - `in-economy`（经济系统）：连接 `economy` 的 `out-text`（接收资源循环、通胀防线与保底报告）；
    - `in-numerical`（数值系统）：连接 `numerical` 的 `out-text`（接收境界曲线、乘区公式、TTK 定标与配表）；
  - 运行 `gameSystem` 即可产出融合小说 IP 与双系统数值契约的六维工业级游戏策划案（GDD），并可继续接入 `ui.split ➔ ui.gen` 生成全套游戏 UI。

---

## 三、节点端口与参数速查

完整的 10 节点拓扑、端口定义、`novel.outline` / `asset.novel` / `game.economy` / `game.numerical` / `asset.gameSystem` 联动参数速查见 `references/stages.md`。
