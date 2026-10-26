# 节点拓扑与端口速查（game-tileset-parallax v1.0）

## 默认种子节点清单（共 6 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `autoTileSheet` | `frame.animGen` | 九宫格地形瓦片图集（`3×3`, `autotile3x3`, `animStabilize: "off"`） |
| `tileSplitter` | `image.gridSplit` | 3×3 瓦片切格器（`gridRows: 3, gridCols: 3`） |
| `seamlessGround` | `asset.image` | 无缝平铺地表基础纹理（`1:1`） |
| `skyFarBg` | `asset.image` | 视差远景层（天空与远山，`16:9`） |
| `midForestBg` | `asset.image` | 视差中景层（透明前景剪影，`16:9`, `generateTransparentBg: true`） |
| `note` | `note.text` | 2D 瓦片地图与视差背景工程化说明 |

## 默认连线关系（共 1 条连线）

```
autoTileSheet:out → tileSplitter:in
```

## 配套 MCP 后处理算子：`game_tile_pbr_process`

- **无缝地表 + PBR 法线贴图派生**：
  `game_tile_pbr_process({ assetId, makeSeamless: true, wrapX: true, wrapY: true, derivePbr: true, pbrMaps: ["normal", "roughness", "ao", "orm"], autoTileLayout: "grid3x3" })`
- **视差滚轴背景仅水平无缝闭合**：
  `game_tile_pbr_process({ assetId, makeSeamless: true, wrapX: true, wrapY: false })`
