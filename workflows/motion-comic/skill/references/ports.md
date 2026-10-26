# 端口与参数速查（motion-comic）

字段名与端口 id 均取自应用源码，可直接写进 `graph_edit` 的 `params`。

## text.plain（动态漫剧本）

| 端口 | 类型 | 说明 |
|---|---|---|
| `out` | `text`（多） | 文本出口，扇出到「角色与画风定妆」及全集各页分镜图节点 |

| 参数 | 说明 |
|---|---|
| `text` | 分幕分格动态漫剧本正文 |

## asset.image（角色与画风定妆 / 各格分镜原画）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-text` | `text`（多） | 提示词来源（`text.plain:out` 接这里） |
| `in-image` | `image`（多） | 参考图（「角色与画风定妆」的 `out` 连入所有分镜图的 `in-image`，锁定跨页角色形象与统一画风） |
| `out` | `image` | 当前选中一张 |

| 参数 | 说明 |
|---|---|
| `generateInstruction` | 画面生成指令（写清景别、动作、表情、环境光影，并注明「严格保持与参考图一致的角色形象与画风，纯画面不含任何文字与气泡」） |
| `generateAspectRatio` | 如 `16:9` / `3:4` / `2:3` / `1:1` |

## asset.video（高潮格 2.5D 微动视频 I2V）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-image` | `image` | 首帧参考图（连接高潮格分镜原画 `p2_3:out`） |
| `out` | `video` | 生成的动态视频，其 `relativePath` 可写入对应分镜格的 `videoUrl` 实现动静混排 |

| 参数 | 说明 |
|---|---|
| `generateInstruction` | 动态漫微动指令（如「保持原漫画分镜画风与构图，角色眼神眨动、发丝披风随风飘动、光影气场流动，镜头平稳推近」） |
| `generateDuration` | 视频时长（秒，建议 `4` 或 `5`） |

## comic.page（漫画页 / 动态漫分镜源）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-image` | `image`（**多**） | 本页各分镜图按阅读顺序逐个连一条边；执行时按阅读顺序填入尚未绑图的分格 |
| `out` / `out-all` | `image` / `images` | 执行时同时烘焙：**① 整页漫画 PNG（`comic-page:<nodeId>`）**、**② 每格无气泡纯净底图切片（`comic-panel:<nodeId>:<panelId>`）**、**③ 每个台词气泡/拟声词的独立透明 PNG 切片（`comic-bubble:<nodeId>:<bubbleId>`）**，直接供下游 `output.timeline` 编译动态漫 |

### `comicPage` 动态漫完整数据结构（`serializeComicPage` 的产物）

