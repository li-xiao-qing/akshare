# AI 生成文档采用 Markdown 还是 HTML：互联网与 GitHub 实践调研

调研日期：2026-07-17

调研对象：AI 生成的研究报告、技术文档和本仓库投资分析报告。

## 1. 结论先行

没有形成“所有 AI 文档只能使用 Markdown”或“HTML 应取代 Markdown”的统一结论，但主流工具和大型文档项目呈现出非常一致的分工：

```text
Markdown / MDX / Quarto / 结构化数据
                ↓ 构建或渲染
HTML 网站 / 单页报告 / PDF / DOCX
```

对本仓库最合适的策略是：

1. **Markdown 继续作为可审计的事实源和 Git 版本。**
2. **HTML 作为面向阅读的生成产物。**它可以有侧边目录、图表、折叠块、打印样式和响应式布局。
3. **不要让 AI 独立维护两份正文。**HTML 必须由 Markdown、结构化数据和固定模板自动生成，否则数值、时点和结论会漂移。
4. 对每日投资报告，进一步推荐“结构化数据 + Markdown 论证 + HTML 模板”的三层模型。

当前 `dca_analysis_20260717.html` 的阅读体验明显优于 Markdown，但它不应取代 `dca_analysis_20260717.md` 成为唯一源文件。

## 2. 为什么 AI 和版本管理更偏向 Markdown

### 2.1 Markdown 保留结构，但标记噪声较少

