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
动态漫剧本 (text.plain: script，独立剧本备忘，不连入各格生图节点的 in-text 以防多格文字污染单格画面)
角色与画风定妆 (asset.image: charAnchor，输出连入全部 6 格分镜图的 in-image 锁定外貌与画风)
  ├─out→in-image──→ 第 1 页（起承对峙）：p1_1, p1_2, p1_3 (asset.image) ──out→in-image──→ 动态漫·第 1 页 (comic.page: page1)
  ├─out→in-image──→ 第 2 页（高潮破局）：p2_1, p2_2, p2_3 (asset.image) ──out→in-image──→ 动态漫·第 2 页 (comic.page: page2)
  │                                         └─(p2_3:out→in-image)──→ 高潮格·觉醒微动视频 (asset.video: climaxVideo) ──out→in-video──→ page2
  └─(page1:out, page2:out → in-image) + 热血配乐 Loop (asset.sfx: bgm → in-music) ──→ 动态漫多轨成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "motion-comic", mode: "seed" })` 物化后，画布上的节点 `id` 与 `workflow.json` 里的 `key` **完全一致**（`script`、`charAnchor`、`p1_1`~`p1_3`、`page1`、`p2_1`~`p2_3`、`climaxVideo`、`page2`、`bgm`、`timeline`、`note`），可直接用于 `graph_edit`！
> **严禁将 `script` 连入分镜生图节点的 `in-text`**：含多格描述与引号台词的剧本若连入 `asset.image` 的 `in-text`，会导致生图模型把单格画成多格漫画并烘入乱码气泡。每格画面描述只写在各自的 `generateInstruction` 中！

---

## B 站商业级动态漫八大核心编译机制（引擎原生内置）

当 `comic.page` 节点连入 `output.timeline` 的 `in-image` 端口执行时，引擎会自动触发 **动态漫多轨编译器（`buildComicMotionTimeline`）**，无需手写复杂时间轴片段即可获得以下 8 大专业效果：

| 序号 | B 站动态漫核心效果 | 在 `comic.page` 中的配置方式 | 时间线（`output.timeline`）自动编译产物 |
|---|---|---|---|
| **①** | **16:9 横屏画幅统一 + 全景建立镜头 → 逐格特写运镜** | 页面设 `width: 1920, height: 1080`（16:9 横屏，与导出视频画幅一致），面板设置 `cameraMotion`（`"auto"` / `"kenBurnsIn"` / `"kenBurnsOut"` / `"panLeft"` / `"panRight"` / `"panUp"` / `"panDown"`） | 先播 `1.2s` 16:9 整页建立镜头（Establishing Shot），再按阅读顺序逐格切入独立 16:9 全画幅无字分镜切片并施加平滑推拉摇移 |
| **②** | **无字纯净底图 + 气泡随开口时机逐个弹出（含镜头跟随微移）** | 在每格 `bubbles` 数组中按阅读位置设置台词与 `speaker` 角色名 | 切入分镜时底图干净无气泡；当时间轴走到每句台词发声秒数（`bStart`）时，对应透明气泡图层在 `overlay` 轨（`lane: 2`）以 `popIn` 或 `elasticIn + shake` 动态弹出，并随底图 `cameraMotion` 产生同向微幅跟随漂移 |
| **③** | **全屏高频动态集中线 / 速度线演出层** | 分镜设置 `effectType: "focus-lines"`（集中线）或 `"speed-lines"`（速度线）及 `effectIntensity`（`0.5 ~ 0.85`） | 主轨自动触发 `zoom-burst`（急速推镜）或 `rgb-glitch`，同时在 `overlay` 轨（`lane: 1`）叠加全屏 `manga-focus-lines`（`3.6Hz pulse` 高频呼吸）或 `manga-speed-lines`（`4.5Hz shake` 高频抖动） |
| **④** | **2.5D 前后景差速视差 & 高潮格微动视频混排（含尾帧保护）** | 将分镜图连入 `image.matte` 节点并将抠图结果连入 `comic.page:in-image`（或手动设 `foregroundImageUrl` + `parallaxDepth`）；高潮格连入 `climaxVideo -> page2:in-video` | 自动识别上游 `image.matte` 前景透明图并在 `overlay` 轨（`lane: 1`）生成双关键帧差速 2.5D 景深视差；高潮格播放 I2V 动态视频且在台词超时时自动克隆尾帧平滑延展 |
| **⑤** | **顺应斜切刀口的漫画硬核转场 + 跨页翻页过渡** | 分镜设置四角斜切 `cornerOffsets`（8 元组归一化偏移） | 自动匹配斜切擦除（`diagtl`）、横向抹擦（`wiperight` / `wipeleft`）、高潮圆形展开（`circleopen`）转场，多页串联时页间自动插入 `0.32s` 横向平滑翻页（`smoothleft`）过渡 |
| **⑥** | **多角色音色自动分配并发 TTS 配音 + 项目角色声纹优先** | 为每个对白气泡填写准确的 `speaker`（角色名，如 `"凌霄"`、`"玄影"`），旁白用 `shape: "narration"` | 执行 `output.timeline` 时自动优先匹配项目已注册的角色声纹（`voiceProfile`），未注册时按 `speaker` 哈希分配差异化音色并 3 路并发合成 TTS 配音，按真实语音时长反推每格停留秒数 |
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
1. **`script`（`text.plain`）**：写入整理好的分幕分格动态漫剧本（仅作画布文档备忘，不要连入 `asset.image` 的 `in-text`）。
2. **`charAnchor`（`asset.image`）**：写明主角外貌、服饰、画风基准，强调「整张图只画独立角色设定图，禁止分格边框、禁止任何对话框气泡与文字」。
3. **分镜图 `p1_1`~`p2_3`（`asset.image`）**：写明每格景别、动作张力、光影氛围，末尾统一注明「严格保持与参考图一致的角色形象与画风；整张图只画这一幅独立分镜画面，禁止绘制漫画分格线、禁止任何台词气泡与文字」。
4. **高潮微动视频 `climaxVideo`（`asset.video`）**：写明基于 `p2_3` 的 2.5D 微动提示词（保持原漫画画风与构图，角色眼神眨动、发丝披风随气浪飘动、剑气雷光闪烁，镜头平稳推近），并确保 `climaxVideo (out) -> page2 (in-video)` 已连接（引擎会自动将 `climaxVideo` 产出的视频绑定至 `page2` 对应高潮格的 `videoUrl`）。
5. **漫画页 `page1`、`page2`（`comic.page`）**：将编排好的 `comicPage` JSON 字符串写入 `params.comicPage`（**注意：不要预填 `imageUrl`，保持为空以便 `comic.page` 执行时自动按阅读顺序填入上游分镜图**；每个气泡务必填好 `speaker`、位于中上部的合理坐标 `y: 0.18 ~ 0.62` 以避开底部字幕带，以及 `shape`）。
6. **配乐 `bgm`（`asset.sfx`）**：使用英文描述纯器乐 BGM Loop（如 `"High-energy anime battle soundtrack loop, shamisen and taiko drums with electronic bass, instrumental without vocals"`，`generateSoundLoop: true`，`generateSoundDurationSec: 16`）。

### 第 4 步：一键运行整图合成动态漫（`task_run`）
- 直接调用一次 `task_run({ assetId })` 即可按拓扑序全自动跑通整条流水线：定妆图 → 并发生成各格分镜图与 BGM → 生成高潮微动视频 `climaxVideo` → 漫画页 `page1` / `page2` 自动关联分镜图与高潮格 `videoUrl` 并切出无字底图切片与独立透明气泡切片 → `timeline`（`output.timeline`）自动按角色分配音色合成 TTS 配音、对齐主轨转场与动态贴纸并导出完整动态漫 MP4！无需拆成多次单节点 `task_run`。

### 第 5 步：时间线抽帧自检或微调（可选）
- 可调用 `timeline_read({ assetId, timelineNodeId: "timeline" })` 查看自动编译出的多轨片段（`video` / `overlay` / `voice` / `subtitle` / `sfx` / `music`），或用 `timeline_preview` 抽帧检视成片效果。

## 端口与 `comicPage` 动态漫完整字段规范

详见 `references/ports.md`。
