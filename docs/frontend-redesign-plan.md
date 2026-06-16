# Frontend Redesign Plan

本文档记录一次针对本站前端审美与阅读体验的改版方案。目标是在不引入重型前端框架、不改变静态站点定位、不编辑 upstream theme submodule 的前提下，让站点更素雅、现代、可读。

## 调研结论

本站是一个轻量 Hugo personal website，当前有效渲染层集中在 `layouts/`：

- `layouts/partials/style.html`：全站主样式，当前为内联 CSS。
- `layouts/_default/baseof.html`：页面壳、SEO、header/footer、全局 edited date。
- `layouts/_default/list.html`：Blog/Tech 等列表页。
- `layouts/_default/single.html`：单篇文章和普通页面。
- `layouts/index.html`：首页。
- `layouts/partials/nav.html`：顶部导航。

当前视觉问题不是内容不足，而是层级和节奏偏接近浏览器默认态：

- Header、导航、更新时间、标题、正文、图片之间的视觉强度区分不够。
- Blog 列表清楚但偏机械，日期和标题只是简单并排。
- 中文长文的行距、段落间距、目录样式还有优化空间。
- Gallery 照片质量不错，但现在只是普通 Markdown 图片流，缺少相册感。
- 暗色模式可用，但颜色体系较粗，代码块、引用、边线等细节可以更统一。

## 设计原则

1. 保持 Bearblog-inspired 的轻量形态。
2. 不引入 React、Vue、Tailwind、Bootstrap、npm build pipeline 或客户端 app shell。
3. 优先使用 Hugo template、Markdown、shortcode 和 CSS。
4. 保持内容为主，装饰为辅。
5. 改动集中在 local layout overrides，不编辑 `themes/hugo-bearblog/`。
6. 不编辑 `public/` 生成物。

## 推荐方案

### 1. 建立轻量 CSS token

在 `layouts/partials/style.html` 中增加 CSS custom properties：

- `--bg`
- `--text`
- `--muted`
- `--heading`
- `--link`
- `--link-hover`
- `--border`
- `--surface`
- `--code-bg`

用这套 token 统一浅色和暗色模式。色彩建议使用近白背景、深灰正文、低饱和蓝绿链接、浅灰边线，避免强烈的品牌色和大面积渐变。

### 2. 优化全站排版

建议将当前 `Verdana, sans-serif` 调整为现代 system font stack，例如：

```css
font-family:
  ui-sans-serif,
  system-ui,
  -apple-system,
  BlinkMacSystemFont,
  "Segoe UI",
  sans-serif;
```

正文最大宽度仍保持在个人站适合阅读的范围，建议 `max-width: 720px` 到 `760px`。中文长文重点优化：

- 正文 `line-height` 约 `1.75`。
- 段落之间保留稳定间距。
- 标题上方间距大于下方间距，形成自然章节节奏。
- 链接使用下划线或 subtle underline offset，而不是大面积背景 hover。

### 3. Header 与导航

当前 header 足够简洁，但视觉层级略硬。建议：

- 站点标题保持文字 logo，不引入图片 logo。
- 标题字号略收敛，字重稳定。
- 导航改为更轻的 inline nav。
- 增加当前页状态、hover、focus-visible。
- 导航在小屏幕允许自然换行，避免挤压。

另外，`content/hidden/_index.md` 当前有 `menu = "main"`，所以 Hidden 会出现在主导航。若 Hidden 只是可访问但不想显式展示，可以移除这个 front matter 字段。这属于内容策略，不建议在纯视觉改版中擅自修改。

### 4. 列表页

Blog/Tech 列表建议做成安静的 archive，而不是卡片流：

- 日期弱化为 muted 色。
- 标题保持主信息。
- 每条之间使用细分隔线或稳定垂直间距。
- 如果 front matter 有 `description`，可以显示一行摘要。
- 移动端改为日期在上、标题在下，避免窄屏横向挤压。

### 5. 文章页

文章页重点是阅读体验：

