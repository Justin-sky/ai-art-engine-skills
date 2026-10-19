---
name: wf-ecom-ad-deep
description: 用「电商带货全案」工作流一次交付「A/B 投放平面矩阵 + 智能分层 PSD + 三镜带货成片」：商品卖点脚本→白底标准主图锚点（in-image 锁商品外观）→同时衍生 3 路不同受众场景 KV、可编辑图层分离（image.layerSplit），以及「痛点暴击 → 沉浸功能实测 → 促销转化尾帧」三镜视频+金牌带货口播+卡点BGM→成片时间线（output.timeline）。
workflow: ecom-ad-deep
workflow-version: 2.0.0
argument-hint: "[商品名称/核心卖点/目标人群/促销利益点] [画幅比例默认 9:16]"
when-to-use: 用户要做抖音/快手/小红书/淘宝电商带货短视频、商品详情页 KV 矩阵、A/B 测图以及图层分离可编辑电商素材时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 电商带货全案 · 平面变体矩阵 + 图层分离 + 三镜转化成片 SOP（ecom-ad-deep v2.0）

## 拓扑架构与交付物

```
商品卖点与带货口播稿 (play.script)
  ├─out→in-text──→ 商品标准主图锚点 (asset.image)
  │                  ├─out→in-image──→ 变体A·都市白领通勤KV (asset.image) ──out→in──→ 主KV智能分层PSD (image.layerSplit)
  │                  ├─out→in-image──→ 变体B·精致居家治愈KV (asset.image)
  │                  ├─out→in-image──→ 变体C·潮流户外社交KV (asset.image)
  │                  ├─out→in-image──→ 视频镜1·痛点对比开场 (asset.image) ──out→in-image──→ 镜头1·痛点引入视频 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 视频镜2·硬核卖点实测 (asset.image) ──out→in-image──→ 镜头2·卖点实测视频 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 视频镜3·优惠促单尾帧 (asset.image) ──out→in-image──→ 镜头3·促单转化视频 (asset.video) ──out→in-clips─┤
  ├─out→in-text──→ 带货种草口播配音 (asset.voice) ────────────────────────────────────────────────────────────────────────────out→in-voice────┤
  ├─out→in-text──→ 带货节奏卡点BGM (asset.music) ─────────────────────────────────────────────────────────────────────────────out→in-music────┤
  └─out→in-subtitles────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 核心设计亮点

1. **一源多用，商品零走形**：以 `productAnchor`（商品主图锚点）锁定商品包装与工业细节，同时向「3 路人群 A/B 测图矩阵」和「3 镜短视频流水线」供图。
2. **平面 + 分层 PSD 一步到位**：`variantA` 自动接入 `image.layerSplit`，拆出独立前景商品透明层与背景层，方便美工在画布内二次排版促销花字。
3. **高转化三镜短视频闭环**：痛点开场抓留存 → 微距实测立信任 → 福利尾帧促下单，配合种草口播与卡点 BGM 在 `output.timeline` 直接合成带字幕成片。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "ecom-ad-deep", name: "<商品名>·电商带货全案", generateAspectRatio: "9:16" })`。
2. `graph_read({ assetId, includeParams: true })` 读取节点 ID。
3. `graph_edit` 覆写 `brief`（三段带货字幕稿）、`productAnchor`、3 路人群变体 `variantA`~`variantC`、3 镜视频关键帧与动态指令，以及 `voiceover` 的纯朗读带货台词。
4. `task_run({ assetId })` 执行全图生成，同时收获平面 KV 矩阵、分层图层与时间线带货成片。

## 端口与参数速查

详见 `references/ports.md`。
