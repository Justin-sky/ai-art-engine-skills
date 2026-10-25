---
name: wf-music-mv
description: 用「音乐 MV 短片」工作流从一段歌词与歌曲意境脚本（script）出发，同步生成 AI 原创主题音乐（bgm: asset.music）、锁定主角与光影色调锚点（charAnchor: asset.image），并拍摄主歌叙事（shot1 → vid1）、副歌高潮（shot2 → vid2）、尾奏余韵（shot3 → vid3）三幕电影级视频，最终在 MV 成片时间线（timeline: output.timeline）完成多轨音画合成与歌词字幕对位。
workflow: music-mv
workflow-version: 1.1.0
argument-hint: "[歌曲主题/歌词/曲风流派] [画幅比例 16:9 或 9:16]"
when-to-use: 用户想要制作音乐 MV、Suno/AI 歌曲视觉短片、情绪卡点短片或带原创音乐与歌词字幕的多镜头概念成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 音乐 MV 短片 · 情绪光影与多轨音画合成 SOP（music-mv v1.1）

## 默认种子拓扑与交付物（Seed Skeleton，11 节点 / 19 连线）

```
歌曲意境与 MV 分幕脚本 (text.plain: script)
  ├─out→in-text──→ MV 主角与色调锚点 (asset.image: charAnchor, 16:9)
  │                  ├─out→in-image──→ 第一幕·主歌氛围关键帧 (asset.image: shot1) ──out→in-image──→ 第一幕·主歌慢镜视频 (asset.video: vid1) ──out→in───────┐
  │                  ├─out→in-image──→ 第二幕·副歌高潮关键帧 (asset.image: shot2) ──out→in-image──→ 第二幕·副歌律动视频 (asset.video: vid2) ──out→in───────┤
  │                  └─out→in-image──→ 第三幕·尾奏余韵关键帧 (asset.image: shot3) ──out→in-image──→ 第三幕·尾奏收束视频 (asset.video: vid3) ──out→in───────┤
  ├─out→in-text──→ 原创主题音乐 BGM/歌曲 (asset.music: bgm) ──────────────────────────────────────────────────────────────────────────────────out→in-music─┤
  └─out→in-text───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ MV 成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "music-mv", mode: "seed" })` 物化后，节点 `id` 即为 `script`、`charAnchor`、`bgm`、`shot1`、`shot2`、`shot3`、`vid1`、`vid2`、`vid3`、`timeline`、`note`。

## 导演分镜与音乐提示词规范

| 节点 Key | 角色定位 | 提示词与参数覆写公式 |
|---|---|---|
| `script` (`text.plain`) | 歌词与分幕母本 | 写入按句分行的核心歌词与三幕视觉意境说明，通过 `out → timeline:in-text` 自动铺排底部歌词字幕轨。 |
| `charAnchor` (`asset.image`) | 主角与胶片调色锚点 | 公式：`主角面部骨相与穿搭设定 + 电影胶片颗粒感（35mm Kodak Vision3） + 主副色温对比（如冷青霓虹 vs 暖金逆光） + 纯画面无文字`。连入 `shot1`~`shot3` 的 `in-image`，杜绝跨镜头换脸。 |
| `bgm` (`asset.music`) | 原创曲风与配器 | 公式：`曲风流派（Synthwave / 独立摇滚 / 国风电子 / 抒情钢琴） + BPM 速度 + 乐器编制 + 情绪推进（主歌克制→副歌宏大→尾奏空灵）`（若需纯伴奏设 `generateMusicInstrumental: true`；若需带人声演唱设 `false` 并写入歌词）。 |
| `shot1` / `shot2` / `shot3` | 三段情绪景别递进 | **主歌（`shot1`）**：中远景孤独氛围，静态构图；**副歌（`shot2`）**：中近景动态张力，强烈光影对比与粒子氛围；**尾奏（`shot3`）**：大远景背影或空镜留白，情绪释怀。 |
| `vid1` / `vid2` / `vid3` | 运镜节奏控制 | `vid1` 用 `极慢横移或缓慢推镜`；`vid2` 用 `环绕运镜 orbit + 慢动作发丝衣角飞扬 + 光斑闪烁`；`vid3` 用 `缓慢拉远固定淡出`。全部设 `generateAudio: false`，由 `bgm` 独占音轨。 |

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

1. **按歌曲段落自由扩镜（Verse → Pre-Chorus → Chorus → Bridge → Outro）**：
   - 默认 3 幕仅为种子骨架。若用户要做 30~60s 完整副歌 MV，可用 `graph_edit` 的 `node_upsert` 新增 `shot4`~`shot6` 与 `vid4`~`vid6`，全部从 `charAnchor:out` 接入 `in-image` 并把 `vid4`~`vid6:out` 按叙事顺序连入 `timeline:in`。
2. **插入主唱对口型镜头（`video.lipSync` 混搭）**：
   - 想让主角在副歌高潮处直视镜头深情演唱？可新增 `vocalLine`（`asset.voice` 或导入干声）+ `lipSyncChorus`（`video.lipSync`），将 `vid2:out` 接 `lipSyncChorus:in-video`，输出连入 `timeline:in`。
3. **利用 `timeline_edit` 精细卡点与转场**：
   - 运行完成后调用 `timeline_edit` 设置 `videoTransition: "dissolve" | "flash" | "zoomin"`，或在副歌重拍处切片加快剪辑密度。

## 标准执行步骤（MCP 自动化路径）

1. `workflow_use_installed({ id: "music-mv", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 将歌词与分幕脚本写入 `script 的 params.`，将曲风配器写入 `bgm 的 params.`，并将分镜提示词写入 `charAnchor`、`shot1`~`shot3`、`vid1`~`vid3`（并根据歌曲结构按需扩充镜头）。
3. `task_run({ assetId })` 执行全图生成，完成后可通过 `timeline_edit` 精调歌词字幕与转场并导出成片。

## 端口与参数速查

详见 `references/ports.md`。