CommonMark 将 Markdown 定义为用于编写结构化文档的纯文本格式，并提供明确规范和测试用例。GitHub Flavored Markdown 在此基础上增加表格、删除线、任务列表等能力。[CommonMark](https://commonmark.org/)、[GFM 规范](https://github.github.com/gfm/)

相对于 HTML，Markdown 的标题、列表、表格和链接通常需要更少字符，因此：

- AI 更容易生成和局部修改；
- Git diff 更接近“内容发生了什么变化”；
- 合并冲突较少；
- 原始文件不经渲染仍然可读；
- 内容可以转换到 HTML、PDF、DOCX 等不同输出。

Microsoft 的 MarkItDown 明确将多种文件转换为 Markdown，目标是供 LLM 和文本分析流程使用。项目同时强调，其输出虽然通常可读，但并不是为面向人的高保真转换设计的。这恰好说明了 Markdown 与 HTML 的分工：前者更适合机器处理和内容中间层，后者更适合精细展示。[Microsoft MarkItDown](https://github.com/microsoft/markitdown)

### 2.2 AI 文档工具通常不会把 HTML 当作唯一内部格式

Docling 先把不同输入解析为统一的 `DoclingDocument` 表示，再支持 Markdown、HTML、无损 JSON 等多种导出。这种架构避免把展示格式当成事实模型。[Docling](https://github.com/docling-project/docling)

GitHub Docs API 允许程序和 AI 直接获取文章的 Markdown，并提供 `llms.txt` 作为内容发现入口。这说明即使最终页面是 HTML，GitHub 仍把 Markdown 作为机器消费的重要接口。[GitHub Docs API](https://docs.github.com/en/get-started/using-github-docs/github-docs-api)

`llms.txt` 也是一个使用 Markdown 的新兴提案，目的是为 LLM 提供比复杂网页更紧凑的内容入口。但它目前是提案，不是经过标准组织批准的正式互联网标准，不能把其流行程度夸大为行业强制规范。[llms.txt 原始提案](https://llmstxt.org/)、[GitHub 仓库](https://github.com/answerdotai/llms-txt)

## 3. 为什么面向人的最终阅读更适合 HTML

HTML、CSS 和 JavaScript 可以提供 Markdown 原文无法稳定表达的能力：

- 固定视觉层级、字体、颜色和留白；
- 响应式手机与桌面布局；
- 粘性目录、折叠内容、标签、筛选和交互；
- 更复杂的表格、图表和数据可视化；
- 打印样式和 PDF 导出；
- 完整的可访问性语义和焦点行为；
- 单文件离线阅读，或发布为可搜索的网站。

MkDocs 的官方描述就是“Markdown 源文件生成完全静态的 HTML 网站”，并提供实时预览服务器。[MkDocs](https://www.mkdocs.org/)

Docusaurus 使用 Markdown 作为主要写作格式，再通过 MDX 编译为 React 组件，以支持交互式文档。[Docusaurus Markdown](https://docusaurus.io/docs/markdown-features)

Quarto 使用 Pandoc Markdown 作为底层语法，可以渲染成包含主题、目录、响应式图片和代码交互的 HTML。[Quarto Markdown](https://quarto.org/docs/authoring/markdown-basics.html)、[Quarto HTML](https://quarto.org/docs/output-formats/html-basics)

Pandoc 官方教程直接演示了将 `.md` 转换成独立 `.html` 文件。这是“写作源”和“阅读产物”分离的典型实现。[Pandoc 入门](https://pandoc.org/getting-started.html)

## 4. GitHub 上真实存在的争议与边界

### 4.1 Markdown 对复杂表格表达不足

Docling 的 issue 显示，Markdown 表格通常只能导出为文本表格，而复杂表格图片、合并单元格和多行内容需要 HTML 或专用序列化器。[Docling Core #474](https://github.com/docling-project/docling-core/issues/474)

另一个 Docling issue 讨论了 Markdown 表格单元格不能可靠表示换行，只能注入 `<br>` 或采用查看器相关的语法，而不同 Markdown 渲染器未必一致支持。[Docling #1927](https://github.com/docling-project/docling/issues/1927)

Pandoc 的 issue 也指出，复杂的表格行组已经超出可读 Markdown 的合理能力，HTML 表格是可选方案。[Pandoc #9856](https://github.com/jgm/pandoc/issues/9856)

这些讨论支持的不是“放弃 Markdown”，而是：**简单正文继续使用 Markdown，复杂展示交给 HTML 组件或模板。**

### 4.2 HTML 与 Markdown 混写会降低可移植性

CommonMark 和 GFM 允许一定程度的原始 HTML，但 GitHub 在渲染后还会进行安全清理，因此不是所有 HTML、属性、样式和脚本都会被保留。[GFM 规范](https://github.github.com/gfm/)

Docusaurus 的 MDX 可以嵌入 React 组件，但官方文档明确提醒：MDX 比 CommonMark 严格，部分合法 Markdown 和原始 HTML 语法不能直接使用，还可能产生编译错误。[Docusaurus MDX](https://www.docusaurus.io/docs/3.8.1/markdown-features/react)

Docusaurus 的 GitHub discussion 也展示了直接链接或混入静态 HTML 文件时，需要理解构建目录、静态资源和最终 URL 之间的关系。[Docusaurus #7734](https://github.com/facebook/docusaurus/discussions/7734)

因此，在 Markdown 中大量内嵌任意 HTML 虽然短期方便，却可能失去跨 GitHub、Sphinx、MkDocs、Pandoc 和其他查看器的一致性。

### 4.3 自包含 HTML 很适合交付，但仍应被视为构建目标

Pandoc 专门支持独立 HTML 和资源嵌入。相关 issue 讨论了在 CMS、模板和单文件交付场景下，如何区分页面骨架与资源嵌入，说明自包含 HTML 是真实且重要的交付需求。[Pandoc #7331](https://github.com/jgm/pandoc/issues/7331)

但 HTML 中每次内容修改都可能同时改动标签、样式、脚本和正文。若直接让 AI 重写整个文件，Git diff 会出现大量与事实无关的变化，审查成本高于 Markdown。

## 5. 互联网与 GitHub 实践归纳

| 项目或规范 | 内容源/内部表示 | 阅读输出 | 对本问题的启示 |
| --- | --- | --- | --- |
| CommonMark / GFM | Markdown | HTML 渲染 | Markdown 适合作为可移植结构化文本，但存在方言和安全清理差异。 |
| GitHub Docs | Markdown、YAML、Liquid | GitHub Docs HTML 网站 | 大型文档项目也保留 Markdown 源，并向 AI 提供 Markdown API。 |
| GitHub Pages / Jekyll | Markdown、模板 | HTML 网站 | Markdown 到 HTML 是官方支持的发布路径。 |
| MkDocs | Markdown、YAML 配置 | 静态 HTML | 典型 docs-as-code 工作流。 |
| Docusaurus | Markdown / MDX | React/HTML 文档站 | 需要交互时可升级为 MDX，但工具复杂度和语法严格度上升。 |
| Quarto | Pandoc Markdown / QMD | HTML、PDF、DOCX 等 | 适合研究报告和可重复分析。 |
| Pandoc | 文档 AST、多种输入 | HTML、PDF、DOCX 等 | 适合单文件转换和多格式交付。 |
| Microsoft MarkItDown | 多种输入转 Markdown | Markdown | Markdown 面向 LLM 和文本分析，不追求高保真人类展示。 |
| Docling | 统一文档模型 | Markdown、HTML、JSON 等 | 正确架构是一个内容模型、多种导出，而不是选一个格式承担全部职责。 |
| llms.txt | Markdown 提案 | AI 读取入口 | Markdown 对 AI 友好，但该约定仍是新兴提案。 |

## 6. Markdown、HTML 与混合方案对比

| 维度 | Markdown | 手写/AI直接生成HTML | Markdown自动生成HTML |
| --- | --- | --- | --- |
| 人工阅读原文 | 好 | 差 | 好 |
| 浏览器成品体验 | 中 | 最好 | 最好 |
| AI生成稳定性 | 好 | 中，容易产生结构和样式噪声 | 好 |
| Git diff与审查 | 最好 | 最差 | 源文件好，产物可不审 |
| 复杂布局和交互 | 弱 | 最强 | 强，取决于模板 |
| 多格式输出 | 强 | 弱 | 强 |
| 安全处理 | 相对简单，仍需处理链接和原始HTML | 需要防范脚本、事件属性和不可信内容 | 可集中在渲染器和模板处理 |
| 内容一致性 | 单文件时好 | 单文件时好 | 自动生成时最好 |
| 长期维护 | 低 | 高 | 中，首次建立模板后较低 |

## 7. 对当前投资报告的具体审查

### 7.1 仓库本身采用的文档模式

本仓库已经在使用“文本源生成 HTML”的模式：

- `docs/conf.py` 将 `.rst` 和 `.md` 都配置为 Sphinx 文档源；
- `docs/README.rst` 使用 `make html` 生成 `build/html`，并将其称为生成后的 HTML 文档；
- `.gitignore` 忽略 `docs/build/` 和 `docs/_build/`；
- 当前 Git 跟踪的文档以 Markdown 为主，没有把构建目录中的 HTML 当作正文源文件。

因此，推荐方案不是在仓库中引入完全不同的理念，而是把现有 docs-as-code 模式延伸到投资报告。

### 7.2 当前文件状态

| 文件 | 状态 | 大小 | 行数 | 定位 |
| --- | --- | ---: | ---: | --- |
| `dca_analysis_20260717.md` | 已进入 Git | 20,568字节 | 303 | 完整分析和来源记录 |
| `dca_analysis_20260717.html` | 当前未跟踪 | 47,140字节 | 921 | 自包含阅读页面 |

HTML 约为 Markdown 的`2.3倍`大小和`3倍`行数，原因是包含完整 CSS、HTML 结构和 JavaScript。

### 7.3 HTML 的实际价值

当前 HTML 不是简单的 Markdown 预览，而是一个质量较高的阅读成品，包含：

- 侧边目录和滚动定位；
- 组合 KPI 与亏损归因图；
- 风险标签和行动状态；
- 可折叠的对抗式审查；
- 打印样式；
- 桌面和移动端响应式布局；
- 无外部前端依赖的单文件结构。

因此，删除 HTML、只保留 Markdown 会损失真实的阅读价值。

### 7.4 当前最大风险

该 HTML 目前不是由 Markdown 的确定性构建脚本生成，而是再次组织和改写了正文。以后如果正式净值、港股收盘价或投资结论发生修正，可能只改到其中一个文件。

对于投资报告，这不仅是普通排版问题，而是审计问题：同一个日期出现两份不同数字，会使后续复盘失去可信度。

## 8. 推荐的目标架构

### 8.1 最低成本方案

```text
dca_analysis_20260717.md        # 唯一事实源，提交 Git
        ↓ render
build/fund_investment/
  dca_analysis_20260717.html    # 自动生成，通常不提交 Git
```

优点是简单、与仓库现有 Sphinx 思路一致。缺点是普通 Markdown 渲染不能自动得到当前 HTML 中所有定制图形。

### 8.2 推荐方案

```text
report data / YAML front matter
        +
Markdown 分析正文
        +
固定 HTML 模板与 CSS
        ↓
自包含 HTML + 可选 PDF
```

建议按以下职责拆分：

- 持仓金额、指数涨跌、估值和损益归因：结构化数据；
- 第一性原理论证、事实/推断/未知、行动建议：Markdown；
- 卡片、图表、侧边目录、打印和响应式布局：HTML 模板；
- 校验脚本：检查金额合计、百分比、日期和来源链接；
- 构建命令：一次生成 HTML，并提供 localhost 预览地址。

### 8.3 是否提交 HTML

优先级建议：

1. **最佳：**提交 Markdown、数据和模板；通过 GitHub Pages 或构建产物发布 HTML。
2. **可接受：**同时提交生成后的 HTML，但文件顶部标明“自动生成，请勿手工编辑”，CI 检查它与源文件一致。
3. **不推荐：**Markdown 与 HTML 都由 AI 独立编写并分别维护。
4. **最不推荐：**只保留 HTML，删除 Markdown 和结构化事实源。

## 9. 最终判断

```text
面向 AI 和 Git：Markdown 更合适。
面向人类阅读：HTML 更合适。
面向长期可靠性：Markdown/数据生成 HTML 最合适。
```

你的直觉“HTML 更适合观看”是正确的，但由此得出“以后只生成 HTML”是不完整的。真正应优化的是生成流水线：让 AI 只维护一次事实和论证，再由稳定模板产出可点击、可打印、可交互的 HTML。

## 10. 主要来源

- [CommonMark](https://commonmark.org/)
- [GitHub Flavored Markdown 规范](https://github.github.com/gfm/)
- [GitHub Docs 写作体系](https://docs.github.com/en/contributing/writing-for-github-docs)
- [GitHub Docs API](https://docs.github.com/en/get-started/using-github-docs/github-docs-api)
- [GitHub Pages 与 Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll)
- [Microsoft MarkItDown](https://github.com/microsoft/markitdown)
- [Docling](https://github.com/docling-project/docling)
- [MkDocs](https://www.mkdocs.org/)
- [Material for MkDocs](https://github.com/squidfunk/mkdocs-material)
- [Docusaurus Markdown](https://docusaurus.io/docs/markdown-features)
- [Quarto Markdown](https://quarto.org/docs/authoring/markdown-basics.html)
- [Pandoc 入门](https://pandoc.org/getting-started.html)
- [llms.txt 提案](https://llmstxt.org/)
