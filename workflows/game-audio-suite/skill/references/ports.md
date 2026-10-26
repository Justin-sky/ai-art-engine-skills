# 节点拓扑与端口速查（game-audio-suite v1.0）

## 默认种子节点清单（共 7 个节点，物化后 `nodeId` 与 `key` 一致）

| key (nodeId) | typeId | 标题 |
|---|---|---|
| `bgmExplore` | `asset.music` | 探索态/主城环境 BGM |
| `bgmBattle` | `asset.music` | 高燃 Boss 战斗态 BGM |
| `sfxAttack` | `asset.sfx` | 近战挥砍/武器攻击音效 |
| `sfxHit` | `asset.sfx` | 重击命中/护甲受击音效 |
| `sfxUiClick` | `asset.sfx` | UI 按钮确认/卡牌出牌音效 |
| `battleVoice` | `asset.voice` | 角色必杀技/受击战斗语音 |
| `note` | `note.text` | 游戏音频工程化后处理管线说明 |

## 配套 MCP 音频工程化工具：`game_audio_variants`

- **BGM 无缝循环处理**：
  `game_audio_variants({ assetId: "<bgm资产ID>", mode: "bgm_loop", crossfadeSec: 2.5 })`
- **SFX 静音切除 + 防疲劳变体矩阵**：
  `game_audio_variants({ assetId: "<sfx资产ID>", mode: "sfx_variants", count: 4, pitchSpreadSemitones: 1.8, trimSilence: true })`
