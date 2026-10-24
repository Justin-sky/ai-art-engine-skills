---
name: wf-ancient-scroll-revive
description: 用「古画一镜入画」工作流将《清明上河图》《千里江山图》等中式传世古画或《雅典学院》《夜巡》《煎饼磨坊的舞会》等西方传世名画转化为「名画水平移动镜头 → 灵活中近景对位推镜 → 视频模型直出人物说话聊天原声 + 独立旁白轨道错峰解说画面」的央视/卢浮宫级短片：自动判别国画手卷（自右向左散点透视 + 东方文博朱印框）或西洋名画（焦点透视构图 + 西洋鎏金馆藏框），由多模态模型灵活框定最佳 16:9 选区并提炼场景名、人物对白与艺术史解说词（image.process + image.toPrompt），每幕还原为 8K 实拍画面，将人物说话台词直接传给视频模型（generateAudio: true）生成口型同步的现场交谈原声，配合独立旁白轨道（asset.voice）错峰解说、多轨音效（asset.sfx）与自适应器乐配乐（asset.music），在多轨成片时间线（output.timeline）一键合成。
workflow: ancient-scroll-revive
workflow-version: 1.4.0
argument-hint: "[中式古画或世界名画名称，如 清明上河图/千里江山图/雅典学院/夜巡/煎饼磨坊的舞会] [画幅比例 16:9]"
when-to-use: 用户想要把一幅中国传世古画（如《清明上河图》等长卷）或外国传世名画（如《雅典学院》《夜巡》等油画/壁画）做成「相机在画作中水平移动镜头、根据画面内容灵活取景并推进进入中近景、将人物对白直接告诉视频模型生成人物开口说话聊天的原声、同时加上独立旁白轨道错峰解说画面」的一镜入画视频时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 古画/世界名画一镜入画 · 中外双模自适应与原声对白+错峰旁白 SOP（ancient-scroll-revive v1.4）

## 拓扑架构与交付物

```
古画全卷母版 (asset.image: scroll_src)
  ├─out→in──→ 第1幕·古画取景（VLM 灵活中近景选区）(image.process: act1_crop)
  │             ├─out→in────────→ 第1幕·多模态识图与对白/解说策划 (image.toPrompt: act1_recognize)
  │             │                   ├─out→in-text───→ 第1幕·北宋实景还原（灵活中近景）(asset.image: act1_real)
  │             │                   ├─out→in-text───→ 第1幕·人物说话与活动复活（含原声对白）(asset.video: act1_video)
  │             │                   ├─out→in-text───→ 第1幕·移镜入画与柳岸环境声 (asset.sfx: act1_sfx)
  │             │                   └─out→in-text───→ 古画一镜入画成片时间线 (output.timeline)
  │             ├─out→in-image──────────────────────→ 古画一镜入画成片时间线 (output.timeline)
  │             └─out→in-image──→ 第1幕·北宋实景还原（灵活中近景）(asset.image: act1_real)
  │                                 └─out→in-first-frame──→ 第1幕·人物说话与活动复活（含原声对白）(asset.video: act1_video) ──out→in─┐
  ├─out→in──→ 第2幕·古画取景 → 第2幕·多模态识图 → 第2幕·北宋实景还原 → 第2幕·人物说话与活动复活 ──────────────────────out→in─┤
  └─out→in──→ 第3幕·古画取景 → 第3幕·多模态识图 → 第3幕·北宋实景还原 → 第3幕·人物说话与活动复活 ──────────────────────out→in─┤
第1~3幕·画面解说旁白 (asset.voice: act1_voice..act3_voice) ────────────────────────────────────────────────────────out→in-voice─┤
第1~3幕·移镜入画与环境音效 (asset.sfx: act1_sfx..act3_sfx) ────────────────────────────────────────────────────────out→in-voice─┤
北宋雅乐环境配乐 (asset.music: bgm) ───────────────────────────────────────────────────────────────────────────────out→in-music─┴─→ 成片时间线 (output.timeline)
```

## 五步核心法则（中外名画双模自适应 + 灵活中近景 + 错峰旁白与视频原声对白）

1. **中外画作构图感知与灵活中近景取景（`act1_crop` ~ `act3_crop`）**：
   - 自动探测源图真实宽高比（超宽幅长卷自适应约束 16:9 选区宽度防止挤压变形）并判别画作体系：
   - **中式古画手卷（`culture: "chinese"`）**：按自右向左散点透视顺序取景（右段起首 → 中段高潮 → 左段收束）；
   - **西方传世名画（`culture: "western"`）**：按线性焦点透视/三角构图顺序取景（第 1 幕前景/左翼铺垫群像 → 第 2 幕透视灭点/黄金分割核心主角 → 第 3 幕右翼/远景呼应群像）。
