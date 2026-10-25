---
name: wf-commerce-semantic-variants
description: 用「带货视频拆解与重制」工作流把口播/卖货实拍视频逆向拆解成结构化时间线，按镜头 / 台词 / 实体做针对性修改，再经重制规划、视频重制合成出新成片（未改的镜头直通原片，改过的镜头重新生成，改过的台词重新配音）。当用户要复刻带货视频、改某几镜、换人换商品、改台词重配音时加载本技能。
workflow: commerce-semantic-variants
workflow-version: 1.4.0
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, semantic_analyze_video, semantic_timeline_read, asset_list, asset_import, task_run, task_status
---

# 带货视频拆解与重制（commerce-semantic-variants）

> **id 说明**：`commerce-semantic-variants` 是**历史安装身份** —— 技能名
> `wf-commerce-semantic-variants`、已安装目录与市场条目都按它对应，重命名会破坏已安装用户的
> 升级路径。标题与摘要已更新为「带货视频拆解与重制」，但 id 保持不动。

## 这条工作流产出什么

`原视频 → 视频拆解（结构化时间线 @2）→ 修改 → 重制规划（脏检测 + 提示词方案）→ 视频重制 → 成片 mp4`

核心流水线：先对原片做拉片拆解得到结构化时间线，再比对拆解基线规划出"哪些镜头与台词要重做"，最后局部重制合成成片。
**零修改时成片 = 原片按镜头重新拼接（还原原片），一次模型都不调。**

- **视频拆解**（`semantic.analyze`）：切镜、关键帧、转写、人声分离、实体；产出
  `aiart.semantic-timeline@2`：结构化镜头树（镜头语言 / 画面 / 主体动作 / 原片入出点）、
  台词表、文字层、实体，并打**基线指纹**。旧的 `@1` 文档读入时自动迁移。
- **修改**：双击拆解节点进时间线，下方「修改」面板里按镜头改主体动作、景别 / 角度 / 运镜、
  画面描述 / 风格 / 光影 / 调色、提示词覆盖、删镜；改台词；改实体名或换参考资产（换人 / 换商品）。
  改动直接存进 `Semantic/<id>/timeline.json`（带 `edits` 审计）；重新拆解会保留这些修改。
- **重制规划**（`semantic.compile`）：与基线比对，给每镜标 `passthrough` / `regenerate`
  （带原因），拼出每镜提示词方案（结构化字段优先），并列出要重新配音的台词与新字幕
  （`aiart.video-prompts@1`）。双击节点打开指令面板：选文本模型、开「用模型写提示词」、
  选导演规则包、写规划要求（追加到每镜提示词）。单镜提示词覆盖在拆解节点的「修改」面板里写
  （覆盖 = 该镜重新生成）；旧图里的 `promptOverridesJson` 仍兼容生效。
- **视频重制**（`semantic.rebuild`）：**只输出视频**。双击节点打开指令面板选视频模型、
  分辨率 / 画幅，并可写一句整片补充指令（追加到每个重新生成镜头的提示词）。
  - 未改镜头直通原片段；改过的镜头用首帧图生视频（换了参考资产时改用参考图），
    生成片段对齐到原镜头时长；生成失败的镜头退回原片段；
  - 改过的台词逐句 TTS 重新配音（可用原人声做声音克隆），有人声分离时只替换人声、保留背景声；
  - 「全部镜头重新生成」开关 = 风格重制；`semanticExecute` 关掉则一次模型都不调，只出清单；
  - 成片预览与清单在右侧面板；文件落在 `Semantic/<id>/rebuilds/<runId>/`。

随包 `semanticPacks/`（纯 JSON，无可执行代码）：

| 文件 | 用途 |
|---|---|
| `commerce-vocabulary.json` | 带货节拍词表 `commerce.v1` |
| `commerce-director-rules.json` | 抖音快节奏导演规则 |
| `commerce-director-persona.json` | 带货导演人格提示 |

## 什么时候用 / 什么时候别用

用：30–90 秒**实拍口播/卖货**，要「拆解 → 改几处 → 重制成片」，或整片风格重制。

别用：

| 用户其实想要 | 该用 |
|---|---|
| 从零生成电商主视觉+变体图 | `ecom-ad-deep` |
| 产品主图+短视频广告（无实拍拆解） | `product-ad` |

## 上游输入怎么给

**必填**：一段可分析的视频资产，接到「视频拆解」的视频入端口。

**建议**：先在设置里装好 ASR（词级转写）、视觉大模型（实体 / 镜头描述）、语音合成（重新配音）。
缺模型时会启发式降级，但仍能出骨架时间线；没有语音合成时改过的台词只更新字幕。

## Agent 操作要点

1. 用 MCP `semantic_analyze_video` 或跑通图上「视频拆解」节点。
2. 用 `semantic_timeline_read` 核对镜头树与事件；改动通过保存时间线写回（字段改完要追加一条
   `edits` 记录，`origin: "agent"`）。
3. 跑「重制规划」：日志里会写"几镜需重新生成 · 几镜删除 · 几句需重新配音"。
4. 跑「视频重制」出成片。**先用 `semanticExecute=false` 看清单（生成几镜、直通几镜、配音几句）再开**；
   零修改时不会调用视频模型。
5. 人格与规则来自本包 JSON；不要在市场包里塞可执行节点代码。

## 动态算子伸缩与跨技能混搭（Seed & Dynamic Composition）

- **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "commerce-semantic-variants", mode: "seed" })` 物化后，画布节点 `id` 即为 `analyze`（`semantic.analyze`）、`compile`（`semantic.compile`）、`rebuild`（`semantic.rebuild`）、`note`。
- **叠加时间线二次包装**：若需对 `rebuild` 输出的重制成片（`out-video`）追加花字弹窗、BGM 混音或音效卡点，可用 `graph_edit` 新增 `timeline`（`output.timeline`）并将 `rebuild:out-video → timeline:in` 连接，随后调用 `timeline_edit` 精修。
