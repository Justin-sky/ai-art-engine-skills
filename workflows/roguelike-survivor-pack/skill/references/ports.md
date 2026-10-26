# 节点拓扑与端口速查（roguelike-survivor-pack v1.0）

## 默认种子节点清单（共 10 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `heroRef` | `asset.image` | 幸存者主角基准立绘（`1:1`, `generateTransparentBg: true`） |
| `heroSheet` | `frame.animGen` | 主角移动循环序列图（`2×2`, `animPresetId: "run"`, `generateTransparentBg: true`） |
| `heroAnim` | `anim.2d` | 主角精灵切帧与GIF（`2×2`, `animKeyColor: ""`, `animStabilize: "ground"`） |
| `mobSheet` | `frame.animGen` | 怪物潮杂兵追击序列图（`2×2`, `animPresetId: "walk"`, `generateTransparentBg: true`） |
| `mobAnim` | `anim.2d` | 怪物潮精灵切帧与GIF（`2×2`, `animKeyColor: ""`, `animStabilize: "ground"`） |
| `lootNames` | `text.plain` | 弹幕投射物与掉落物名单（9 枚） |
| `lootSheet` | `asset.image` | 弹幕与掉落宝石·3×3整版（`1:1`） |
| `lootPack` | `image.iconPack` | 弹幕与掉落物切格透明打包（`rows: 3, cols: 3, canvasSize: 256`） |
| `juicyHitSfx` | `asset.sfx` | 爽快割草受击与宝石拾取音效 |
| `note` | `note.text` | 肉鸽割草（Survivor）全流程工程化说明 |

## 默认连线关系（共 6 条连线）

```
heroRef:out → heroSheet:in
heroSheet:out → heroAnim:in
mobSheet:out → mobAnim:in
lootNames:out → lootSheet:in
lootNames:out → lootPack:in-text
lootSheet:out → lootPack:in
```
