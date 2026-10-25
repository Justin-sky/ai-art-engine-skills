---
name: wf-game-ua-video
description: 用「游戏买量」工作流把游戏卖点转化为工业级多镜竖屏买量成片：买量脚本（script）→ 角色与画风锚点（keyart，in-image 锁画风）→ 黄金3秒危机钩子（hookImg → hookVid）+ 核心爽点觉醒实机（coreImg → coreVid）+ 福利转化尾帧（ctaVid）+ 双段激情口播（voice1, voice2）+ 高燃卡点 BGM（bgm）→ 买量成片时间线（timeline: output.timeline）。
workflow: game-ua-video
workflow-version: 2.1.0
argument-hint: "[游戏品类/核心卖点/福利钩子] [画幅比例默认 9:16]"
when-to-use: 用户要制作手游/端游/小游戏的信息流买量投放视频、UA 转化素材或黄金三段式钩子广告短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 游戏买量 · 黄金三段式多轨转化成片 SOP（game-ua-video v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，12 节点 / 21 连线）

```
买量三段式脚本 (text.plain: script)
  ├─out→in-text──→ 角色与画风锚点 (asset.image: keyart, 9:16)
  │                  ├─out→in-image──→ 黄金3秒·危机钩子帧 (asset.image: hookImg) ──out→in-image──→ 黄金3秒·钩子视频 (asset.video: hookVid) ──────out→in───────┐
  │                  ├─out→in-image──→ 核心爽点·觉醒爆发帧 (asset.image: coreImg) ──out→in-image──→ 核心爽点·实机演示视频 (asset.video: coreVid) ──out→in───────┤
  │                  └─out→in-image──────────────────────────────────────────────────────────────→ 福利收尾·转化镜头 (asset.video: ctaVid) ─────────out→in───────┤
  ├─out→in-text──→ 钩子痛点口播 (asset.voice: voice1) ────────────────────────────────────────────────────────────────────────────────────────────out→in-voice─┤
  ├─out→in-text──→ 爽点与福利转化口播 (asset.voice: voice2) ──────────────────────────────────────────────────────────────────────────────────────out→in-voice─┤
  ├─out→in-text──→ 高燃卡点买量 BGM (asset.music: bgm) ───────────────────────────────────────────────────────────────────────────────────────────out→in-music─┤
  └─out→in-text───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 买量成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "game-ua-video", mode: "seed" })` 物化后，画布节点 `id` 即为 `script`、`keyart`、`hookImg`、`hookVid`、`coreImg`、`coreVid`、`ctaVid`、`voice1`、`voice2`、`bgm`、`timeline`、`note`。

## 游戏买量导演公式（3-Second Hook → Core Loop → CTA）

| 节点 Key | 买量职能 | 画面与参数覆写公式 |
|---|---|---|
| `keyart` (`asset.image`) | 视觉资产锁 | 锁定主角外观、装备材质与游戏渲染管线风格（UE5 暗黑写实 / 二次元赛璐璐 / 欧美卡通 SLG），连入下游 `in-image` 保证全片同一个人、同一画风。 |
| `hookImg` → `hookVid` | 黄金 3 秒痛点钩子 | 极具戏剧冲突的开局反差（如 Lv.1 新手被巨龙 Boss 逼入绝境 / 战力悬殊危机特写），急推镜头 + 强烈光影压迫感，瞬间抓住滑屏注意力。 |
| `coreImg` → `coreVid` | 核心玩法/爆装爽点 | 神装觉醒、全屏大招清怪、高爆率满地金色光柱或丝滑连招实机视角，高速环绕/震屏打击感运镜。 |
| `ctaVid` (`asset.video`) | 福利定格与转化 CTA | 顶级神装英雄霸气定格亮相、宝箱大爆特效，预留底部安全区承载「上线领百连抽 / 立即开玩」字幕引导。 |
| `voice1` / `voice2` (`asset.voice`) | 双段转化口播 | **铁律**：`params.generateInstruction` **直接写要被 TTS 朗读的买量台词正文**（如 `voice1`：「开局总是卡关被Boss秒？那是你没解锁这套隐藏神装流派！」；`voice2`：「上线直接领免费百连抽，全屏暴击一刀满级！」），严禁混入语气说明词。 |

## 动态算子伸缩与 `timeline_edit` 爆款买量精剪

1. **完整因果爽感弧线（拒绝拼凑）**：
   买量视频必须保持「受挫痛点（因）→ 意外奇遇/输入礼包码（转折）→ 战力暴涨秒杀 Boss（果）→ 福利号召（CTA）」的紧密因果链。
2. **动态增删算子（`graph_edit`）**：
   - **增加「假选择/失败操作」互动镜**：在 `hookVid` 与 `coreVid` 之间用 `node_upsert` 插入 `choiceImg` → `choiceVid`（展示玩家拉栓/二选一弹框选出神装），提升副玩法买量转化率；
   - **增加暴击/爆金币打击音效**：新增 `sfxHit`（`asset.sfx`，提示词 `"Crisp sword slash, massive explosion impact and golden coin shower sound effect"`）连入 `timeline:in-voice`；
   - **叠加游戏 UI 外框**：合入 `game-ui` 或 `svg.gen` 生成战力暴涨数字横幅叠入时间线。
3. **`timeline_edit` 高阶买量包装**：
   - 调用 `timeline_edit` 开启 `subtitleStyle: "karaoke-pop"`（黄底黑边大字逐词弹跳）；
   - 对 `voice1`（受挫吐槽）或 Boss 咆哮配音设置 `pitchSemitones` / `voiceEffect`（如 `"demon"` 魔王低音或 `"megaphone"` 战报广播音），拉满听觉反差。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "game-ua-video", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 将游戏买量三段式脚本写入 `script 的 params.`，将纯口播台词写入 `voice1` 与 `voice2` 的 `params.generateInstruction`，并定制 `keyart`、`hookImg`、`hookVid`、`coreImg`、`coreVid`、`ctaVid`、`bgm` 的提示词。
3. `task_run({ assetId })` 执行全图生成，在 `timeline` 合成买量成片；可追加 `timeline_edit` 精修卡点与花字。

## 端口与参数速查

详见 `references/ports.md`。
