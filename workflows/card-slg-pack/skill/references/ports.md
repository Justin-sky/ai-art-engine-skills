# 节点拓扑与端口速查（card-slg-pack v1.0）

## 默认种子节点清单（共 7 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `cardIllust` | `asset.image` | 核心卡面英雄/技能插画（`3:4`） |
| `cardFrameUi` | `asset.image` | 可拉伸卡牌外框与羊皮纸面板（`3:4`, `generateTransparentBg: true`） |
| `badgeNames` | `text.plain` | 卡牌费用与SLG阵营徽章名单（每行一枚，最多 9 枚） |
| `badgeSheet` | `asset.image` | 费用水晶与阵营徽章·3×3整版（`1:1`） |
| `badgePack` | `image.iconPack` | 徽章切格与透明打包（`rows: 3, cols: 3, canvasSize: 256`） |
| `slgHexTile` | `asset.image` | SLG 等轴测六角主城/资源地块（`1:1`, `generateTransparentBg: true`） |
| `note` | `note.text` | 卡牌构筑（DBG）与 SLG 战棋全套资产管线说明 |

## 默认连线关系（共 3 条连线）

```
badgeNames:out → badgeSheet:in
badgeNames:out → badgePack:in-text
badgeSheet:out → badgePack:in
```
