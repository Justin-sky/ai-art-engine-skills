# 暴击美术跳字与Shader特效套件 · 端口与参数速查

## 节点清单

| 节点 Key | 类型 (`typeId`) | 核心参数 |
| :--- | :--- | :--- |
| `glyphNames` | `text.plain` | 16 行字符名称清单（`0-9`、`+`、`-`、`!`、`%`、`C`、`M`） |
| `fontSheet` | `asset.image` | `generateAspectRatio: "1:1"`, 4×4 均匀网格艺术跳字整版图 |
| `fontPack` | `image.iconPack` | `rows: 4`, `cols: 4`, `canvasSize: 256`, 自动切格去底输出 16 枚透明字符 PNG |
| `dissolveNoise` | `asset.image` | `generateAspectRatio: "1:1"`, 灰度无缝溶解噪声贴图 |
| `critHitSfx` | `asset.sfx` | 重击暴击音效 |

## 连线关系

| 源节点 (`from`) | 目标节点 (`to`) | 说明 |
| :--- | :--- | :--- |
| `glyphNames` | `fontSheet` | 16 字符清单注入整版生成提示词 |
| `glyphNames` | `fontPack` | 16 字符清单驱动切图文件命名 |
| `fontSheet` | `fontPack` | 4×4 跳字整版图送入切格去底打包器 |
