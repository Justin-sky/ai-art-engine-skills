---
name: wf-character-sheet
description: 用「角色设定」工作流从一段人设描述出发，先生成正面全身基准立绘作为唯一视觉锚点（通过 in-image 锁定五官骨相、发色与服装结构），再并发衍生正侧背标准三视图、六宫格微表情包、高张力战斗/招牌动作立绘以及 360° 环绕动态展示视频。
workflow: character-sheet
workflow-version: 2.0.0
argument-hint: "[角色姓名/性格/体型外貌/服装道具/美术风格]"
when-to-use: 用户需要设计游戏/动漫/漫剧角色原画设定集、标准三视图、表情包设定表或锁死角色一致性锚点时使用。
allowed-tools: workflow_list_installed, workflow_use_installed, graph_read, graph_edit, task_run, task_status
---

# 角色设定全案 · 主锚点锁死三视图与动态展示 SOP（character-sheet v2.0）

## 拓扑架构与交付物

```
人设档案 (text.plain)
  └─out→in-text──→ ① 正面全身主锚点立绘 (asset.image)
                     ├─out→in-image──→ ② 正/侧/背标准三视图设定表 (asset.image, 16:9)
                     ├─out→in-image──→ ③ 六宫格喜怒哀乐表情设定表 (asset.image, 16:9)
                     ├─out→in-image──→ ④ 招牌技能/动态姿势立绘 (asset.image, 9:16)
                     └─out→in-image──→ ⑤ 角色 360° 环绕动态展示视频 (asset.video, 9:16)
```

## 为什么必须采用「主锚点辐射拓扑」

旧式设定图如果让正面、侧面、表情各自只吃文本独立生成，模型会画出**三个完全不同的人**。本工作流 v2.0 强制以 **① 正面全身主锚点（`masterFront`）** 为唯一视觉源头，将其 `out` 连入下游所有衍生节点的 `in-image`：
- **三视图（`turnaround`）**：基于主锚点的发型、五官与服装剪裁，在同一张横版画布上展开正面、正侧面（90°）、正背面（180°）等高对齐视图；
- **表情表（`expressions`）**：锁死主锚点脸型、瞳色与发型，在同一张画布上绘制微笑、愤怒、惊讶、害羞、哭泣、得意 6 种头部微表情；
- **动态立绘（`actionPose`）**：继承主锚点全套穿搭，展示最具张力的战斗或性格招牌姿势；
- **动态展示（`turntableVid`）**：以主锚点为参考图生成呼吸待机或环绕展示短片。

## 提示词覆写规范（Agent 必读）

1. **`bio`（`text.plain`）**：结构化撰写「姓名/年龄体态 + 面部骨相与发型发色 + 上装/下装/鞋履/配饰材质 + 标志性手持道具 + 渲染画风 + 绝对禁止项（无杂乱背景、无文字水印）」。
2. **`masterFront`（主锚点）**：强调 `标准 A-Pose 正面全身站姿、头顶至脚底完整入画、纯净浅灰底、高精细节`。
3. **分步省流策略（可选）**：若用户想先挑出最满意的正面立绘再跑衍生图，可先单独对 `masterFront` 节点调用 `task_run({ assetId, nodeId: "<masterFrontId>" })`，确认后再整图运行。

## 端口与参数速查

详见 `references/ports.md`。