```jsonc
{
  "title": "第一话 · 暗夜锋芒",       // 可选，页首标题（在动态漫开场全景镜头中自动生成标题大字幕）
  "backgroundColor": "#0f172a",     // 页面底色（动态漫推荐深色 #0f172a 或纯白 #ffffff）
  "columns": 6,                      // 网格列数（推荐 6）
  "rows": 6,                         // 网格行数（推荐 6）
  "gutter": 16,                      // 分格间距（px）
  "padding": 32,                     // 页面四周外边距（px）
  "borderWidth": 3,                  // 全局分格描边宽度（px）
  "borderColor": "#111111",          // 全局分格描边颜色
  "borderRadius": 0,                 // 全局分格圆角（px）
  "readingDirection": "ltr",         // 阅读顺序：ltr（国漫/条漫左到右，默认）| rtl（日漫右到左）
  "width": 1080,                     // 页面像素宽（默认 1080）
  "height": 1520,                    // 页面像素高（页漫推荐 1520，条漫可设 1920~2400）
  "panels": [
    {
      "id": "p2-panel-1",
      "row": 0, "col": 0,
      "rowSpan": 2, "colSpan": 3,
      "title": "格 1 · 疾闪突袭",
      "caption": "可选：分镜底部旁白条文字",
      // 斜切破格：四角归一化偏移 [tlX, tlY, trX, trY, brX, brY, blX, blY]，每项 -0.45 ~ 0.45
      // 动态漫编译器会自动根据斜切方向匹配 diagtl / wiperight 等斜切擦除转场！
      "cornerOffsets": [0, 0, 0, 0, -0.08, 0, 0, 0],
      // 镜头氛围特效：none | speed-lines（速度线）| focus-lines（集中线）| vignette（暗角）| halftone-frame（网点边框）
      // 开启 focus-lines 或 speed-lines 时，动态漫会自动在 overlay 轨叠加全屏高频脉冲/震动漫画气氛线！
      "effectType": "speed-lines",
      "effectIntensity": 0.72,       // 特效强度 0.1 ~ 1
      "effectAngle": 0,              // 速度线角度 0 ~ 360
      "effectColor": "#111111",      // 特效线条颜色（#111111 黑墨锋线 | #ffffff 白光闪线）
      "imageFilter": "none",         // none | manga-bw（黑白日漫）| halftone（网点）| sepia（复古泛黄）| dramatic（戏剧暗调）
      // ── 动态漫专属分镜演出字段 ──
      // 单格运镜预设：auto（默认智能推算）| kenBurnsIn（推近特写）| kenBurnsOut（拉远全景）| panLeft | panRight | panUp | panDown | none
      "cameraMotion": "panLeft",
      // 单格停留时长覆写（秒，0.5~30；缺省时自动按格内气泡台词字数或真实 TTS 配音时长反推）
      "durationSec": 3.2,
      // 动静混排视频源（可选）：填入工程内视频相对路径（如 climaxVideo 产出的 MP4），该格在动态漫中将直接播放动态视频
      "videoUrl": "",
      // 2.5D 视差前景透明立绘图（可选）：填入透明人物 PNG 相对路径，动态漫自动在 overlay 轨生成与背景差速移动的 2.5D 景深视差关键帧
      "foregroundImageUrl": "",
      // 2.5D 视差深度强度（0.1 ~ 1.0，默认 0.5）
      "parallaxDepth": 0.55,
      // 注意：自动生产时不要预填 imageUrl，留空以便执行 comic.page 时自动填入上游生成的分镜图
      "bubbles": [
        {
          "id": "p2-b1",
          "text": "接我这一剑试试！！",
          "speaker": "凌霄",          // 说话人角色名（动态漫自动按角色名哈希分配不同 TTS 音色！）
          // 气泡形状：rounded（圆角）| oval（椭圆）| shout（爆炸咆哮，自动触发震屏与爆点贴纸）| thought（云朵心声）| narration（方形旁白）| sfx（拟声词大字，自动触发刀光/雷电/爆炸贴纸与音效）
          "shape": "shout",
          "writingMode": "horizontal-tb", // horizontal-tb（横排）| vertical-rl（日漫/古风竖排）
          "x": 0.5, "y": 0.24,       // 格内归一化锚点 0~1（动态漫中气泡将在此相对坐标处按开口时间动态弹出）
          "tail": "bl",              // tl / tr / bl / br
          "scale": 1.1,              // 缩放 0.5 ~ 4
          "rotation": 0              // 旋转角 -60 ~ 60
        }
      ]
    }
  ]
}
```

## output.timeline（动态漫多轨成片时间线）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-image` / `in-images` | `image` / `images` | 接收上游 `comic.page:out`（支持接入 1~N 张漫画页，自动按页序串联编译为完整多轨动态漫） |
| `in-voice` / `in-voices` | `voice` / `voices` | 可选：接收上游自定义 `asset.voice` 配音；**若未连接，引擎会自动提取漫画页全部对白气泡并按 `speaker` 角色名分配不同音色合成 TTS 配音** |
| `in-music` | `voice` | 接收上游 `asset.music:out` 背景音乐，自动铺满全片并开启人声闪避（`mixAutoDucking`） |
| `out` | `video` | 导出的完整有声动态漫 MP4 成片 |
