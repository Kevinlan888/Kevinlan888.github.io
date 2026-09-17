# 高数笔记更新说明

博客入口：`content/posts/calculus-basics.md`。正文按主题保存在同目录下的 `calculus-basics-*.md` 中，各篇是对应内容的唯一维护源，不必同步修改一份完整长文。

## 修改已有内容

1. 找到对应主题文件，在原章节下补充说明、例题或易错点。
2. 保留 `date`（首次发布日期），把该篇的 `lastmod` 改成实际更新日期。页尾会自动显示该日期。
3. 若有重要补充，在总目录的“更新记录”写一条简述，并更新总目录的 `lastmod`。
4. 保留文件名和已有的 `{#section-N}` 锚点，避免收藏链接失效。

原文 1—7 节在 algebra，8—10 节在 exponents-logarithms，11—15 节在 functions，16—19 节在 function-relations，20 节在 limits，21 节在 inverse-example，22 节在总目录。

## 新增主题

复制一篇主题笔记的 front matter，修改标题、简介、日期和 `studyorder`，保留 `studyseries = 'calculus-basics'`。`studyorder` 只控制系列阅读顺序，使用不重复的数值；0 留给总目录。正文结尾保留 `{{< study-nav >}}`，上一篇和下一篇会自动按顺序生成。将新篇加入总目录表格。

已有章节不必重新编号。后续独立主题可以使用不带原章节编号的标题，并设置清晰、稳定的锚点。

## 数学公式与验证

- 行内使用 `$...$`，独立公式使用单独成行的 `$$` 包围。
- Hugo 的 passthrough 扩展保留 LaTeX，由 `layouts/_markup/render-passthrough.html` 在构建时生成 MathML。无需远端数学脚本或字体。
- 普通文字中的美元符号可以写成 `&#36;`，代码中的美元符号放在反引号内，避免误识别为公式。
- 本地使用与 `.github/workflows/hugo.yaml` 相同版本的 Hugo，执行 `hugo --gc --minify`；检查目录跳转、公式、表格与手机宽度下的横向滚动。
- 推送到 `main` 会触发现有 GitHub Pages 发布流程，发布后检查工作流结果和博客页面。
