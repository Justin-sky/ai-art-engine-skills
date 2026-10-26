---
name: wf-motion-comic
description: 用「B站高燃动态漫剧」工作流将一段剧本或小说章节全自动制作为 B 站商业级有声动态漫 MP4 成片（script → charAnchor 角色与画风定妆锚点 → p1_1..p2_3 分镜原画 + climaxVideo 高潮格微动视频 → page1..page2 斜切破格漫画页 + bgm 热血配乐 → timeline 动态漫多轨时间线自动完成多角色 TTS 配音、无字底图推拉摇移、气泡随开口时机逐个弹出、全屏高频闪烁速度线/集中线、2.5D 前后景差速视差、刀口擦除转场与卡拉 OK 字幕合成）。当用户要做动态漫、有声漫剧、条漫改视频、小说漫改短片或 B 站漫剧时加载本技能。
workflow: motion-comic
workflow-version: 1.0.0
argument-hint: "[动态漫剧本/小说片段/题材画风（国漫玄幻/日漫热血黑白网点/韩系条漫/悬疑惊悚）] [页数与时长]"
when-to-use: 用户要做 B 站动态漫、有声漫剧、漫画分镜转动态视频、小说推文漫剧或带气泡按序弹出与 2.5D 视差的有声漫画短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, timeline_preview, asset_list, asset_import, task_run, task_status
---

# B站高燃动态漫剧 · 全自动生产与 2.5D 多轨编译 SOP（motion-comic v1.0）

## 默认种子拓扑与产出物（Seed Skeleton）

```
动态漫剧本 (text.plain: script)
  ├─out→in-text──→ 角色与画风定妆 (asset.image: charAnchor，输出连入全部 6 格分镜图的 in-image 锁定外貌与画风)
  ├─out→in-text──→ 第 1 页（起承对峙）：p1_1, p1_2, p1_3 (asset.image) ──out→in-image──→ 动态漫·第 1 页 (comic.page: page1)
  ├─out→in-text──→ 第 2 页（高潮破局）：p2_1, p2_2, p2_3 (asset.image) ──out→in-image──→ 动态漫·第 2 页 (comic.page: page2)
  │                                        └─(p2_3:out→in-image)──→ 高潮格·觉醒微动视频 (asset.video: climaxVideo)
  └─(page1:out, page2:out → in-image) + 热血配乐 (asset.music: bgm → in-music) ──→ 动态漫多轨成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "motion-comic", mode: "seed" })` 物化后，画布上的节点 `id` 与 `workflow.json` 里的 `key` **完全一致**（`script`、`charAnchor`、`p1_1`~`p1_3`、`page1`、`p2_1`~`p2_3`、`climaxVideo`、`page2`、`bgm`、`timeline`、`note`），可直接用于 `graph_edit`！

---

## B 站商业级动态漫八大核心编译机制（引擎原生内置）

当 `comic.page` 节点连入 `output.timeline` 的 `in-image` 端口执行时，引擎会自动触发 **动态漫多轨编译器（`buildComicMotionTimeline`）**，无需手写复杂时间轴片段即可获得以下 8 大专业效果：

