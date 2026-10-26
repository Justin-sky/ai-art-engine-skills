# Steam商店胶囊图与宣传片 · 端口与参数速查

## 节点清单

| 节点 Key | 类型 (`typeId`) | 核心参数 |
| :--- | :--- | :--- |
| `headerCapsule` | `asset.image` | `generateAspectRatio: "16:9"`, 构图预留左侧 1/3 标题负空间 |
| `verticalCapsule` | `asset.image` | `generateAspectRatio: "3:4"`, 竖幅主视觉，顶部预留标题负空间 |
| `gameLogo` | `asset.image` | `generateTransparentBg: true`, `generateAspectRatio: "16:9"`, 原生透明通道徽标 |
| `trailerVid` | `asset.video` | `generateAspectRatio: "16:9"`, `generateDuration: 5`, 接收 `headerCapsule` 作为首帧 |
| `trailerBgm` | `asset.music` | `musicInstrumental: true`, `generateDuration: 30`, 商店预告片配乐 |
| `storeTimeline` | `output.timeline` | 汇总 `trailerVid` 与 `trailerBgm` 合成商店宣传片成片 |

## 连线关系

| 源节点 (`from`) | 源端口 (`fromPort`) | 目标节点 (`to`) | 目标端口 (`toPort`) | 说明 |
| :--- | :--- | :--- | :--- | :--- |
| `headerCapsule` | `out` | `trailerVid` | `in` | 横版主视觉作为商店预告片首帧参考 |
| `trailerVid` | 媒体引用 | `storeTimeline` | 媒体引用 | 视频镜头汇入时间轴 |
| `trailerBgm` | 媒体引用 | `storeTimeline` | 媒体引用 | 预告片配乐汇入时间轴 |
