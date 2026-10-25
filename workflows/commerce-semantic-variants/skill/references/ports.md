# 端口与数据包速查

## 节点

| key | 名称 | typeId | 入 | 出 |
|---|---|---|---|---|
| analyze | 视频拆解 | `semantic.analyze` | video + 参考图 | `semanticTimeline`（拆解时间线文档） |
| compile | 重制规划 | `semantic.compile` | `semanticTimeline` | `videoPrompts`（重制提示词方案 `aiart.video-prompts@1`） |
| rebuild | 视频重制 | `semantic.rebuild` | `videoPrompts` | `out-video`（重制成片） |
| note | 使用说明 | `note.text` | — | — |

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
