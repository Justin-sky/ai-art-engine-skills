---
name: wf-product-ad
description: 用「产品广告」工作流打造商业级品牌 TVC 短片：从品牌卖点与脚本（copy）出发，先生成高定商业棚拍主视觉（hero）作为工业造型与材质锚点（in-image），再展开真实生活使用场景（scene → videoScene）、产品流光亮相视频（videoHero）与微距工艺卖点特写（detail），配合质感品牌旁白（voice）、商业氛围配乐（bgm）在广告成片时间线（timeline: output.timeline）自动合成。
workflow: product-ad
workflow-version: 2.1.0
argument-hint: "[产品名称/品类材质/核心卖点/目标人群] [画幅比例 16:9 或 9:16]"
when-to-use: 用户要为消费品、数码硬件、美妆香氛、食品饮品或家居产品制作高质感商业 TVC 广告短片时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, timeline_read, timeline_edit, timeline_export, asset_list, asset_import, task_run, task_status
---

# 产品广告 · 商业级品牌 TVC 成片 SOP（product-ad v2.1）

## 默认种子拓扑与交付物（Seed Skeleton，10 节点 / 17 连线）

```
品牌卖点与脚本 (text.plain: copy)
  ├─out→in-text──→ 产品棚拍主视觉·锚点 (asset.image: hero)
  │                  ├─out→in-image────────────────────────────────────────────────────→ 产品流光亮相视频 (asset.video: videoHero) ──out→in───────┐
  │                  ├─out→in-image──→ 真实生活使用场景 (asset.image: scene) ──out→in-image──→ 场景使用体验视频 (asset.video: videoScene) ──out→in───────┤
  │                  └─out→in-image──→ 微距工艺卖点特写 (asset.image: detail) ─────────────────────────────────────────────────────out→in-image─┤
  ├─out→in-text──→ 品牌质感旁白配音 (asset.voice: voice) ──────────────────────────────────────────────────────────────────────────out→in-voice─┤
  ├─out→in-text──→ 商业品牌氛围配乐 (asset.music: bgm) ────────────────────────────────────────────────────────────────────────────out→in-music─┤
  └─out→in-text─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────→ 广告成片时间线 (output.timeline: timeline)
```

> **节点 ID 直通原则**：调用 `workflow_use_installed({ id: "product-ad", mode: "seed" })` 物化后，画布节点 `id` 即为 `copy`、`hero`、`scene`、`detail`、`videoHero`、`videoScene`、`voice`、`bgm`、`timeline`、`note`。

## 商业广告摄影三段式法则

1. **产品造型锚点（`hero` · `asset.image`）**：
   - 先用高定商业棚拍光效（柔光箱轮廓光、精确工业材质反射）生成产品标准主视觉，通过 `in-image` 锁死瓶身/机身比例、配色与材质，防止跨镜头产品外观变形（若用户已有实拍产品图，可用 `asset_import` 导入替换 `hero`）。
2. **镜头 1 · 流光亮相（`videoHero` · `asset.video`）**：
   - 基于 `hero` 生成光束缓慢掠过产品轮廓的戏剧化开场镜头，瞬间建立高端品牌心智。
3. **镜头 2 · 生活方式共鸣（`scene` → `videoScene`）**：
   - 展示目标受众的高格调真实使用场景，产品自然融入环境光影与人物互动。
4. **镜头 3 · 微距工艺特写（`detail` · `asset.image`）**：
   - 100mm 微距浅景深镜头，展现水珠飞溅、金属拉丝光泽、膏体丝滑质地或精密结构特写，连入 `timeline:in-image`（或接 `videoDetail` 升级为高速慢动作视频）。
5. **品牌旁白防呆铁律（`voice` · `asset.voice`）**：
   - 在 `voice 的 params.` 中**只写精炼的纯广告朗读词正文**（如「在快节奏的都市清晨，让肌肤重获深层呼吸……」），严禁混入提示词指令或语气说明。

## 动态算子伸缩与跨技能自由混搭（Seed & Dynamic Composition）

- **升级微距动态镜头（`detail` → `videoDetail`）**：若用户希望全片皆为动态视频，可用 `graph_edit` 新增 `videoDetail`（`typeId: "asset.video"`，慢动作微距运镜），连接 `detail:out → videoDetail:in-image` 与 `videoDetail:out → timeline:in`，并断开 `detail → timeline`。
- **混搭 SVG 科技参数动效（跨技能融合）**：数码/美妆成分党广告可新增 `svgSpec`（`svg.gen`）→ `svgSpecAnim`（`svg.anim`），生成芯片架构或成分渗透动态矢量图解，从 `out-gif` 连入 `timeline:in-image`。
- **后期时间线精剪（`timeline_edit`）**：通过 `timeline_edit` 配置高级电影感转场（`videoTransition: "dissolve" | "fadewhite"`）、环境音效（如倒水声、机械旋钮声 `asset.sfx`）与品牌 Slogan 收尾字幕。

## 标准执行步骤（MCP 路径）

1. `workflow_use_installed({ id: "product-ad", mode: "seed" })` 落盘种子拓扑。
2. `graph_edit` 写入 `copy` 的三段式广告脚本、更新 `hero` / `scene` / `detail` / `videoHero` / `videoScene` 的商业摄影提示词（统一横屏 `"16:9"` 或信息流竖屏 `"9:16"`），并将纯朗读台词写入 `voice 的 params.`。
3. `task_run({ assetId })` 执行全图生成并合成 `timeline` 成片。

## 端口与参数速查

详见 `references/ports.md`。
