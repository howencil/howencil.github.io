---

title: "Hugo文章模板的前置参数"
date: 2022-06-06T14:35:55+08:00
draft: false
author: 插画师
tags: ["hugo"]
---

- **title**: 文章标题.
- **subtitle**: 文章副标题.
- **date**: 这篇文章创建的日期时间. 它通常是从文章的前置参数中的 `date` 字段获取的, 但是也可以在 [网站配置](https://shuzang.github.io/2019/theme-documentation-basics/#site-configuration) 中设置.
- **lastmod**: 上次修改内容的日期时间.
- **draft**: 如果设为 `true`, 除非 `hugo` 命令使用了 `--buildDrafts`/`-D` 参数, 这篇文章不会被渲染.
- **author**: 文章作者.
- **authorLink**: 文章作者的链接.
- **description**: 文章内容的描述.
- **license**: 这篇文章特殊的许可.
- **tags**: 文章的标签.
- **categories**: 文章所属的类别.
- **hiddenFromHomePage**: 如果设为 `true`, 这篇文章将不会显示在主页上, 但是此行为可以在 [网站配置](https://shuzang.github.io/2019/theme-documentation-basics/#site-configuration) 中设置的.
- **featuredImage**: 文章的特色图片.
- **featuredImagePreview**: 用在主页预览的文章特色图片.
- **toc**: 如果设为 `true`, 这篇文章会显示右侧目录.
- **autoCollapseToc**: 如果设为 `true`, 文章目录会自动折叠.
- **math**: 如果设为 `true`, 将自动渲染文章中的数学公式.
- **mapbox**: 和 [网站配置](https://shuzang.github.io/2019/theme-documentation-basics/#site-configuration) 中的 `params.mapbox` 对象相同.
- **lightgallery**: 如果设为 `true`, 文章中的图片将可以按照画廊形式呈现.
- **linkToMarkdown**: 如果设为 `true`, 内容的页脚将显示指向原始 Markdown 文件的链接.
- **share**: 和 [网站配置](https://shuzang.github.io/2019/theme-documentation-basics/#site-configuration) 中的 `params.share` 对象相同.
- **comment**: 如果设为 `true`, 将启用评论系统.

> [参考文章](https://shuzang.github.io/2019/hugo-blog-article-write/)
