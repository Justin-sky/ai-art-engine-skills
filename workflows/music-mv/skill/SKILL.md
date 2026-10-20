---
name: wf-music-mv
description: 用「音乐 MV 短片」工作流从一段歌词/情绪企划出发，同步生成 AI 原创音乐（asset.music）、锁定主角与光影美学锚点（in-image），并拍摄主歌叙事、副歌爆发、尾奏意境三幕电影级视频，最终在成片时间线（output.timeline）自动完成音画合成与歌词字幕对位。
workflow: music-mv
workflow-version: 1.0.0
argument-hint: "[歌曲主题/歌词/曲风流派] [画幅比例 16:9 或 9:16]"
when-to-use: 用户想要制作音乐 MV、Suno/AI 歌曲视觉短片、情绪卡点短片或带原创音乐与歌词字幕的多镜头概念成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 音乐 MV 短片 · 情绪光影与多轨音画合成 SOP（music-mv）

## 拓扑架构与交付物

```
歌词与MV视觉企划 (text.plain)
  ├─out→in-text──→ 主角与视觉美学锚点 (asset.image)
  │                  ├─out→in-image──→ 主歌·叙事铺垫关键帧 (asset.image) ──out→in-image──→ 主歌·叙事视频 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 副歌·情绪爆发关键帧 (asset.image) ──out→in-image──→ 副歌·高潮视频 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 尾奏·余韵收束关键帧 (asset.image) ──out→in-image──→ 尾奏·意境视频 (asset.video) ──out→in-clips─┤
  ├─out→in-text──→ 主题音乐生成 (asset.music) ──────────────────────────────────────────────────────────────────────out→in-music─────┤
  └─out→in-subtitles─────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 导演分镜与音乐提示词规范

| 阶段节点 | 角色定位 | 提示词与参数覆写公式 |
|---|---|---|
| `lyrics` (`text.plain`) | 歌词字幕源 | 写入按句分行的核心歌词（每行一句，不带 `[Verse]` 等结构标签以免出现在屏幕字幕上），直接流入 `output.timeline` 的 `in-subtitles` 自动铺排歌词字幕轨。 |
| `visualAnchor` (`asset.image`) | 主角与胶片调色锚点 | 公式：`主角面部骨相与穿搭设定 + 电影胶片颗粒感（35mm Kodak Vision3） + 主副色温对比（如冷青霓虹 vs 暖金逆光） + 干净无文字`。连入三幕关键帧的 `in-image`，杜绝跨镜头换脸。 |
| `musicTrack` (`asset.music`) | 原创曲风与配器 | 公式：`曲风流派（Synthwave / 独立摇滚 / 国风电子 / 抒情钢琴） + BPM 速度 + 乐器编制 + 情绪推进（主歌克制→副歌宏大→尾奏空灵）`。 |
| `verseImg` / `chorusImg` / `outroImg` | 三段情绪景别递进 | **主歌（Verse）**：中远景孤独氛围，静态构图；**副歌（Chorus）**：中近景动态张力，强烈光影对比与粒子氛围；**尾奏（Outro）**：大远景背影或空镜留白，情绪释怀。 |
| `verseVid` / `chorusVid` / `outroVid` | 运镜节奏控制 | 主歌用 `极慢横移或缓慢推镜`；副歌用 `环绕运镜 orbit + 慢动作发丝衣角飞扬 + 光斑闪烁`；尾奏用 `缓慢拉远固定淡出`。全部设置 `generateAudio: false`，由 `musicTrack` 独占音轨。 |

## 标准执行步骤（MCP 自动化路径）

1. `workflow_use_installed({ id: "music-mv", name: "<歌曲名>·MV短片", generateAspectRatio: "16:9" })`。
2. `graph_read({ assetId, includeParams: true })` 拿到所有节点 ID。
3. `graph_edit` 将歌词写入 `lyrics` 的 `params.text`，将曲风配器写入 `musicTrack` 的 `params.generateInstruction`，并将三段分镜提示词分别写入 `visualAnchor` 及三组图文/视频节点。
4. `task_run({ assetId })` 执行全图生成，完成后引导用户双击 `output.timeline` 节点预览 MV 成片并导出。

## 端口与参数速查

详见 `references/ports.md`。
