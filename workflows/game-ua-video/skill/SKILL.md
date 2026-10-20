---
name: wf-game-ua-video
description: 用「游戏买量」工作流把游戏卖点转化为工业级三镜竖屏买量成片：卖点脚本→游戏主视觉/主角锚点（in-image 锁画风）→黄金前3秒痛点钩子+中段核心玩法高潮+尾帧福利转化三镜关键帧与图生视频+激昂买量口播（asset.voice）+卡点战歌 BGM（asset.music）→成片时间线（output.timeline）自动合成。
workflow: game-ua-video
workflow-version: 2.0.0
argument-hint: "[游戏品类/核心卖点/福利钩子] [画幅比例默认 9:16]"
when-to-use: 用户要制作手游/端游/小游戏的信息流买量投放视频、UA 转化素材或三段式黄金钩子广告短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 游戏买量 · 黄金三段式多轨转化成片 SOP（game-ua-video v2.0）

## 拓扑架构与交付物

```
买量脚本与字幕 (text.plain)
  ├─out→in-text──→ 主角与美术风格锚点 (asset.image)
  │                  ├─out→in-image──→ 镜头1·黄金3秒钩子图 (asset.image) ──out→in-image──→ 镜头1·黄金3秒钩子视频 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 镜头2·核心爽点实机图 (asset.image) ──out→in-image──→ 镜头2·核心爽点实机视频 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 镜头3·福利转化尾帧图 (asset.image) ──out→in-image──→ 镜头3·福利转化尾帧视频 (asset.video) ──out→in-clips─┤
  ├─out→in-text──→ 买量激情口播配音 (asset.voice) ──────────────────────────────────────────────────────────────────────────────out→in-voice────┤
  ├─out→in-text──→ 高燃节奏卡点 BGM (asset.music) ──────────────────────────────────────────────────────────────────────────────out→in-music────┤
  └─out→in-subtitles──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 游戏买量导演公式（3-Second Hook → Core Loop → CTA）

| 镜头阶段 | 买量职能 | 画面与运镜公式 |
|---|---|---|
| `heroAnchor` (`asset.image`) | 视觉资产锁 | 锁定主角外观、装备材质与游戏渲染管线风格（UE5 暗黑写实 / 二次元赛璐璐 / 欧美卡通 SLG），连入三镜 `in-image` 保证全片同一个人、同一画风。 |
| **镜头 1（0~5s）** `hookImg` → `hookVid` | 黄金 3 秒痛点钩子 | 极具戏剧冲突的开局反差（如 1 级新手被 Boss 逼入绝境 / 战力悬殊危机特写），急推镜头 + 强烈光影压迫感，瞬间抓住滑屏注意力。 |
| **镜头 2（5~10s）** `climaxImg` → `climaxVid` | 核心玩法/爆装爽点 | 神装觉醒、全屏大招清怪、高爆率满地金色光柱或丝滑连招实机视角，高速环绕/震屏打击感运镜。 |
| **镜头 3（10~15s）** `ctaImg` → `ctaVid` | 福利定格与转化 CTA | 顶级神装英雄霸气定格亮相、宝箱大爆特效，预留底部安全区承载「开局送百抽 / 立即下载」字幕引导。 |
| `voiceover` (`asset.voice`) | 转化口播 | **注意**：`generateInstruction` 直接写要朗读的买量台词正文（如「开局只有一把木剑？输入礼包码直接爆终极神装！」），不要写语气说明文字。 |

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "game-ua-video", name: "<游戏名>·买量视频", generateAspectRatio: "9:16" })`。
2. `graph_read({ assetId, includeParams: true })` 读取节点列表。
3. `graph_edit` 将具体游戏品类、3 段字幕脚本写入 `script`，将纯口播台词写入 `voiceover`，并定制 `heroAnchor` 与三镜图文/视频节点的 `generateInstruction`。
4. `task_run({ assetId })` 执行全图生成，在 `output.timeline` 自动汇集三镜视频、口播、战歌 BGM 与字幕输出成片。

## 端口与参数速查

详见 `references/ports.md`。
