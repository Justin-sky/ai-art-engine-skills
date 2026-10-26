---
name: wf-comic-publish
description: 用「漫画出版」工作流把一段剧本自动变成一集多页漫画（script 剧本备忘 → charAnchor 角色与画风定妆锚点 → 多页按格比例生图 p1_1..p3_3 → page1..page3 漫画页自动排版、斜切破格、速度线/集中线与台词气泡/音效字合成 PNG），支持日漫黑白网点（右到左竖排）、韩系手机长条漫（Webtoon）、国漫全彩页漫及自由增删页数/格数。当用户要做一集漫画、多页连载条漫/日漫、四格漫画或输入剧本自动生成带气泡的漫画成片时加载本技能。
workflow: comic-publish
workflow-version: 1.2.0
argument-hint: "[漫画剧本/故事梗概/画风（日漫黑白网点/韩系全彩条漫/国漫古风/美漫）] [页数与格数]"
when-to-use: 用户要做一集漫画、多页连载条漫/日漫、四格漫画或输入一段剧本自动生成带台词气泡的漫画成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 漫画出版 · 整集漫画自动生产与专业排版 SOP（comic-publish v1.2，默认 16 节点 / 20 连线）

## 默认种子拓扑与产出物（Seed Skeleton）

```
漫画剧本 (text.plain: script，独立剧本备忘节点，不连入各格生图节点的 in-text 以防多格文字与台词引号污染单格画面)
角色与画风定妆 (asset.image: charAnchor，输出参考图连入全集 10 格分镜的 in-image 端口，锁定角色外貌与统一画风)
  ├─out→in-image──→ 第 1 页（起承）：p1_1 (16:9), p1_2 (2:3), p1_3 (2:3) ──out→in-image──→ 漫画页·第 1 页 (comic.page: page1)
  ├─out→in-image──→ 第 2 页（高潮）：p2_1 (16:9), p2_2 (1:1), p2_3 (1:1), p2_4 (16:9) ──out→in-image──→ 漫画页·第 2 页 (comic.page: page2)
  └─out→in-image──→ 第 3 页（收尾）：p3_1 (1:1), p3_2 (1:1), p3_3 (4:3) ──out→in-image──→ 漫画页·第 3 页 (comic.page: page3)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "comic-publish", mode: "seed" })` 物化后，画布上的节点 `id` 与 `workflow.json` 里的 `key` **完全一致**（`script`、`charAnchor`、`p1_1`~`p1_3`、`page1`、`p2_1`~`p2_4`、`page2`、`p3_1`~`p3_3`、`page3`、`note`），可直接用于 `graph_edit`！
> **严禁把 `script` 连入分镜生图节点的 `in-text`**：含多格描述与引号对白的整集剧本若连入 `asset.image:in-text`，会导致生图模型把单格画成多格分镜并烘入乱码文字气泡。每格画面指令只写在各自的 `params.generateInstruction` 中！

---

## 核心能力：用户输入一段剧本 → AI 全自动创建一集漫画 SOP

当用户选中本工作流并输入一段剧本（或故事梗概）时，请直接按以下 5 步全自动完成整集漫画的编排与出片：

### 第 1 步：落盘工作流种子骨架
调用 `workflow_use_installed({ id: "comic-publish", mode: "seed" })` 落盘工作流资产，拿到 `assetId`。

### 第 2 步：确定漫种规格与「起承转合」页数/格数（支持动态增删）
先根据用户指定的题材或漫种，选择对应的页面与分镜配方：

| 漫种模式 | `comicPage` 页面级参数配方 | 分格与气泡推荐设定 |
|---|---|---|
| **① 国漫/美漫全彩页漫（默认）** | `width: 1080, height: 1520, columns: 6, rows: 6, readingDirection: "ltr", backgroundColor: "#ffffff"` | 气泡 `writingMode: "horizontal-tb"`，按分格长宽比设 `generateAspectRatio` |
| **② 日式黑白网点漫画（Manga）** | `width: 1080, height: 1520, columns: 6, rows: 6, readingDirection: "rtl", backgroundColor: "#ffffff"` | 每格设 `imageFilter: "manga-bw"`（回忆/情绪格设 `"halftone"`），气泡设 `writingMode: "vertical-rl", shape: "oval"` |
| **③ 韩系手机竖屏长条漫（Webtoon）** | `width: 1080, height: 2400, columns: 1, rows: 8, gutter: 36, padding: 40, readingDirection: "ltr"` | 单列纵向流式排布（每格 `col: 0, colSpan: 1, rowSpan: 2`），生图统一用 `3:4` 或 `16:9` |
| **④ 单页四格漫画（4-Koma / 四宫格）** | 用 `graph_edit` 删掉 `p2_*`、`page2`、`p3_*`、`page3`，仅保留 `page1` 并连入 4 个分镜节点 | 四格竖排用 `columns: 1, rows: 4, width: 900, height: 2200`；对称四宫格用 `columns: 2, rows: 2, width: 1080, height: 1440` |

