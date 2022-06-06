---
title: "Hugo添加tags等分类"
date: 2022-06-06T14:53:34+08:00
draft: false
author: 插画师
tags: ["hugo"]
categories: ["Tech"]
---

## 自定义分类

Hugo是支持用户自定义分类的，这个称为taxonomy，可以来对网页内容进行逻辑划分，详情可以在[这里](https://gohugo.io/content-management/taxonomies/)查看。

分类taxonomy有3个概念：

- **Taxonomy 分类**: 可以用来对内容进行分类的类别
- **Term 术语**: 分类的一个键
- **Value 值**: 分配给这个Term的具体内容

例如我需要增加3个分类，分别是：

- tag：文章标签
- topic：文章主题/文章系列
- category：文章分类

以tag为例，则对应Taxonomy是tag，Term是具体标签内容比如docker或者hugo，Value是打上这个标签的对应网页。

### 配置分类

需要在 `config.toml` 中增加分类。还是这个例子，则需要增加如下内容：

```
[taxonomies]
  tag = "tags"
  series = "series"
  category = "categories"
```

而将每个post的头部也相应增加对应的分类，例如这篇的头部就相应为：

```yaml
date: "2018-01-09T16:22:25+08:00"
draft: false
title: "Blog养成记(4) 增加tags等分类"
tags: ["hugo"]
series: ["Blog养成记"]
categories: ["杂技浅尝"]
```

当然实际上，Hugo默认会产生 `tags` 和 `categories` 的分类，如果只需要这两个，可以不用在 `config.toml` 中声明就在post头部使用。

### 分类集合查看

使用分类taxonomy之后，Hugo会使用分类的模板 (taxonomy templates) 来自动生成一个显示所有分类的term术语的网页以及一个显示该术语的所有value内容列表网页。

还是以tag为例：

`example.com/tags/` 会列出tags中的所有术语；

`example.com/tags/docker` 会列出tags标为docker的所有网页列表。

### 分类排序

> 分类排序还未正式尝试，无法确认正式效果，还需后面确认后再补充。

在内容页头部中以 `术语名称_weight` 来标志权重，可在分类的查看野种按该权重进行排序。但暂时只支持默认情况下以日期作为权重排序标准。

还是以这篇博文为例：

```yaml
date: "2018-01-09T16:22:25+08:00"
draft: true
title: "Blog养成记(4) 增加tags、topics和categories"
tags: ["hugo"]
tags_weight: 66
series: ["Blog养成记"]
series_weight: 96
categories: ["浅尝杂技"]
categoryes_weight: 96
```

## 网页布局修改

我的Hugo源码在[这里](https://github.com/orianna-zzo/blog-hugo)，可以进行参考。

我现在使用的是[cocoa主题](https://github.com/nishanths/cocoa-hugo-theme)，该主题并不会显示tags等分类，因此对于网页布局需要进行覆盖。

我先将 `themes/cocoa/layouts/blog` 中的 `single.html` 复制到 `layouts/blog` 中，只需要对主文件夹中 `layouts` 文件夹中进行修改，修改后的布局会自动覆盖主题中的布局。

网页布局主要涉及到Hugo的模板，可对现有主题的布局进行的修改有很多，具体语法和教程可参考官网对于[模板的介绍](https://gohugo.io/templates/)，现暂且从简单入手，只考虑对博客正文页增加tags、topics、categories，并增加table of contents作为目录导航，因此只需要考虑single page template单页模板，[此处](https://gohugo.io/templates/single-page-templates/)有官网具体说明。

#### 显示分类

在显示文章的元信息块 (`<div class="meta">`) 中，在阅读时间下面 (`{{ if not .Site.Params.noshowreadtime }}...{{ end }}`)，增加显示tags、topics和categories的代码：

```html
{{ with .Params.tags }}
<ul id="tags">
  {{ range . }}
    <li> <a href="{{ "tags" | absURL }}/{{ . | urlize }}">{{ . }}</a> </li>
  {{ end }}
</ul>
{{ end }}

{{ with .Params.topics }}
<ul id="topics">
  {{ range . }}
    <li> <a href="{{ "topics" | absURL }}/{{ . | urlize }}">{{ . }}</a> </li>
  {{ end }}
</ul>
{{ end }}

{{ with .Params.categories }}
<ul id="categories">
  {{ range . }}
    <li><a href="{{ "categories" | absURL}}/{{ . | urlize }}">{{ . }}</a> </li>
  {{ end }}
</ul>
{{ end }}
```

这样，这篇博文头部定义的tag、topic和category就都显示出来了。

#### 增加page-tag CSS类

虽然显示出来了，但是显示格式需要通过css定义呀。在 `static/css` 文件夹中的 `override.css`中增加 `page-tag`的css类：

```css
.page-tag {
    font-size: 14px;
    font-family: 'Raleway', 'Helvetica Neue', 'Arial', sans-serif;
    letter-spacing: -0.005rem;
    font-weight: 400;
    color: #666 !important;
    font-style: normal;
    margin-right: 5px;
    margin-left: 0px;
    display: flex;
  }
```

将每一个显示分类的代码都放入一个div块，并使用page-tag作为css类。

#### 布局修改

虽然现在都显示出来了，但是都没有放在我想要的位置。我对前端一知半解，每次调试都耗费不少时间，如果你对这些很熟悉，完全可以自己试着玩玩，我更多的目的为了记录。

首先，因为meta类中定义了 `display: flex`，所以这个div中的div都是浮动的，无法占有一行，所以先在自定义css中对meta的 `display` 重新定义：

```css
.meta {
    display:block !important;
}
```

这样meta类的div中的所有div都占有一行，但是如此一来，显示post时间和显示阅读时间也分成了两行，所以对于这两部分，在外面也套一个div，定义style的 `display: flex`。同样的，我也希望topic和category也在一行，所以做同样的操作。

再加上一些局部布局样式修改以及若标签内没有内容就不显示这部分内容，这一块代码现在应为：

```python
 1<div class="meta">
 2    <div style="display: flex">
 3        {{ if and .Site.Params.dateformfull .Site.Params.dateform }}
 4        <div class="date" title='{{ .Date.Format .Site.Params.dateformfull }}'>{{ .Date.Format .Site.Params.dateformfull }}</div>
 5        {{ else }}
 6        <div class="date" title='{{ .Date.Format "Mon Jan 2 2006 15:04:05" }}'>{{ .Date.Format "Jan 2, 2006" }}</div>
 7        {{ end }}
 8      
 9        {{ if not .Site.Params.noshowreadtime }}
10        <div class="reading-time"><div class="middot"></div>{{ i18n "readingTime" .ReadingTime }}</div>
11        {{ end }}
12    </div>
13    <div style="display: flex;">
14        {{ if .Params.series }}
15        <div class="page-tag">
16            <div class="page-tag">topic:</div>
17            {{ with .Params.series }}
18            <div class="page-tag" id="series">
19                {{ range . }}
20                <a href="{{ "series" | absURL }}/{{ . | urlize }}">{{ . }}</a> 
21                {{ end }}
22            </div>
23            {{ end }}
24        </div>
25        {{ end }}
26
27        {{ if .Params.categories }}
28        <div class="page-tag">
29            <div class="page-tag">category: </div>
30            {{ with .Params.categories }}
31            <div class="page-tag" id="categories">
32                {{ range . }}
33                <a href="{{ "categories" | absURL}}/{{ . | urlize }}">{{ . }}</a> 
34                {{ end }}
35            </div>
36            {{ end }}
37        </div>
38        {{ end }}
39    </div>
40
41    {{ if .Params.tags }}
42    <div class="page-tag">
43        <div class="page-tag">tags: </div>
44        {{ with .Params.tags }}
45        <div class="page-tag" id="tags">
46            {{ range . }}
47            <div class="page-tag">
48                <a href="{{ "tags" | absURL }}/{{ . | urlize }}">{{ . }}</a>
49            </div>
50            {{ end }}
51        </div>
52        {{ end }}
53    </div>
54    {{ end }}
55</div>
```

## Resource资源链接汇总

我建立的docker for Hugo开发镜像: [Docker Hub上的repo](https://hub.docker.com/r/orianna/hugo-docker-dev/)、[Github上的repo](https://github.com/orianna-zzo/hugo-docker-dev)。
我的个人主页Hugo代码: [blog-hugo](https://github.com/orianna-zzo/blog-hugo)

[Hugo官网](https://gohugo.io/)、[关于Taxonomy分类说明](https://gohugo.io/content-management/taxonomies/)、[Hugo Template介绍](https://gohugo.io/templates/)

> [参考文章](https://orianna-zzo.github.io/sci-tech/2018-01/blog%E5%85%BB%E6%88%90%E8%AE%B04-hugo%E4%B8%AD%E5%A2%9E%E5%8A%A0tags%E7%AD%89%E5%88%86%E7%B1%BB/)
