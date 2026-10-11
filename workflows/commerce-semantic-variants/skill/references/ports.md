# 端口与数据包速查

## 节点

| key | typeId | 入 | 出 |
|---|---|---|---|
| analyze | `semantic.analyze` | video | `semanticTimeline`（整份文档） |
| compile | `semantic.compile` | `semanticTimeline` | `videoPrompts`（`aiart.video-prompts@1`） |
| rebuild | `semantic.rebuild` | `videoPrompts` | text（计划/清单）+ video（成片） |
| note | `note.text` | — | — |

端口类型说明：

- `semanticTimeline` / `videoPrompts` 都是**结构化口**（接错类型连线时就被拒）；
  两者都可**单向**流向 `text` 口（交给文本消费者或 agent 读），反向不允许。
- `videoPrompts` 的每条提示词带 `firstFrameRelativePath`（分析证据的关键帧）——
  重建据此走**图生视频**，不会凭空文生视频。

## semanticPacks 相对路径

均相对工作流根目录，安装后落在 `<installed>/<id>/semanticPacks/`。

- `semanticPacks/commerce-vocabulary.json`
- `semanticPacks/commerce-director-rules.json`
- `semanticPacks/commerce-director-persona.json`
