---
name: wf-product-ad
description: 用「产品广告」工作流打造商业级品牌 TVC 短片：从产品 Brief 出发，先生成高定商业棚拍主视觉作为工业造型与材质锚点（in-image），再展开「生活痛点/氛围引入 → 微距工艺卖点特写 → 理想生活场景收尾」三镜关键帧与图生视频，配合质感广告旁白（asset.voice）、品牌调性配乐（asset.music）在成片时间线（output.timeline）自动合成。
workflow: product-ad
workflow-version: 2.0.0
argument-hint: "[产品名称/品类材质/核心卖点/目标人群] [画幅比例 16:9 或 9:16]"
when-to-use: 用户要为消费品、数码硬件、美妆香氛、食品饮品或家居产品制作高质感商业 TVC 广告短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 产品广告 · 商业级三镜 TVC 成片 SOP（product-ad v2.0）

## 拓扑架构与交付物

```
产品Brief与广告词 (play.script)
  ├─out→in-text──→ 商业棚拍产品主视觉锚点 (asset.image)
  │                  ├─out→in-image──→ 镜头1·氛围与痛点引入图 (asset.image) ──out→in-image──→ 镜头1·引入视频 (asset.video) ──out→in-clips─┐
  │                  ├─out→in-image──→ 镜头2·微距工艺卖点图 (asset.image) ───out→in-image──→ 镜头2·微距视频 (asset.video) ──out→in-clips─┤
  │                  └─out→in-image──→ 镜头3·使用场景与定格图 (asset.image) ──out→in-image──→ 镜头3·定格视频 (asset.video) ──out→in-clips─┤
  ├─out→in-text──→ 品牌质感旁白配音 (asset.voice) ────────────────────────────────────────────────────────────────────────out→in-voice────┤
  ├─out→in-text──→ 商业TVC定制配乐 (asset.music) ────────────────────────────────────────────────────────────────────────out→in-music────┤
  └─out→in-subtitles────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 成片时间线 (output.timeline)
```

## 商业广告摄影三段式法则

1. **产品造型锚点（`heroPackshot`）**：先用高定商业棚拍光效（柔光箱轮廓光、精确工业材质反射）生成产品标准主视觉，通过 `in-image` 锁死瓶身/机身比例、配色与材质，防止三镜里产品外观变形（若用户已有实拍产品白底图，可直接上传替换此节点）。
2. **镜头 1（氛围/痛点引入）**：展示目标受众的高格调生活场景或痛点瞬间，产品自然融入环境光影。
3. **镜头 2（微距工艺高潮 Macro Hero Shot）**：100mm 微距浅景深镜头，展现水珠飞溅、金属拉丝光泽、膏体丝滑质地或精密结构特写，高速慢动作（Slow-Motion）呈现极致品质感。
4. **镜头 3（品牌定格 Hero Packshot Reveal）**：产品在纯净光影中居中定格，光束缓慢掠过轮廓，配合 Slogan 字幕完成心智收口。
5. **广告旁白（`voiceover`）**：在 `generateInstruction` 中写入精炼的三句式广告朗读词（纯朗读文本，不含提示词指令）。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "product-ad", name: "<产品名>·TVC广告", generateAspectRatio: "16:9" })`（信息流竖屏投放传 `"9:16"`）。
2. `graph_read({ assetId, includeParams: true })` 读取节点 ID。
3. `graph_edit` 注入产品广告词、三镜商业摄影提示词与纯口播台词。
4. `task_run({ assetId })` 执行全图生成并合成 `output.timeline` 成片。

## 端口与参数速查

详见 `references/ports.md`。