| 序号 | B 站动态漫核心效果 | 在 `comic.page` 中的配置方式 | 时间线（`output.timeline`）自动编译产物 |
|---|---|---|---|
| **①** | **开场整页全景建立镜头 → 逐格特写运镜** | 面板设置 `cameraMotion`（`"auto"` / `"kenBurnsIn"` / `"kenBurnsOut"` / `"panLeft"` / `"panRight"` / `"panUp"` / `"panDown"`）与页面 `readingDirection`（`"ltr"` 国漫 / `"rtl"` 日漫） | 先播 `1.2s` 整页建立镜头（Establishing Shot），再按阅读顺序逐格切入独立无字分镜切片并施加平滑推拉摇移 |
| **②** | **无字纯净底图 + 气泡随开口时机逐个弹出** | 在每格 `bubbles` 数组中按阅读位置设置台词与 `speaker` 角色名 | 切入分镜时底图干净无气泡；当时间轴走到每句台词发声秒数（`bStart`）时，对应透明气泡图层在 `overlay` 轨（`lane: 2`）以 `popIn` 或 `elasticIn + shake` 动态弹出并保持至该格结束 |
| **③** | **全屏高频动态集中线 / 速度线演出层** | 分镜设置 `effectType: "focus-lines"`（集中线）或 `"speed-lines"`（速度线）及 `effectIntensity`（`0.5 ~ 0.85`） | 主轨自动触发 `zoom-burst`（急速推镜）或 `rgb-glitch`，同时在 `overlay` 轨（`lane: 1`）叠加全屏 `manga-focus-lines`（`3.6Hz pulse` 高频呼吸）或 `manga-speed-lines`（`4.5Hz shake` 高频抖动） |
| **④** | **2.5D 前后景差速视差 & 高潮格微动视频混排** | 分镜设置 `foregroundImageUrl`（前景透明人物 PNG）+ `parallaxDepth`（`0.1 ~ 1.0`）；或为高潮格设置 `videoUrl`（绑定 `climaxVideo` 产出的 MP4 相对路径） | 绑定 `foregroundImageUrl` 时在 `overlay` 轨（`lane: 1`）自动生成双关键帧差速 2.5D 景深视差；绑定 `videoUrl` 时该格直接无缝播放 I2V 动态视频 |
| **⑤** | **顺应斜切刀口的漫画硬核转场** | 分镜设置四角斜切 `cornerOffsets`（8 元组归一化偏移） | 自动匹配斜切擦除（`diagtl`）、横向抹擦（`wiperight` / `wipeleft`）、高潮圆形展开（`circleopen`）转场，并自动预留 `0.28s` 几何交叠窗口确保 Web 预览与 FFmpeg `xfade` 丝滑过渡 |
| **⑥** | **多角色音色自动分配 TTS 配音 + 以音定画** | 为每个对白气泡填写准确的 `speaker`（角色名，如 `"凌霄"`、`"玄影"`），旁白用 `shape: "narration"` | 若未手动连入外部 `asset.voice`，执行 `output.timeline` 时自动按 `speaker` 哈希分配差异化 TTS 音色（旁白用沉稳 `onyx`，不同角色自动分配 `nova` / `alloy` / `shimmer` / `echo` / `fable`），并按合成语音真实时长反推每格停留秒数 |
| **⑦** | **咆哮框与拟声词（SFX）冲击贴纸 + 程序化音效** | 情绪爆发台词设 `shape: "shout"`；刀剑/爆炸/雷电拟声词设 `shape: "sfx"`（如 `"铮——！"`、`"轰隆！！"`、`"咔嚓！"`） | 自动识别文本语义，在 `overlay` 轨（`lane: 3`）触发 `neon-slash-combo`（刀光）、`electric-lightning`（雷电）、`comic-boom`（爆炸）、`shockwave-ring`（气浪）贴纸，并在 `sfx` 轨同步触发 `whoosh` / `boom` / `pop` 音效 |
| **⑧** | **逐词弹跳卡拉 OK 字幕 + BGM 自动闪避** | 上游连入 `bgm`（`asset.music`） | 自动生成 `subtitleKaraokeMode: "karaoke-pop"` 逐词高亮字幕（呐喊框自动切红底黄字暴击高亮），并开启 `mixAutoDucking: true`（有人声时 BGM 自动压低至 25%） |

---

## 标准全自动执行 SOP（用户输入剧本 → 一键产出有声动态漫 MP4）

当用户选中本工作流并输入一段剧本或故事梗概时，请按以下 5 步全自动完成整集动态漫的编排与出片：

### 第 1 步：落盘工作流种子骨架
调用 `workflow_use_installed({ id: "motion-comic", mode: "seed" })` 落盘工作流资产，拿到 `assetId`。

