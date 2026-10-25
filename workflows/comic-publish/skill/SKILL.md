---
name: wf-comic-publish
description: 用「漫画出版」工作流把一段剧本自动变成一集多页漫画（script → charAnchor 角色与画风定妆锚点 → 多页分镜图 p1_1..p3_3 → page1..page3 漫画页自动排版、斜切破格、速度线/集中线与台词气泡/音效字合成 PNG），并支持自由增删页数/格数或接入时间线制作有声动态漫。当用户要做一集漫画、多页连载条漫/日漫、四格漫画或输入剧本自动生成带气泡的漫画成片时加载本技能。
workflow: comic-publish
workflow-version: 1.1.0
argument-hint: "[漫画剧本/故事梗概/画风（日漫黑白网点/韩系全彩条漫/国漫古风/美漫）] [页数与格数]"
when-to-use: 用户要做一集漫画、多页连载条漫/日漫、四格漫画或输入一段剧本自动生成带台词气泡的漫画成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 漫画出版 · 整集漫画自动生产与自由排版 SOP（comic-publish v1.1，默认 16 节点 / 31 连线）

## 默认种子拓扑与产出物（Seed Skeleton）

```
漫画剧本 (text.plain: script)
  ├─out→in-text──→ 角色与画风定妆 (asset.image: charAnchor，输出参考图连入全集所有分镜的 in-image 端口，锁定主角外貌与统一画风)
  ├─out→in-text──→ 第 1 页（起承）：分镜图 p1_1, p1_2, p1_3 (asset.image) ──out→in-image──→ 漫画页·第 1 页 (comic.page: page1)
  ├─out→in-text──→ 第 2 页（高潮）：分镜图 p2_1, p2_2, p2_3, p2_4 (asset.image) ──out→in-image──→ 漫画页·第 2 页 (comic.page: page2)
  └─out→in-text──→ 第 3 页（收尾）：分镜图 p3_1, p3_2, p3_3 (asset.image) ──out→in-image──→ 漫画页·第 3 页 (comic.page: page3)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "comic-publish", mode: "seed" })` 物化后，画布上的节点 `id` 与 `workflow.json` 里的 `key` **完全一致**（`script`、`charAnchor`、`p1_1`~`p1_3`、`page1`、`p2_1`~`p2_4`、`page2`、`p3_1`~`p3_3`、`page3`、`note`），可直接用于 `graph_edit`！

| 产物层级 | 节点 Key (`typeId`) | 作用 |
|---|---|---|
| 角色与画风定妆图 | `charAnchor` (`asset.image`) | 第 1 层最先执行，产出主角全身/神态设定图，通过 `out → in-image` 连入后续全部 10 格分镜图，确保跨页角色长相、服装、画风一致 |
| 全集分镜原画（默认 10 张） | `p1_1`~`p3_3` (`asset.image`) | 第 2 层并发执行，结合剧本与角色定妆参考图生成纯画面分镜（不含画面文字） |
| 合成漫画页 PNG（默认 3 页） | `page1`~`page3` (`comic.page`) | 第 3 层自动执行，按阅读顺序把上游分镜图填入分格，并把 `params.comicPage` 中配置的**分格布局、斜切多边形（`cornerOffsets`）、镜头特效（`effectType`）与 6 类台词气泡（`bubbles`）**合成为高清漫画页 PNG |

---

## 核心能力：用户输入一段剧本 → AI 全自动创建一集漫画 SOP

当用户选中本工作流并输入一段剧本（或故事梗概）时，**不要让用户自己去手动排版或手填台词**，请直接按以下 5 步全自动完成整集漫画的编排与生成：

### 第 1 步：落盘工作流种子骨架
调用 `workflow_use_installed({ id: "comic-publish", mode: "seed" })` 落盘工作流资产，拿到 `assetId` 与 `nodeKeyMap`。

### 第 2 步：按故事体量规划「起承转合」页数与格数（支持动态增删）
- **默认 3 页 10 格种子结构**：
  - **第 1 页（`page1`，起承 3 格）**：`p1_1` 顶部横幅定场远景（`row:0, col:0, rowSpan:2, colSpan:6`）+ `p1_2` 左下竖格主角登场（`row:2, col:0, rowSpan:4, colSpan:3`）+ `p1_3` 右下竖格异变突起（`row:2, col:3, rowSpan:4, colSpan:3`）。
  - **第 2 页（`page2`，高潮 4 格斜切破格）**：`p2_1` 顶部斜切疾速逼近 + `p2_2` 中左斜切正面交锋 + `p2_3` 中右斜切特写反击 + `p2_4` 底部横幅破局一击。
  - **第 3 页（`page3`，收尾 3 格终幕悬念）**：`p3_1` 左上余波散尽 + `p3_2` 右上特写线索 + `p3_3` 底部跨幅大格终幕悬念。
