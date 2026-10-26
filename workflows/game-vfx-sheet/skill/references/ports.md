# 节点拓扑与端口速查（game-vfx-sheet v1.0）

## 默认种子节点清单（共 4 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `vfxSheet` | `frame.animGen` | 生成VFX特效序列图（默认 `2×4`, `16:9`, `generateTransparentBg: true`, `animStyleLock: "vfx"`） |
| `vfxAnim` | `anim.2d` | 特效切格与GIF合成（默认 `animKeyColor: ""`, `animStabilize: "center"`, `animGifFps: 14`） |
| `vfxSfx` | `asset.sfx` | 配套技能打击音效 |
| `note` | `note.text` | 2D 游戏特效（VFX Flipbook）工业管线说明 |

## 默认连线关系（共 1 条连线）

```
vfxSheet:out → vfxAnim:in
```

## 核心参数与配套 MCP 工具

- `vfxSheet`（`frame.animGen`）：
  - `animPresetId`: `"vfxSlash"`（刀光剑气）| `"vfxExplosion"`（爆炸冲击波）| `"vfxAura"`（能量光环循环）
  - `animRows` / `animCols`: `2×2`（`1:1` 画幅，4 帧轻量受击火花）| `2×4`（`16:9` 画幅，8 帧细腻刀光/爆炸）| `3×3`（`1:1` 画幅，9 帧大招特效）
  - `generateTransparentBg`: `true`
  - `animKeyColor`: `""`
- `game_audio_variants`（MCP 工具）：
  - 对 `vfxSfx` 产出的音频资产执行 `mode: "sfx_variants"`，自动裁切静音并生成 4 组音高/明暗度微调变体。