2. **多模态识图策划「人物原声对白 + 艺术史解说旁白」（`act1_recognize` ~ `act3_recognize`）**：识别切片中的人物身份、时代背景与互动状态，同时产出：
   - **供视频模型直出的人物说话对白**：契合画作时代与角色身份的口语台词，通过 `in-text` 喂给视频生成节点；
   - **供旁白轨道配音的画面解说词**：40~60 字专业考究的画面与艺术史解说词。
3. **材质消融与 8K 实景光影还原（`act1_real` ~ `act3_real`）**：国画消融宣纸水墨线条还原宋代服饰建筑；西洋名画消融亚麻布裂纹与油彩厚涂（Impasto），1:1 保留伦勃朗明暗对照光（Chiaroscuro）或印象派外光斑驳感，还原欧洲历史人物真实面部神态与丝绒/石材肌理。
4. **以写实还原照片作为首帧（`in-first-frame`）生成「全景复苏 → 镜头拉近画中人物特写 + 开口对白原声」视频（`act1_video` ~ `act3_video`）**：显式设置 `generateFrameMode: "first"` 并把 `actN_real` 连入 `in-first-frame` 端口，开启 `generateAudio: true`；提示词明确要求**以首帧 1:1 全景写实照片起幅，前 1.5 秒展示全景复苏，随后镜头平滑拉近推入画中对应焦点人物（如临窗阁楼、桥头、街肆摊位前的人物）的近景特写**，一次性生成**近景特写下口型同步的人物说话交谈声、微表情与现场环境原声**（严禁跳过多幕切片而直接用单节点生成长镜头视频）。
5. **央视/卢浮宫双美学包装 +「原画移镜定格 → 快门闪光显影写实照片 → 首帧起动拉近人物特写 → 回落全景写实图并退回古画移向下一焦点」多段剪辑合成（`output.timeline`）**：
   - **双模视觉包装（`overlay` 轨）**：国画自动挂载 `scroll-seal-frame`（宋锦双线回纹 + 朱砂「御览」篆印）；西洋名画自动切换为 `museum-gilded-frame`（巴洛克鎏金花角 + 卢浮宫式紫红铜牌 `GALLERY · ARCHIVE`）；
   - **五段式对位复活剪辑（`video` 轨）**：每幕先在古画上做 `3.6s` `scrollPanZoomIn`（`0~68%` 从 `1.22x` 水平移镜收束至 `1.00x` 焦点，68~100% 在焦点位置 `1.00x` 锁定定格）→ 触发 `0.45s` `flash`（`fadewhite`）拍照快门闪光，原地显影出 `1.6s` 的 **1:1 写实还原照片（`actN_real`，`1.00x` 定格）** → 通过 `0.35s` `dissolve` 无缝过渡到以该写实照片为首帧生成的视频（`actN_video`，全景起幅后拉近至画中人物近景特写说话）→ 幕尾先回落至 **1:1 全景写实照片（`clip:auto:real_out:i`）** 再通过 `0.75s` `dissolve` **1:1 融回本幕古画原图（`clip:auto:outro:i`）** → 最后通过 `0.85s` `smoothleft` / `smoothright` 横移手卷滑向下一幕焦点；
   - **错峰旁白与智能闪避（`voice` + `video` + `sfx` + `music`）**：`0.15s – 4.35s` 旁白率先切入解说画面（此时视频原声与环境音自动柔和闪避至 `42%`，BGM 压至 `25%`）；`4.4s – 10.2s` 旁白停歇，`100%` 释放视频模型直出的画中人物开口交谈原声；
   - **时序接力字幕（`subtitle` 轨 `lane 0`）**：`0.15s – 2.05s` 场景铭牌 → `2.10s – 5.25s` 艺术史解说字幕 → `5.35s – 幕尾` 画内人物开口对白字幕。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "ancient-scroll-revive", name: "<画作名>·一镜入画", generateAspectRatio: "16:9" })`。
2. `graph_read({ assetId, includeParams: true })` 获取各节点 ID。
3. 若用户已提供古画图片，将其绑定至 `scroll_src`；若根据用户指定的其它名画创作，用 `graph_edit` 覆写 `scroll_src`、视频对白指令与旁白解说词。
4. `task_run({ assetId })` 一键跑通全图，自动合成「水平移镜 → 灵活中近景 → 视频直出人物说话 + 旁白轨画面解说」完整成片。

## 端口与参数速查

详见 `references/ports.md`。
