# 节点拓扑与端口速查（avg-portrait-pack v1.0）

## 默认种子节点清单（共 8 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `charPortrait` | `asset.image` | 角色基准半身立绘（`3:4`, `generateTransparentBg: true`） |
| `exprSheet` | `frame.animGen` | 6格表情差分表（`2×3`, `4:3`, `animPresetId: "expr6"`, `generateTransparentBg: true`） |
| `exprPack` | `anim.2d` | 表情差分切帧与对齐输出（`2×3`, `animKeyColor: ""`, `animStabilize: "ground"`） |
| `lipSheet` | `frame.animGen` | 4格说话口型与眨眼差分（`2×2`, `1:1`, `animPresetId: "lipSync4"`, `generateTransparentBg: true`） |
| `lipAnim` | `anim.2d` | 口型眨眼动态精灵（`2×2`, `animKeyColor: ""`, `animPlayMode: "pingpong"`, `animGifFps: 6`） |
| `storyBg` | `asset.image` | 配套剧情舞台背景CG（`16:9`） |
| `charVoice` | `asset.voice` | 角色代表台词配音 |
| `note` | `note.text` | AVG / RPG 立绘与差分资产包工业管线说明 |

## 默认连线关系（共 4 条连线）

```
charPortrait:out → exprSheet:in
exprSheet:out → exprPack:in
charPortrait:out → lipSheet:in
lipSheet:out → lipAnim:in
```
