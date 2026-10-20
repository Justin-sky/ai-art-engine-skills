---
name: wf-hand-drawn-styles
description: 用「21种手绘插画画风」技能与工作流按原样配方生成21种手绘插画（xkcd火柴人、蜡笔童涂、吉卜力、亲子投稿蜡笔卡3.1、小豆人信息图、MS Paint烂涂鸦、圆珠笔涂鸦、蜡笔实拍、水墨写意、复古像素、淡彩速写、复古动画概念稿、暖光童画、北欧纸雕、北欧绘本水粉、大鼻软偶、聚光水粉立绘、墨线绘本、暖色扁平绘本、圆头红线19、暖黄墨线20）。当用户提到手绘画风、绘本插画或编号1-20/3.1时加载。
workflow: hand-drawn-styles
workflow-version: 1.0.0
---

# 21种手绘插画画风（hand-drawn-styles）

> 整合自开源项目 [`threerocks/hand-drawn-styles`](https://github.com/threerocks/hand-drawn-styles)，包含 **21 种经实测定稿的手绘插画原样配方**（`1`–`20` + `3.1`）与 **附录 A · 实拍纸质感增强层**。
>
> - 执行协议与占位符规则见同目录 `references/PROTOCOL.md`
> - 全部 21 种原样提示词配方与附录 A 见同目录 `references/STYLES.md`
> - 节点端口与参数映射见同目录 `references/ports.md`

---

## 21 种手绘画风速查表

| 编号 | 中文名称 | 英文别名 | 占位符 | 特性与锚点要求 |
| --- | --- | --- | --- | --- |
| `1` | 极简黑白线条讲解漫画(xkcd 火柴人) | `xkcd`, `stickman`, `minimal-line` | `【N】【分镜列表】` | 网格多格讲解图；未给比例时软提示"多格网格排版" |
| `2` | 蜡笔童涂(5岁小孩坏画) | `crayon`, `kid-crayon` | `【主体】【主色调】【标题词】` | 纯风格；可叠附录 A 纸纹层 |
| `3` | 吉卜力风 | `ghibli` | `【主体】` | 纯风格；反堆细节、平滑水彩渐变 |
| `3.1` | 蜡笔童涂-潦草自画版 | `rawkid`, `kid-scrawl`, `family-crayon-card` | `【主体】【文字】` | **锁定锚点** `anchor-family.png` + 3阶段合同 `family-crayon-card-v3` |
| `4` | 小豆人黑色涂鸦信息图 | `bean`, `blob` | `【N】【分镜列表】` | 竖版 N 格信息图；每格唯一橙色焦点 |
| `5` | MS Paint 烂涂鸦 | `ms-paint`, `bad-doodle`, `ugly` | `【主体】` | 刻意笨拙像素锯齿；图生图效果最佳 |
| `6` | 圆珠笔单线涂鸦 | `scribble`, `pen-scribble`, `ballpoint` | `【主体】` | 单一黑圆珠笔缠绕速写线 |
| `7` | 蜡笔实拍(真·儿童手涂纸张) | `real-crayon`, `crayon-photo` | `【主体】【主色调】【标题词】` | 手机实拍微皱白纸 + 50–65% 强制露白 |
| `8` | 水墨写意 | `ink-wash`, `ink`, `shuimo` | `【主体】` | 生宣墨分五色 + 飞白墨晕 + 朱红小印章 |
| `9` | 复古像素 | `pixel`, `pixel-art`, `8-bit`, `16-bit` | `【主体】【主色调】` | 硬方块像素网格 + 抖动点阵 + 零抗锯齿 |
| `10` | 情绪叙事淡彩速写 | `emo-sketch`, `story-sketch`, `light-watercolor` | `【主体】【橙色关键物】【文字】` | 靛蓝搜索线 + 60–70%留白 + 全画唯一高饱和橙色 |
| `11` | 二维水彩风格(复古动画概念稿) | `retro-concept`, `mid-century`, `gouache-concept` | `【主体】【背景浓度】【焦点物件】【构图】【文字】` | 1950s Disney/UPA 水粉概念稿 + 桃粉形状阴影 |
| `12` | 暖光童画(动画概念暖绘) | `sunlit-storybook`, `vis-dev` | `【主体】【构图】【场景】【文字】` | 现代动画 vis-dev 哑光水粉（留白纸边档 / 满幅高调档） |
| `13` | 北欧纸雕 | `paper-folk`, `papercraft`, `quilling` | `【主体】【构图】【底色】【点缀元素】` | 立体层叠纸雕塑 + 定格木偶雕塑头 + 真实柔光投影 |
| `14` | 北欧绘本水粉 | `nordic-storybook`, `scandi-gouache` | `【主体】【构图】【纸底色】【光影】` | 全画纸纹牙口 + 丹宁蓝×芥末黄 + 脚下唯一淡蓝椭圆影 |
| `15` | 大鼻软偶 | `softnose`, `softnose-vinyl`, `art-toy` | `【主体】【构图】【眼型】【穿搭】【胡茬】【肤色】【背景底色】` | 磨砂软胶潮玩 + 超大垂管鼻 + 高位小眼无眉毛 |
| `16` | 聚光水粉立绘 | `gouache-spotlight`, `spotlight-gouache` | `【主体】【背景色】【构图】【文字】` | 单色满幅干刷底 + 身后聚光晕 + 羽化发簇 |
| `17` | 墨线绘本 | `inked-storybook`, `sketch-storybook` | `【主体】【构图】【背景色】【配色】【文字】` | 分区深色墨线 + 局部斜排线腮红 + 四边奶白毛刷纸边 |
| `18` | 暖色扁平绘本 | `warm-flat-storybook`, `flat-storybook` | `【主体】【构图】【文字】` | 零外轮廓色块相接 + 蓝橙限色板 + 七成暖白留白 |
| `19` | 圆头红线极简童画 | `roundhead-redline`, `graphite-redline` | `【主体】【文字】` | **锁定锚点** `anchor-roundhead-redline.png`；黑红白限色 + 两道竖痕眼 |
| `20` | 暖黄墨线情绪小剧场 | `warm-yellow-ink-story`, `mustard-ink-story` | `【主体】【文字】` | **锁定锚点** `anchor-warm-yellow-ink-story.png`；暖白/墨黑/芥末黄/淡珊瑚 |

---

## 核心执行协议（必读）

1. **先读 `references/PROTOCOL.md` 与 `references/STYLES.md`**：
   - 用户未指定画风时，先展示上方 21 种画风菜单请用户选择。
   - 用户已指定编号/名称/别名时，从 `references/STYLES.md` 对应章节的代码块中**一字不差地原样提取配方全文**。
   - **严禁缩写、意译或删减任何负向门禁**（如 `【最重要·硬性负向约束】`、`NO neat diagonal hatching` 等必须原样保留）。
2. **自动推断并填入 `【...】` 占位符**：
   - 根据用户主题自动补全所有 `【参数名】`（英文配方 `#12`–`#20` 必须填英文）。
   - 对于锚点锁定画风（`#3.1` / `#19` / `#20`），`【主体】` 只能写人物、动作、关系和 0–2 件必要道具，**严禁在 `【主体】` 中夹带线宽、留白比例、配色等画风控制词**。
   - `#19` 与 `#20` 的 `【文字】` 正式生产固定填 `No text anywhere.`。
3. **在 AI Art Engine 中交付或运行工作流**：
   - **仅需提示词**：直接将填好占位符的原样配方放入代码块返回给用户。
   - **在画布落地运行**：
     - 可直接调用 `workflow_use_installed({ id: "hand-drawn-styles" })` 实例化本工作流，或直接在当前画布上的 `asset.image` 节点上写入 `params.generateInstruction`。
     - 工作流预置了两条分支：
       1. **常规单阶段手绘生成**（节点 `styleCardA` / `styleCardB` / `styleCardC`）：适合 `#1`–`#18` 任意画风，将组装好的完整配方写入节点的 `generateInstruction`（注意：若 `generateInstruction` 已包含完整主体内容，可断开 `subject -> styleCard*` 的 `in-text` 连线，避免上游 `text.plain` 文本在提示词末尾重复拼接）。
       2. **主视觉锚点锁定与 `#3.1` 三阶段涂抹修正链**（节点 `anchorRef` → `lockedBase` → `scribblePass1` → `scribblePass2`）：
          - `#3.1` 官方锚点 URL：`https://gentle-starburst-99bd99.netlify.app/hand-drawn/assets/style-3.1/anchor-family.png`
          - `#19` 官方锚点 URL：`https://gentle-starburst-99bd99.netlify.app/hand-drawn/assets/style-19/anchor-roundhead-redline.png`
          - `#20` 官方锚点 URL：`https://gentle-starburst-99bd99.netlify.app/hand-drawn/assets/style-20/anchor-warm-yellow-ink-story.png`
          - 跑 `#3.1`（`family-crayon-card-v3`）时，先将锚点图写入 `anchorRef`，运行 `lockedBase` 产出初稿后，依次运行 `scribblePass1`（第一轮打散统一斜线）与 `scribblePass2`（第二轮放大粗钝笔触与 35–55% 大片漏白），以 `scribblePass2` 的输出作为最终成品。
