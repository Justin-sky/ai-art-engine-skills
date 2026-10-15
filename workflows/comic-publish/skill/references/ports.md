# 端口与参数速查（comic-publish）

字段名与端口 id 均取自应用源码，可直接写进 `graph_edit` 的 `params`。
节点 id 是随机生成的（`node-<uuid>`），改参数前先用 `graph_read`（`includeParams: true`）拿到真实 id。

## play.script（漫画剧本 / 文本节点）

| 端口 | 类型 | 说明 |
|---|---|---|
| `out` | `text`（多） | 文本出口，扇出到「角色与画风定妆」及全集各页分镜图节点 |

| 参数 | 说明 |
|---|---|
| `text` | 整集分场分格剧本正文；也接受 `generateInstruction` |

## asset.image（角色与画风定妆 / 各页分镜图）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-text` | `text`（多） | 提示词来源（`play.script:out` 接这里） |
| `in-image` | `image`（多） | 参考图（「角色与画风定妆」的 `out` 连入所有分镜图的 `in-image`，锁定跨页角色形象与统一画风） |
| `out` | `image` | 当前选中一张（新建生成后默认 = 最新一张） |
| `out-all` | `images`（多） | 全部历史，仅供 `image.select` 一族消费 |

| 参数 | 说明 |
|---|---|
| `generateInstruction` | 画面生成指令（分镜图请写清景别、动作、表情、环境光影，并注明「严格保持与参考图一致的角色形象与画风，纯画面不含任何文字」） |
| `generateAspectRatio` | 如 `2:3` / `3:4` / `16:9` / `1:1`；可根据分格横竖比例为单张分镜图设合适宽高比 |
| `generateCount` | 一次出几张（能力档位通常 1 / 2 / 4）；**漫画页只取 `out` 当前选中的那一张** |
| `generateModel` / `generateProviderInstanceId` | 覆盖模型与自建提供商 |
| `generateSeed` / `generateSeedUseGlobal` | 复现用；默认跟随工程全局种子 |
| `styleImages` / `styleImagesUseGlobal` / `styleReferenceSubject` / `characterRefs` | 风格 / 角色参考，条目必须带可解析的图 |
| `mediaOutputDir` | 覆盖落盘目录 |

## comic.page（漫画页）

| 端口 | 类型 | 说明 |
|---|---|---|
| `in-image` | `image`（**多**） | 本页各分镜图按阅读顺序逐个连一条边；执行时按阅读顺序填入尚未绑图的分格 |
| `out` / `out-all` | `image` / `images` | 合成后的漫画页 PNG 进图库 |

| 参数 | 说明 |
|---|---|
| `comicPage` | 页面数据的序列化 JSON 字符串（`defaultParams` 已声明，物化与 `graph_edit` 直接放行） |

### `comicPage` 完整数据结构（`serializeComicPage` 的产物）

```jsonc
{
  "title": "第 1 页 · 起承",         // 可选，页首标题
  "backgroundColor": "#ffffff",     // 可选；缺省或空串 = 导出透明底 PNG
  "columns": 6,                      // 网格列数（1~12），推荐 6 便于灵活切分
  "rows": 6,                         // 网格行数（1~12），推荐 6
  "gutter": 18,                      // 分格间距（0~96 px）
  "padding": 36,                     // 页面四周外边距（0~160 px，默认 36）
  "borderWidth": 3,                  // 全局分格描边宽度（0~24 px，默认 3）
  "borderColor": "#111111",          // 全局分格描边颜色（默认 #111111）
  "borderRadius": 0,                 // 全局分格圆角（0~64 px，默认 0）
  "readingDirection": "ltr",         // 阅读顺序：ltr（左到右，默认）| rtl（日漫右到左）
  "width": 1080,                     // 页面像素宽（240~4096，默认 1080）
  "height": 1520,                    // 页面像素高（240~4096，默认 1440/1520）
  "panels": [
    {
      "id": "p1-panel-1",
      "row": 0, "col": 0,            // 网格起点（0-based）
      "rowSpan": 2, "colSpan": 6,    // 跨行数、跨列数
      "title": "格 1 · 开篇定场",     // 可选，无图时作为占位提示
      "backgroundColor": "#f4f4f6",  // 可选，单格底色
      "borderWidth": 3,              // 可选，覆盖全局分格描边宽
      "borderColor": "#111111",      // 可选，覆盖全局分格描边色
      "zIndex": 0,                   // 可选，叠放层级（-20 ~ 20）
      // 斜切破格：四角归一化偏移 [tlX, tlY, trX, trY, brX, brY, blX, blY]，每项 -0.35 ~ 0.35
      "cornerOffsets": [0, 0, 0, 0, 0, -0.1, 0, 0.08],
      // 镜头氛围特效：none | speed-lines（速度线）| focus-lines（集中线）| vignette（暗角）| halftone（网点）
      "effectType": "speed-lines",
      "effectIntensity": 0.45,       // 特效强度 0.05 ~ 1，默认 0.45
      // 格内构图与滤镜（可选）：
      "imageFit": "cover",           // cover（默认裁切铺满）| contain
      "imagePanX": 0, "imagePanY": 0,// 格内图片平移 -1 ~ 1
      "imageZoom": 1,                // 格内图片缩放 0.5 ~ 4
      "imageRotation": 0,            // 格内图片旋转 -180 ~ 180 度
      "imageFlipX": false,           // 水平镜像翻转
      "imageFilter": "none",         // none | manga-bw（黑白日漫）| high-contrast（高对比）| sepia（复古泛黄）
      // 注意：自动生产时不要预填 imageUrl，留空以便执行 comic.page 时自动填入上游生成的分镜图
      "bubbles": [
        {
          "id": "p1-b1",
          "text": "台词或音效正文（空白会被丢弃）",
          "speaker": "主角",          // 可选，说话人标签（sfx 与 narration 通常不填）
          // 气泡形状：rounded（圆角）| oval（椭圆）| shout（爆炸咆哮）| thought（云朵心声）| narration（方形旁白）| sfx（拟声词大字）
          "shape": "rounded",
          // 排版方向：horizontal-tb（默认横排）| vertical-rl（日漫/古风竖排）
          "writingMode": "horizontal-tb",
          "x": 0.68, "y": 0.22,      // 格内归一化锚点 0~1
          "tail": "bl",              // 默认尾巴朝向：tl / tr / bl / br
          "tailTipX": 0.45,          // 可选，自由尾巴尖端归一化坐标（-0.5 ~ 1.5），指向说话角色嘴部
          "tailTipY": 0.42,
          "scale": 1,                // 气泡字号与尺寸缩放 0.5 ~ 4
          "rotation": 0,             // 可选，旋转角度 -180 ~ 180（常用于倾斜 sfx 音效字）
          "bgColor": "#ffffff",      // 可选，自定义气泡底色
          "borderColor": "#111111",  // 可选，自定义气泡描边色
          "textColor": "#111111"     // 可选，自定义文字颜色
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
| 分镜图重跑后漫画页没换新图 | `comic.page` 默认只填未绑图（`!imageUrl`）的分格；重跑前先清空对应分格的 `imageUrl` |
| 参数写了但没生效 | 键名拼错会被当作"未声明参数"丢弃，检查 `graph_edit` 返回的 `warnings` |
| 画面里出现了乱码文字 | 分镜图的 `generateInstruction` 忘了写「纯画面不含任何文字」，把台词放进 `bubbles` 而不是让生图模型画字 |