- 日期、Last edited、tags 统一成 metadata 样式。
- TOC 做成轻量目录块，不使用大边框卡片。
- `blockquote` 使用细左边线和 muted 文本。
- `code` 和 `pre` 增强暗色模式可读性。
- 图片增加一致的边界、圆角和下方说明空间。
- 表格增加横向滚动容器或基础边线样式。

### 6. Gallery

Gallery 是最值得单独优化的页面。当前 Markdown 写法是：

```markdown
![guabi road](/images/gallery/guabi_road.jpg)
__挂壁公路__(Guabi Gonglu), Pingshun county, Shanxi, China, 2017/07/22

Had never realized there was such a stunning view in my hometown.
```

推荐引入一个轻量 Hugo shortcode，例如 `gallery-item`，生成语义化结构：

```html
<figure class="gallery-item">
  <img src="..." alt="...">
  <figcaption>
    <strong>...</strong>
    <span>...</span>
    <p>...</p>
  </figcaption>
</figure>
```

这样可以让照片、标题、地点、日期、说明形成更清晰的相册单元。视觉上建议：

- 图片保留原始内容感，不做重滤镜。
- 使用很轻的圆角、边线或阴影。
- 标题与地点日期分层。
- 单列布局即可，不建议上 masonry 或 JS lightbox。

## 移动端阅读体验

已考虑移动端体验。调研时用本地 Hugo 服务打开站点，并通过 Chrome headless / DevTools Protocol 检查了 390px 宽度下的首页、Gallery 和文章页。

检查结果：

- `window.innerWidth` 为 390。
- `document.documentElement.scrollWidth` 为 390。
- `document.body.scrollWidth` 为 390。
- 没有检测到真实横向溢出元素。

这说明当前页面没有明显的移动端横向滚动 bug。之前普通 headless 截图中出现的右侧裁切，更像是截图命令窗口参数导致的显示问题，而不是页面真实布局溢出。

但移动端仍有可优化点：

- 首页标题略重，首屏信息节奏偏直。
- 导航在小屏幕上可读，但需要更明确的换行和点击目标。
- 中文长文需要更舒服的行高和段落间距。
- TOC 在移动端应更轻，不应抢正文注意力。
- Gallery 图片和说明之间需要形成图文单元，否则移动端会像普通正文连续堆叠。
- 代码块和表格需要保证横向滚动，而不是撑破页面。

移动端实现建议：

- 使用 `box-sizing: border-box` 全局约束。
- `body` 使用 `padding: clamp(18px, 5vw, 32px)`。
- 图片统一 `display: block; max-width: 100%; height: auto;`。
- 列表页在小屏下改为单列堆叠。
- 导航使用 `display: flex; flex-wrap: wrap; gap: ...`。
- 代码块使用 `overflow-x: auto`，普通正文仍保持自然换行。

## 可选配置清理

当前 Hugo 会提示：

- `languageCode` deprecated，未来建议迁移到 `locale`。
- `.Site.LanguageCode` deprecated，模板中建议改为 `.Site.Language.Locale`。

这不是视觉改版的前置条件，但可以在改模板时顺手清理。

## 实施顺序

建议分三步实施：

1. 先改 `style.html`：完成设计 token、排版、导航、文章元素、暗色模式。
2. 再改列表和单页模板：优化 metadata、TOC、列表结构。
3. 最后处理 Gallery：新增 shortcode 或专用布局，并迁移 `content/gallery.md`。

每一步都应运行：

```sh
hugo --gc --minify
```

并检查至少以下页面：

- `/`
- `/blog/`
- `/2024-summary/`
- `/tech/`
- `/tech/inv_idx/`
- `/gallery/`
- `/resume/`

## 不建议做的事

- 不引入 SPA。
- 不引入大型 CSS framework。
- 不做营销式 hero。
- 不做大面积渐变、装饰光斑或复杂动画。
- 不把文章列表改成厚重卡片流。
- 不依赖 GitHub Pages 之外的动态能力。