### 第 2 步：按剧情高潮结构规划页数与分镜（可按需用 `graph_edit` 增删页数或挂载 2.5D 抠图节点）
- **默认 2 页 6 格 + 1 高潮微动视频架构**：
  - **第 1 页（`page1`，起承对峙 3 格）**：`p1_1` 顶部横幅定场远景（`panRight`）+ `p1_2` 左下竖格强敌现身（`kenBurnsIn` + `vignette`）+ `p1_3` 右下竖格按刀警觉（`kenBurnsIn` + `focus-lines` + 拔刀拟声词 `"铮——！"`）。
  - **第 2 页（`page2`，高潮破局 3 格斜切破框）**：`p2_1` 左上斜切疾闪突袭（`panLeft` + `speed-lines`）+ `p2_2` 右上斜切兵刃交锋（`kenBurnsIn` + `focus-lines` + `shout` 咆哮框）+ `p2_3` 底部跨幅大格觉醒破局（配合 `climaxVideo` 生成微动视频）。
- **可选高阶扩展（`graph_edit`）**：
  - 若需为某个关键角色特写格开启 **2.5D 前后景视差**，可新增 `image.matte` 节点抠取角色透明前景，或提醒用户在漫画页下钻编辑器中点「✂️ AI 抠取前景立绘 (2.5D)」。

### 第 3 步：批量写入剧本、角色定妆、各格画面指令、漫画页 JSON 与 BGM 指令（`graph_edit`）
在一次 `graph_edit` 批量操作中同时更新：
1. **`script`（`text.plain`）**：写入整理好的分幕分格动态漫剧本。
2. **`charAnchor`（`asset.image`）**：写明主角外貌、服饰、画风基准，强调「纯画面不含任何文字」。
3. **分镜图 `p1_1`~`p2_3`（`asset.image`）**：写明每格景别、动作张力、光影氛围，末尾统一注明「严格保持与参考图一致的角色形象与画风，纯画面不含任何文字与气泡」。
4. **高潮微动视频 `climaxVideo`（`asset.video`）**：写明基于 `p2_3` 的 2.5D 微动提示词（保持原漫画画风与构图，角色眼神眨动、发丝披风随气浪飘动、剑气雷光闪烁，镜头平稳推近）。
5. **漫画页 `page1`、`page2`（`comic.page`）**：将编排好的 `comicPage` JSON 字符串写入 `params.comicPage`（**注意：不要预填 `imageUrl`，保持为空以便 `comic.page` 执行时自动按阅读顺序填入上游刚生成的分镜图**；每个气泡务必填好 `speaker`、合理的格内坐标 `x, y` 与 `shape`）。
6. **配乐 `bgm`（`asset.music`）**：根据题材（热血燃系 / 悬疑诡谲 / 古风仙侠）写入纯音乐 `generateInstruction`（`generateMusicInstrumental: true`）。

### 第 4 步：一键运行整图合成动态漫（`task_run`）
- 调用 `task_run({ assetId })` 按拓扑序自动执行：先出定妆图 → 并发生成各格分镜图与 BGM → 合成漫画页无字分镜切片与独立透明气泡切片 → 在 `timeline`（`output.timeline`）中自动合成多角色 TTS 配音并导出完整动态漫 MP4！
- 若希望将 `climaxVideo` 生成的高潮微动视频绑定进 `page2` 的最后一格，可在 `climaxVideo` 生成完成后读取其 `previewRelativePath`，用 `graph_edit` 写入 `page2` 对应 panel 的 `videoUrl` 字段并重跑 `timeline`。

### 第 5 步：时间线抽帧自检或微调（可选）
- 可调用 `timeline_read({ assetId, timelineNodeId: "timeline" })` 查看自动编译出的多轨片段（`video` / `overlay` / `voice` / `subtitle` / `sfx` / `music`），或用 `timeline_preview` 抽帧检视成片效果。

## 端口与 `comicPage` 动态漫完整字段规范

详见 `references/ports.md`。
