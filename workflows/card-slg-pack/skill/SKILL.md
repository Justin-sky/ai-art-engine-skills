---
name: wf-card-slg-pack
description: 用「卡牌与SLG策略游戏资产包」工作流一键生产卡牌构筑（DBG / CCG）与 SLG 策略战棋游戏所需的 3:4 卡面主视觉插画、可 9-Slice 九宫格拉伸的卡牌边框底板、费用水晶/攻防属性/阵营纹章透明图标包与斜 45° 等轴测六角战略建筑地块。
workflow: card-slg-pack
workflow-version: 1.0.0
argument-hint: "[游戏题材（奇幻DBG卡牌/三国SLG沙盘/科幻机甲战棋/克苏鲁暗黑卡牌）/卡面主题/阵营徽章列表/六角地块类型]"
when-to-use: 用户需要制作卡牌游戏（如杀戮尖塔/炉石/三国杀类）的卡面插画、卡牌外框UI、费用属性徽章，或 SLG 策略战棋游戏的六边形地块、主城建筑、资源点图标时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_node_types, graph_read, graph_edit, asset_list, asset_import, task_run, task_status, game_tile_pbr_process, game_bundle_export
---

# 卡牌构筑与 SLG 策略战棋资产套件 · 工业管线 SOP（card-slg-pack v1.0）

## 一、默认种子拓扑与交付物（Seed Skeleton，7 节点 / 3 连线）

```
核心卡面英雄/技能插画 (asset.image: cardIllust, 3:4 竖版卡面主视觉)
可拉伸卡牌外框与羊皮纸面板 (asset.image: cardFrameUi, 3:4 原生透明外沿 → 可调 game_tile_pbr_process 检测 9-Slice 九宫格边距)
卡牌费用与SLG阵营徽章名单 (text.plain: badgeNames, 9 枚核心图标名单)
  ├─→ 费用水晶与阵营徽章·3×3整版 (asset.image: badgeSheet, 1:1 九宫格整版表)
  └─→ 徽章切格与透明打包 (image.iconPack: badgePack ← badgeSheet, 自动切格+键控透明落盘 Assets/IconPacks/card_badges)
SLG 等轴测六角主城/资源地块 (asset.image: slgHexTile, 1:1 原生透明背景斜 45° 六角战略建筑地块)
```

---

## 二、卡牌与 SLG 策略游戏三大核心工程规范

1. **规范 1 · 卡牌分层组件化（拒绝把错乱 AI 文字焊死在卡面上）**：
   - 工业级卡牌游戏必须将「卡面插画（`cardIllust`）」「卡牌边框底板（`cardFrameUi`）」与「费用/攻击/护甲/阵营角标（`badgePack`）」拆分为独立图层；
   - `cardFrameUi` 的提示词必须明确 **`严禁包含任何文字或数字，中心区域保持均匀底板纹理`**；
   - 生成后调用 `game_tile_pbr_process({ assetId: "<cardFrameUi资产ID>", detectNineSlice: true })`，自动测量四角金属雕花保留边距并输出 `_9slice.json`（含 Godot `StyleBoxTexture` 与 Unity `Sprite.border` 参数），使同一面板可无损适配任意尺寸的技能描述弹窗与卡牌框！

2. **规范 2 · 费用水晶、属性角标与阵营纹章批量打包（`badgeNames` → `badgeSheet` → `badgePack`）**：
   - 在 `badgeNames` 中按行填写最多 9 枚徽章名称（如法力水晶、攻击剑标、护甲盾标、传说星标、五大阵营徽记）；
   - `badgePack`（`image.iconPack`）自动切出 9 枚独立图标、键控去底、居中对齐到 `256×256` 标准画布并按名单命名落盘。

3. **规范 3 · SLG 斜 45° 等轴测六角地块（`slgHexTile`）**：
   - `slgHexTile` 开启 `generateTransparentBg: true`，提示词锁定 `斜 45° 等轴测（Isometric 2.5D）正六边形底座，外部为原生 RGBA 透明通道`；
   - 若需批量生成多级主城、伐木场、铁矿、兵营、迷雾山脉，可用 `graph_edit` 并联扩展多个 `asset.image` 节点并将首个 `slgHexTile:out` 连入 `in-image` 锁死六边形底座透视角度！

---

## 三、标准执行步骤（MCP 路径）

1. 调用 `workflow_use_installed({ id: "card-slg-pack", mode: "seed" })` 在当前打开的画布上实例化种子拓扑。
2. 根据用户的卡牌/SLG题材调用 `graph_edit` 修改 `cardIllust`、`cardFrameUi`、`badgeNames`、`slgHexTile` 的参数。
3. 调用 `task_run({ assetId })` 并发生成全套卡面、卡框、透明徽章包与 SLG 六角地块。
4. 调用 `game_tile_pbr_process({ assetId, detectNineSlice: true })` 为 UI 面板生成引擎级 9-Slice 九宫格切线配置。

## 四、端口与参数速查

详见 `references/ports.md`。
