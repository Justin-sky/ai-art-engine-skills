# `hand-drawn-styles` 节点与端口参考

## 工作流节点拓扑

| 节点 `key` | `typeId` | 默认标题 | 作用说明 |
| --- | --- | --- | --- |
| `subject` | `text.plain` | 画面主题与故事设定 | 存放通用故事主题（输出口 `out`） |
| `styleCardA` | `asset.image` | 手绘画风 A · 暖光童画（#12） | 常规单阶段手绘生成槽位 A（默认预置 `#12` 原样配方） |
| `styleCardB` | `asset.image` | 手绘画风 B · 情绪叙事淡彩速写（#10） | 常规单阶段手绘生成槽位 B（默认预置 `#10` 原样配方） |
| `styleCardC` | `asset.image` | 手绘画风 C · 北欧纸雕（#13） | 常规单阶段手绘生成槽位 C（默认预置 `#13` 原样配方） |
| `anchorRef` | `asset.image` | 画风主视觉锚点（#3.1 / #19 / #20 专用） | 存放 `#3.1` / `#19` / `#20` 的官方 `style-only` 锚点参考图 |
| `lockedBase` | `asset.image` | 锚点锁定画风 · 基础生成（#3.1 / #19 / #20） | 接收 `anchorRef`（`in-image-1`），执行锚点锁定画风的第 1 阶段生成 |
| `scribblePass1` | `asset.image` | #3.1 第一轮蜡笔涂抹打散（scribble-correction） | 接收 `lockedBase`（`in-image-1`）+ `anchorRef`（`in-image-2`），打散规整排线 |
| `scribblePass2` | `asset.image` | #3.1 第二轮粗钝漏白终稿（scribble-chaos-correction） | 接收 `scribblePass1`（`in-image-1`）+ `anchorRef`（`in-image-2`），产出 `#3.1` 终稿 |
| `note` | `note.text` | 21种手绘画风速查与用法 | 画布说明便签，不参与执行 |

## `asset.image` 关键参数与连线注意

- **`generateInstruction`**：写入从 `references/STYLES.md` 原样提取并填好 `【...】` 占位符后的完整提示词。
- **`generateAspectRatio`**：
  - 用户明确指定画幅比例（如 `"3:4"`、`"16:9"`、`"1:1"`）时，同时设置 `params.generateAspectRatio` 并在 `generateInstruction` 末尾追加 `画幅比例:3:4。`。
- **关于上游 `subject`（`text.plain`）连线**：
  - 当你已经把具体主体填进 `generateInstruction` 的 `【主体】` 占位符时，可保持 `subject.params.text` 为空或移除 `subject -> styleCard*` 的 `in-text` 连线，避免 `asset.image` 执行器把 `incomingText` 再次追加到提示词末尾破坏原样配方的收尾结构。
- **关于 `#3.1` / `#19` / `#20` 主视觉锚点连线**：
  - `lockedBase` 的 `in-image-1` 接 `anchorRef.out`（对应配方里的 `Image 1 is the only approved style-only anchor`）。
  - `#3.1` 的两轮编辑节点 `scribblePass1`、`scribblePass2` 均需要 **2 张输入参考图**：
    - `in-image-1`：上一阶段生成的待修图（`Image 1 is the edit target`）
    - `in-image-2`：`anchorRef` 主视觉锚点图（`Image 2 is the approved style-only reference`）
