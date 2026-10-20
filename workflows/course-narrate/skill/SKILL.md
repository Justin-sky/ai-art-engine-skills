---
name: wf-course-narrate
description: 用「知识口播」工作流制作专业数字人课程与知识博主视频：课程讲稿→讲师半身定妆照（in-image）→讲师自然手势参考视频+讲师口播配音（asset.voice）→口型同步成片（video.lipSync），并同步生成 SVG 核心知识框架动效（svg.gen→svg.anim）、课程背景轻音乐（asset.music）汇入成片时间线（output.timeline）。
workflow: course-narrate
workflow-version: 2.0.0
argument-hint: "[课程主题/讲稿内容/讲师人设形象] [画幅比例默认 9:16 或 16:9]"
when-to-use: 用户要制作知识博主口播视频、在线课程微课、数字人讲师对口型解说视频或带知识板书图解的口播成片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 知识口播 · 口型同步（LipSync）+ 知识板书动效成片 SOP（course-narrate v2.0）

## 拓扑架构与交付物

```
课程讲稿与字幕 (text.plain)
  ├─out→in-text──→ 主讲人定妆形象 (asset.image)
  │                  └─out→in-image──→ 口播底板视频 (asset.video) ──out→in-video─┐
  ├─out→in-text──→ 讲师口播配音 (asset.voice) ──────────────────────┬─out→in-voice─┴─→ 精准口型同步 (video.lipSync) ──out→in-clips─┐
  │                                                                └─out→in-voice──────────────────────────────────────────────────┤
  ├─out→in───────→ 核心知识图解 (svg.gen) ──out→in──→ 知识板书动效 (svg.anim) ───────────────────────────────────out-video→in-clips─┤
  ├─out→in-text──→ 课程背景轻音乐 (asset.music) ─────────────────────────────────────────────────────────────────────out→in-music────┤
  └─out→in-subtitles───────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 关键技术规范

1. **口型同步双源机制（`video.lipSync`）**：
   - `portrait`（讲师定妆照）保证人物专业度与演播室柔和布光；
   - `talking`（口播底板视频）基于定妆照生成自然点头、眼神交流与温和手势（设置 `generateAudio: false` 静音）；
   - `voice`（讲师配音）朗读课程讲稿；
   - `lipSync` 同时接收 `talking` 的视频流（`in-video`）与 `voice` 的音频流（`in-voice`），使讲师唇形与语音毫秒级严丝合缝。
2. **SVG 知识板书动效插片（`svgBoard` → `svgBoardAnim`）**：自动将讲稿中的核心公式/思维导图生成矢量动态板书并汇入 `output.timeline`，打破纯大头口播的单调感。
3. **配音防呆铁律**：`voice` 节点的 `generateInstruction` 留空即可自动朗读上游 `script` 的正文；若单独填写 `generateInstruction`，**只准写纯朗读台词，严禁写"请用知性女声朗读"等说明**。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "course-narrate", name: "<课程主题>·知识口播", generateAspectRatio: "9:16" })`（横屏微课传 `"16:9"`）。
2. `graph_read({ assetId, includeParams: true })` 获取节点 ID。
3. `graph_edit` 将完整口播讲稿写入 `script` 的 `params.text`，并根据讲师人设调整 `portrait`、`talking` 与 `svgBoard` 的 `generateInstruction`。
4. `task_run({ assetId })` 一键跑通口型同步与板书动效，在 `output.timeline` 合成完整课程视频。

## 端口与参数速查

详见 `references/ports.md`。
