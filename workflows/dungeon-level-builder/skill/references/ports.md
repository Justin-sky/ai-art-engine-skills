# 2D地牢与平台关卡设计套件 · 端口与参数速查

## 节点清单

| 节点 Key | 类型 (`typeId`) | 核心参数 |
| :--- | :--- | :--- |
| `tilesetImg` | `asset.image` | `generateAspectRatio: "1:1"`, 关卡地面与墙体瓦片整版图 |
| `propNames` | `text.plain` | 9 行关卡可交互物件清单（宝箱、传送门、陷阱、祭坛、拉杆等） |
| `propSheet` | `asset.image` | `generateAspectRatio: "1:1"`, 3×3 九宫格交互物件整版图 |
| `propPack` | `image.iconPack` | `rows: 3`, `cols: 3`, `canvasSize: 256`, 自动切格并去底为透明 PNG |
| `dungeonBgm` | `asset.music` | `musicInstrumental: true`, `generateDuration: 30`, 关卡探索循环配乐 |

## 连线关系

| 源节点 (`from`) | 目标节点 (`to`) | 说明 |
| :--- | :--- | :--- |
| `propNames` | `propSheet` | 物件名单注入整版生成提示词 |
| `propNames` | `propPack` | 物件名单驱动切图文件命名 |
| `propSheet` | `propPack` | 3×3 整版图送入切格去底打包器 |
