---
name: wf-ecom-ad-deep
description: 用「电商带货」工作流一次交付「白底棚拍主图锚点 + 场景/微距详情页套图 + A/B 投放变体矩阵 + 智能分层 PSD + 主图种草短视频」：电商卖点策略（copy）→ 商品棚拍主图锚点（hero，in-image 锁死商品外观）→ 痛点种草场景图（scene）+ 核心工艺微距图（closeup）+ A/B 投放变体矩阵（adVariants: image.adVariants）+ 主图 PSD 图层分离（split: image.layerSplit）+ 主图种草短视频（promoVideo: asset.video），并支持动态挂载多镜带货时间线（output.timeline）。
workflow: ecom-ad-deep
workflow-version: 2.1.0
argument-hint: "[商品名称/核心卖点/目标人群/促销利益点] [画幅比例默认 3:4 或 9:16]"
when-to-use: 用户要做淘宝/京东/小红书/抖音电商主图套图、详情页 KV、A/B 投放变体矩阵、PSD 图层分离可编辑电商素材以及种草短视频时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 电商带货全案 · 主图锚点 + A/B 变体矩阵 + 分层 PSD + 种草视频 SOP（ecom-ad-deep v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，8 节点 / 9 连线）

```
电商卖点与转化策略 (text.plain: copy)
  ├─out→in-text──→ 商品白底/棚拍主图·锚点 (asset.image: hero, 3:4)
  │                  ├─out→in-image──→ 痛点种草使用场景图 (asset.image: scene, 3:4)
  │                  │                   └─out→in-image──→ 主图种草短视频 (asset.video: promoVideo, 3:4, 5s)
  │                  ├─out→in-image──→ 核心工艺微距卖点图 (asset.image: closeup, 3:4)
  │                  ├─out→in────────→ A/B 投放变体矩阵 (image.adVariants: adVariants)
  │                  └─out→in────────→ 主图 PSD 图层分离 (image.layerSplit: split)
  └─out→in-text──────┴───────────────────┴─────────────────┘
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "ecom-ad-deep", mode: "seed" })` 物化后，画布节点 `id` 即为 `copy`、`hero`、`scene`、`closeup`、`adVariants`、`split`、`promoVideo`、`note`。

## 核心算子能力拆解

1. **一源多用，商品零走形（`hero` 锚点）**：
   - 先由 `hero` 生成高质感商业棚拍主图（或通过 `asset_import` 导入用户实拍商品图替换 `hero`），再通过 `in-image` 锁死 `scene` 与 `closeup` 的商品包装、瓶身比例与 Logo 色板。
2. **A/B 投放变体矩阵（`adVariants` · `image.adVariants`）**：
   - 接收 `hero:out`，按 `params.adVariantMatrix`（人群 × 场景 × 光影色调维度）批量裂变多张 A/B 测款图（参数结构详见 `references/ad-variants.md`）。
3. **美工级 PSD 图层分离（`split` · `image.layerSplit`）**：
   - 将主图拆解为独立商品前景层、背景层与可编辑文字层，导出标准分层 PSD（`out-psd`），方便设计师二次排版大促利益点。
4. **主图种草短视频（`promoVideo` · `asset.video`）**：
   - 以 `scene` 为首帧/参考图生成 3:4 或 9:16 动态主图视频，直接用于天猫/京东主图视频或小红书种草笔记。

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

Agent 应根据用户的具体投放渠道，用 `graph_edit` 自由裁剪或升级拓扑：

- **只做平面套图 / 详情页 KV**：若用户不需要视频或 PSD 拆层，可用 `node_delete` 删除 `promoVideo` 或 `split`。
- **升级为「多镜带货口播成片时间线」（Scale Up to Full Video Ad）**：
  当用户需要一条带金牌带货口播、卡点 BGM、促销花字的多镜抖音/快手带货短视频时，无需受限于单个 `promoVideo`，直接用 `graph_edit` 动态扩展：
  1. `node_upsert` 新增 `videoCloseup`（`asset.video`，接 `closeup:out → in-image` 拍微距实测镜头）、`voice`（`asset.voice`，写纯带货口播词）、`bgm`（`asset.music`）与 `timeline`（`output.timeline`）；
  2. `edge_connect` 将 `promoVideo:out` 与 `videoCloseup:out` 连入 `timeline:in`，`voice:out` 连入 `timeline:in-voice`，`bgm:out` 连入 `timeline:in-music`，`copy:out` 连入 `timeline:in-text`；
  3. 运行后调用 `timeline_edit` 编排「痛点暴击 → 微距实测 → 福利促单」三段式节奏与 `subtitleStyle: "karaoke-pop"` 爆款带货花字。
- **叠加数字人主播带货**：合入 `video.lipSync` 算子，让数字人主播手持商品口播讲解。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "ecom-ad-deep", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 写入 `copy` 的商品卖点策略，更新 `hero`、`scene`、`closeup`、`promoVideo` 的提示词，并配置 `adVariants` 的 `adVariantMatrix`（详见 `references/ad-variants.md`）。
3. 可先调用 `task_run({ assetId, nodeIds: ["hero"] })` 确认商品主图，再运行全图产出套图、变体矩阵、PSD 与种草视频。

## 端口与参数速查

详见 `references/ports.md` 与 `references/ad-variants.md`。
