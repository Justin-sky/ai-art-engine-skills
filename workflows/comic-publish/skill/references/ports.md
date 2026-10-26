# 端口与参数速查（comic-publish）

字段名与端口 id 均取自应用源码，可直接写进 `graph_edit` 的 `params`。
通过 `workflow_use_installed({ id: "comic-publish", mode: "seed" })` 落盘后，节点 `id` 与 `workflow.json` 中的 `key` 完全一致（`script`、`charAnchor`、`p1_1`~`p3_3`、`page1`~`page3`、`note`），可直接用于 `graph_edit`（亦支持点路径局部更新如 `"comicPage.panels[0].bubbles[0].text"`）。

## text.plain（漫画剧本 / 文本备忘节点）

| 端口 | 类型 | 说明 |
|---|---|---|
| `out` | `text`（多） | 文本出口（注意：多页整集剧本请勿连入单格 `asset.image` 的 `in-text`，避免把全集台词与多格描述混入单格画面） |

| 参数 | 说明 |
|---|---|
| `text` | 整集分场分格剧本正文 |

## asset.image（角色与画风定妆 / 各页分镜图）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-text` | `text`（多） | 提示词来源 |
| `in-image` | `image`（多） | 参考图（「角色与画风定妆」的 `out` 连入所有分镜图的 `in-image`，锁定跨页角色形象与统一画风） |
| `out` | `image` | 当前选中一张（新建生成后默认 = 最新一张） |
| `out-all` | `images`（多） | 全部历史，仅供 `image.select` 一族消费 |

| 参数 | 说明 |
|---|---|
| `generateInstruction` | 画面生成指令（分镜图请写清景别、动作、表情、环境光影，并注明「严格保持与参考图一致的角色形象与画风；整张图只画这一幅独立分镜画面，禁止绘制漫画分格线、禁止任何台词气泡与文字」） |
| `generateAspectRatio` | 按分格在网格中的几何长宽比匹配：`16:9`（全宽横幅 `rowSpan:2, colSpan:6`）/ `2:3` 或 `3:4`（半宽竖格 `rowSpan:4, colSpan:3`）/ `1:1`（半宽方格 `rowSpan:2, colSpan:3`）/ `4:3`（跨幅大格 `rowSpan:4, colSpan:6`） |
| `generateCount` | 一次出几张（能力档位通常 1 / 2 / 4）；**漫画页只取 `out` 当前选中的那一张** |
| `generateModel` / `generateProviderInstanceId` | 覆盖模型与自建提供商 |
| `generateSeed` / `generateSeedUseGlobal` | 复现用；默认跟随工程全局种子 |
| `styleImages` / `styleImagesUseGlobal` / `styleReferenceSubject` / `characterRefs` | 风格 / 角色参考，条目必须带可解析的图 |
| `mediaOutputDir` | 覆盖落盘目录 |

## comic.page（漫画页）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-image` | `image`（**多**） | 本页各分镜图按阅读顺序逐个连一条边；执行时按阅读顺序自动填入或刷新分格 `imageUrl` |
| `in-video` | `video`（**多**） | 可选：接入高潮格微动视频（I2V），升级动态漫时自动绑定至对应分格 `videoUrl` |
| `out` / `out-all` | `image` / `images` | 合成后的高清漫画页 PNG 进图库 |

| 参数 | 说明 |
|---|---|
| `comicPage` | 页面数据的序列化 JSON 字符串（`defaultParams` 已声明，物化与 `graph_edit` 直接放行） |

### `comicPage` 完整数据结构（`serializeComicPage` 的产物）

