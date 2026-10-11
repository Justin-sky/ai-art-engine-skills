---
name: wf-commerce-semantic-variants
description: 用「带货语义拆解与重建」工作流把口播/卖货实拍视频拆成语义时间线，编译成每镜视频生成提示词，再重建出成片。当用户要复刻带货视频、按提示词重生成镜头、或在原片基础上重建时加载本技能。
workflow: commerce-semantic-variants
workflow-version: 1.1.0
---

# 带货语义拆解与重建（commerce-semantic-variants）

> **id 说明**：`commerce-semantic-variants` 是**历史安装身份** —— 技能名
> `wf-commerce-semantic-variants`、已安装目录与市场条目都按它对应，重命名会破坏已安装用户的
> 升级路径。标题与摘要已更新为「带货语义拆解与重建」，但 id 保持不动。

## 这条工作流产出什么

`口播卖货视频 → 语义时间线（镜头/事件/节拍/意图）→ 每镜视频生成提示词 → 重建出成片 mp4`

- **语义分析**（`semantic.analyze`）：切镜、话语、实体与启发式事件/节拍；落 `Semantic/<id>/`。
- **语义编译**（`semantic.compile`）：把时间线编译成**视频生成提示词计划**
  （`aiart.video-prompts@1`，从提示词口输出）：每镜一条提示词 + 该镜首帧（来自分析证据的关键帧）。
  - 默认**规则拼装**（实体主体 → 事件动作 → 台词 → 运镜/光影，确定性、零成本）；
  - 节点上的「用模型写提示词」打开后，先用文本模型把画面说清楚，**最终仍走规则拼装**；
  - 可按 `shotId` 人工覆盖单镜提示词（`promptOverridesJson`）。
- **语义重建**（`semantic.rebuild`）：按提示词生成并拼接出成片。
  - `shots` 逐镜头（每镜一次生成，首帧走**图生视频**）或 `full` 整片一次（一条长提示词）；
  - 每镜上限默认 12；`semanticExecute` 关掉则**一次模型都不调**，只出计划；
  - 拼接时可选**保留原片音轨**、**烧录转写字幕**；成片与清单落在
    `Semantic/<id>/rebuilds/<runId>/`。

随包 `semanticPacks/`（纯 JSON，无可执行代码）：

| 文件 | 用途 |
|---|---|
| `commerce-vocabulary.json` | 带货节拍词表 `commerce.v1` |
| `commerce-director-rules.json` | 抖音快节奏导演规则 |
| `commerce-director-persona.json` | 带货导演人格提示 |

## 什么时候用 / 什么时候别用

用：30–90 秒**实拍口播/卖货**，要「拆解 → 提示词 → 重建成片」。

别用：

| 用户其实想要 | 该用 |
|---|---|
| 从零生成电商主视觉+变体图 | `ecom-ad-deep` |
| 产品主图+短视频广告（无实拍拆解） | `product-ad` |
| 只在原片上做局部裁剪/重排（不重生成） | 直接编辑成片时间线 |

## 上游输入怎么给

**必填**：一段可分析的视频资产，接到「语义分析」的视频入端口。

**建议**：先在设置里装好 ASR（词级转写）；实体检测走视觉大模型。缺模型时会启发式降级，
但仍能出骨架时间线（提示词的画面描述会弱一些）。

## Agent 操作要点

1. 用 MCP `semantic_analyze_video` 或跑通图上「语义分析」节点。
2. 用 `semantic_timeline_read` 核对事件是否带证据；无证据事件会被丢弃。
3. 跑「语义编译」拿到提示词计划（`out` 是 `aiart.video-prompts@1` JSON）；
   要改某一镜就写 `promptOverridesJson`（`{"shotId": "提示词"}`）。
4. 跑「语义重建」出成片。**逐镜模式会按镜头数调用视频模型 —— 先用 `semanticExecute=false`
   看清单（生成几次、跳过几镜）再开**。
5. 人格与规则来自本包 JSON；不要在市场包里塞可执行节点代码。