### 第 3 步：更新剧本、角色定妆锚点与各格画面指令（严格对齐分格宽高比）
在一次 `graph_edit` 批量操作中同时更新：
1. **`script`（`text.plain`）**：把整理好的分场分格剧本写入 `params.text`（仅作画布文档备忘，不连线）。
2. **`charAnchor`（`asset.image`）**：在 `params.generateInstruction` 中写明主角（若对手戏激烈，可同框画出主角与宿敌双人设定集）的外貌、发型、服饰与画风基准，设 `generateAspectRatio: "16:9"`，强调「整张图只画角色设定图，禁止分格边框、禁止任何对话框气泡与文字」。
3. **全部分镜节点 `p1_1`~`p3_3`（`asset.image`）**：
   - **必须按该分格在页面网格中的几何长宽比设置 `generateAspectRatio`**，防止 `imageFit: "cover"` 裁掉角色头脚：
     - 横跨全宽横幅格（`rowSpan: 2, colSpan: 6`，如 `p1_1`, `p2_1`, `p2_4`）→ **`"16:9"`**
     - 半宽竖向高格（`rowSpan: 4, colSpan: 3`，如 `p1_2`, `p1_3`）→ **`"2:3"`**（或 `"3:4"`）
     - 半宽中段方格（`rowSpan: 2, colSpan: 3`，如 `p2_2`, `p2_3`, `p3_1`, `p3_2`）→ **`"1:1"`**
     - 全宽大跨幅收尾格（`rowSpan: 4, colSpan: 6`，如 `p3_3`）→ **`"4:3"`**
   - 每格 `generateInstruction` 写明景别机位、动作张力、光影氛围，末尾统一注明：「严格保持与参考图一致的角色形象与画风；整张图只画这一幅独立分镜画面，禁止绘制漫画分格线、禁止任何台词气泡与文字」。

### 第 4 步：自动编排每张「漫画页」（`page1`~`page3`）的分格、斜切、镜头特效与台词气泡
在同一批 `graph_edit` 中，为每个 `comic.page` 节点写入 `params.comicPage` JSON 字符串：
- **不要预填 `imageUrl`**（保持缺省），以便 `comic.page` 执行时自动按阅读顺序填入上游刚生成的分镜图。
- **按剧情情绪搭配斜切与镜头特效**：
  - 斜切破格（`cornerOffsets`）：8 元组顶点偏移 `[tlX, tlY, trX, trY, brX, brY, blX, blY]`（范围 `-0.35 ~ 0.35`）。
  - 镜头氛围特效（`effectType`）：疾速冲刺用 `"speed-lines"`（速度线），高潮爆发/特写用 `"focus-lines"`（集中线），悬念压迫用 `"vignette"`（暗角），复古网点边框用 `"halftone-frame"`（注意：整图网点化请用滤镜 `imageFilter: "halftone"` 或 `"manga-bw"`）。
- **把剧本中的旁白、对白、心声、怒吼、拟声词转化为每格的 `bubbles` 数组**：
  - `shape`: `"narration"`（方形旁白框）、`"rounded"` / `"oval"`（圆角/椭圆对白框）、`"shout"`（爆炸咆哮框）、`"thought"`（云朵心声框）、`"sfx"`（漫画拟声词大字）；
  - 合理设置气泡锚点 `x, y`（通常置于分镜上方或角色侧上方 `y: 0.18 ~ 0.32`，避免遮挡角色面部中心）及指向说话人嘴部的 `tailTipX, tailTipY`。

### 第 5 步：一键运行整图出片（`task_run`）
- 直接调用一次 `task_run({ assetId })` 即可按拓扑序全自动跑完：角色定妆 → 并发生成 10 格分镜图 → 自动排版合成 3 页带气泡与特效的高清漫画页 PNG！

## 参数与 `comicPage` 完整 JSON 规范

节点端口、参数与 `comicPage` 的完整字段定义（分格、斜切、滤镜、特效、气泡）见 `references/ports.md`。