- **动态裁剪或扩页（Seed & Dynamic Composition）**：
  - **单页四格漫画 / 单页条漫**：用 `graph_edit` 的 `node_delete` 删掉 `p2_*`、`page2`、`p3_*`、`page3`，只在 `page1` 上连接 4 个分镜节点，立省 60% 生图耗时；
  - **长篇 4~6 页连载**：用 `node_upsert` 新增 `p4_1`~`p4_3`（`asset.image`）与 `page4`（`comic.page`），用 `edge_connect` 将 `script:out → p4_*:in-text`、`charAnchor:out → p4_*:in-image`、`p4_*:out → page4:in-image` 按阅读顺序连好；
  - **升级为「有声动态漫视频」（跨技能混搭 `output.timeline`）**：新增 `voice1`~`voice3`（`asset.voice`）、`bgm`（`asset.music`）与 `timeline`（`output.timeline`），将 `page1`~`page3:out` 连入 `timeline:in-image`，即可一键导出带配音与 BGM 的有声漫画视频！

### 第 3 步：更新剧本、角色定妆锚点与每一格画面指令（`graph_edit`）
在一次 `graph_edit` 批量操作中同时更新：
1. **`script`（`text.plain`）**：把整理好的分场分格剧本写入 `params.text`。
2. **`charAnchor`（`asset.image`）**：在 `params.generateInstruction` 中写明主角外貌特征（发型、发色、瞳色、服饰、年龄气质）与整集漫画画风（如「日系热血黑白网点漫画风」「韩系全彩条漫风」「国漫古风厚涂」「美式硬派漫画风」），强调「角色设定基准图，纯画面不含任何文字」。
3. **全部分镜节点 `p1_1`~`p3_3`（`asset.image`）**：为每一格写入具体的 `params.generateInstruction`，包含景别机位、角色动作神态、戏剧冲突，并在末尾统一加上「严格保持与参考图一致的角色形象与画风，纯画面不含任何文字与气泡」。

### 第 4 步：自动编排每张「漫画页」（`page1`~`page3`）的分格、斜切、镜头特效与台词气泡
在同一批 `graph_edit` 中，为每个 `comic.page` 节点构造并写入序列化后的 `params.comicPage` JSON 字符串：
- **重要：不要在 `panels` 里预填 `imageUrl`**（保持 `imageUrl` 缺省），这样当 `comic.page` 执行时，会自动按阅读顺序把上游刚生成的分镜图填入对应分格！
- **按剧情情绪自动搭配分格斜切与镜头特效**：
  - 斜切破格（`cornerOffsets`）：动作交锋或情绪突变格可设 8 元组顶点偏移 `[tlX, tlY, trX, trY, brX, brY, blX, blY]`（范围 `-0.35` ~ `0.35`）。
  - 镜头特效（`effectType`）：疾速冲刺/打斗用 `"speed-lines"`（速度线），爆发高潮/震惊特写用 `"focus-lines"`（集中线），回忆/悬念/压迫感用 `"vignette"`（暗角），复古漫感用 `"halftone"`（漫画网点）。
- **把剧本中的旁白、对白、心声、怒吼、拟声词自动转化为每格的 `bubbles` 数组**：
  - `shape: "narration"`（方形旁白框）、`"rounded"` / `"oval"`（标准圆角/椭圆对白框）、`"shout"`（爆炸锯齿咆哮框）、`"thought"`（云朵心声框）、`"sfx"`（漫画拟声词/音效字）；
  - `writingMode`: 默认 `"horizontal-tb"`（横排），日漫/古风漫画可设为 `"vertical-rl"`（竖排从右至左）。

### 第 5 步：一键运行整图出片（`task_run`）
- 调用 `task_run({ assetId })` 启动整图按拓扑序执行，并用 `task_status({ mcpTaskId })` 跟踪进度。

## 参数与 `comicPage` 完整 JSON 规范

节点端口、参数与 `comicPage` 的完整字段定义（分格、斜切、特效、气泡）见 `references/ports.md`。