```jsonc
{
  "title": "第 1 页 · 起承",         // 可选，页首标题
  "backgroundColor": "#ffffff",     // 可选；缺省或空串 = 导出透明底 PNG
  "columns": 6,                      // 网格列数（1~12），页漫推荐 6，竖屏条漫推荐 1
  "rows": 6,                         // 网格行数（1~12），页漫推荐 6，竖屏条漫推荐 8
  "gutter": 18,                      // 分格间距（0~96 px）
  "padding": 36,                     // 页面四周外边距（0~240 px，默认 36）
  "borderWidth": 3,                  // 全局分格描边宽度（0~24 px，默认 3）
  "borderColor": "#111111",          // 全局分格描边颜色（默认 #111111）
  "borderRadius": 0,                 // 全局分格圆角（0~64 px，默认 0）
  "readingDirection": "ltr",         // 阅读顺序：ltr（国漫/条漫左到右，默认）| rtl（日漫右到左）
  "width": 1080,                     // 页面像素宽（页漫/条漫推荐 1080，16:9 横屏漫推荐 1920）
  "height": 1520,                    // 页面像素高（页漫推荐 1520，长条漫推荐 2400，16:9 横屏漫推荐 1080）
  "panels": [
    {
      "id": "p1-panel-1",
      "row": 0, "col": 0,            // 网格起点（0-based）
      "rowSpan": 2, "colSpan": 6,    // 跨行数、跨列数
      "title": "格 1 · 开篇定场",     // 可选，无图时作为占位提示
      "caption": "可选：分镜底部旁白字幕条",
      "backgroundColor": "#f4f4f6",  // 可选，单格底色
      "borderWidth": 3,              // 可选，覆盖全局分格描边宽（0 = 无框出血格）
      "borderColor": "#111111",      // 可选，覆盖全局分格描边色
      "zIndex": 0,                   // 可选，叠放层级（-10 ~ 10）
      // 斜切破格：四角归一化偏移 [tlX, tlY, trX, trY, brX, brY, blX, blY]，每项 -0.45 ~ 0.45
      "cornerOffsets": [0, 0, 0, 0, 0, -0.1, 0, 0.08],
      // 镜头氛围特效：none | speed-lines（平行速度线）| focus-lines（放射集中线）| vignette（情绪暗角）| halftone-frame（网点边框）
      "effectType": "speed-lines",
      "effectIntensity": 0.6,        // 特效强度 0.1 ~ 1，默认 0.6
      "effectAngle": 0,              // speed-lines 角度 0 ~ 360
      "effectFocusX": 0.5,           // focus-lines 焦点 X（0 ~ 1）
      "effectFocusY": 0.5,           // focus-lines 焦点 Y（0 ~ 1）
      "effectColor": "#111111",      // 特效线条颜色（#111111 或 #ffffff）
      // 格内构图与漫画滤镜（可选）：
      "imageFit": "cover",           // cover（默认裁切铺满）| contain
      "imagePanX": 0, "imagePanY": 0,// 格内图片平移 -1 ~ 1
      "imageZoom": 1,                // 格内图片缩放 0.5 ~ 4
      "imageRotation": 0,            // 格内图片旋转 -180 ~ 180 度
      "imageFlipX": false,           // 水平镜像翻转
      "imageFilter": "none",         // none | manga-bw（黑白高反差日漫）| halftone（复古漫画网点纸）| sepia（怀旧泛黄）| dramatic（戏剧暗调）
      // 动态漫演出字段（可选，当接入 output.timeline 或点「生成动态漫」时生效）：
      "cameraMotion": "auto",        // auto | kenBurnsIn | kenBurnsOut | panLeft | panRight | panUp | panDown | none
      "durationSec": 3.0,            // 单格停留秒数（缺省按气泡台词与配音时长自动反推）
      "videoUrl": "",                // 动静混排视频相对路径
      "foregroundImageUrl": "",      // 2.5D 视差前景透明立绘 PNG 相对路径
      "parallaxDepth": 0.5,          // 2.5D 视差深度强度（0.1 ~ 1.0）
      // 注意：自动生产时不要预填 imageUrl，留空以便执行 comic.page 时自动填入上游生成的分镜图
      "bubbles": [
        {
          "id": "p1-b1",
          "text": "台词或音效正文（空白会被丢弃）",
          "speaker": "主角",          // 可选，说话人角色名（sfx 与 narration 通常不填）
          // 气泡形状：rounded（圆角）| oval（传统日漫椭圆）| shout（爆炸咆哮）| thought（云朵心声）| narration（方形旁白）| sfx（拟声词特效字）
          "shape": "rounded",
          // 排版方向：horizontal-tb（默认横排）| vertical-rl（日漫/古风从右向左竖排）
          "writingMode": "horizontal-tb",
          "x": 0.68, "y": 0.22,      // 格内归一化锚点 0~1
          "tail": "bl",              // 默认尾巴朝向：tl / tr / bl / br
          "tailTipX": 0.45,          // 可选，自由尾巴尖端归一化坐标（-0.5 ~ 1.5），指向说话角色嘴部
          "tailTipY": 0.42,
          "scale": 1,                // 气泡字号与尺寸缩放 0.5 ~ 4
          "rotation": 0,             // 可选，旋转角度 -60 ~ 60（常用于倾斜 sfx 音效字）
          "backgroundColor": "#ffffff", // 可选，自定义气泡填充底色
          "borderColor": "#111111",  // 可选，自定义气泡描边色
          "textColor": "#111111",    // 可选，自定义文字颜色
          "strokeWidth": 2.5,        // 可选，气泡描边粗细（0 ~ 12）
          "fontWeight": "bold"       // 可选，normal | bold | black
        }
      ]
    }
  ]
}
```

## 常见错误排查

| 现象 | 原因与解决 |
|---|---|
| 报 `GRAPH_COMIC_PAGE_EMPTY` | 页面没有任何分格，且上游没图可自动建格；检查 `comicPage` JSON 是否合法或上游图片节点是否成功出图 |
| 单格画面里被画成了多格或出现乱码文字 | 检查是否误把 `script` 连入了分镜节点的 `in-text`，或 `generateInstruction` 漏写了「整张图只画这一幅独立分镜画面，禁止绘制漫画分格线、禁止任何台词气泡与文字」 |
| 分镜人物头脚被裁切过多 | 分镜节点的 `generateAspectRatio` 与分格几何长宽比不匹配（例如横幅格用了 `1:1`）；按分格 `rowSpan × colSpan` 设置 `16:9` / `2:3` / `1:1` / `4:3` |
| 参数写了但没生效 | 检查键名是否拼错（如网点滤镜是 `imageFilter: "halftone"`、网点边框特效是 `effectType: "halftone-frame"`、气泡底色是 `backgroundColor`） |
