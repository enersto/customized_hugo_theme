---
categories: ["English","introduce"]
tags: ["markdown","Hugo","theme","Hexo"]
author: pauke
date: "2019-01-18"
title: "About This Site Build: blogdown, Hugo and Theme Tanka"
---

## why blogdown and hugo
My initiate site build way is by Hexo. The [wreckage](https://enersto.github.io/) is still on. The theme and build process are enough satisfied, and I also got to learn things about git and github basically in the process. But when I want actually to post my article, which is  based on Rmarkdown, I find [blogdown](https://bookdown.org/yihui/blogdown/), the product both comes from [Yihui](https://yihui.name/cn/about/),Rstudio, whom My R learning process benefits superbly from.
The profound tutorial material and theme libraries, super family GUI tool(Rstudio) and seem fast build process(heard the fast character of go before). These are my thought about the blogdown after read the [book](https://bookdown.org/yihui/blogdown/).
So why not switch my site build to hugo. 
As for the building workflow, I think blogdown has optimized so extremely that you can just dive in without any web site and deployment knowledge, and the [article](https://bookdown.org/yihui/blogdown/workflow.html) make me think that I have no need to build wheel again.


## why the theme
My site is based on the theme [Tanka](https://github.com/road2stat/hugo-tanka), which is a totally simple, clean and word orientation Hugo theme.
Based on the theme, I added a fews tools to enrich my site. Here it is:

### taxonomies index
Tanka has not  tags and categories index, but the social account line. And I think an achieved index is necessary for a blog site. And I fork the code of taxonomies from [Xmin](https://github.com/yihui/hugo-xmin).

### table format
There is no table setting in the theme too, and my site also gets code from Xmin.

| Sepal.Length| Sepal.Width| Petal.Length| Petal.Width|Species |
|------------:|-----------:|------------:|-----------:|:-------|
|          5.1|         3.5|          1.4|         0.2|setosa  |
|          4.9|         3.0|          1.4|         0.2|setosa  |
|          4.7|         3.2|          1.3|         0.2|setosa  |
|          4.6|         3.1|          1.5|         0.2|setosa  |
|          5.0|         3.6|          1.4|         0.2|setosa  |
|          5.4|         3.9|          1.7|         0.4|setosa  |

### comments system
The comments system of original theme is hugo embedded comments, Disqus. I don't find much articles comparing of comments in Hugo, which recommends [these comments system](https://gohugo.io/content-management/comments/#comments-alternatives) on documents. But Hexo and other static site generators articles are also sightful:
- [Various ways to include comments on your static site](https://darekkay.com/blog/static-site-comments/)
- [第三方评论系统推荐](https://3mile.github.io/archives/128/).

And my demand of comments is: **simple and lightweight, self-host and not blocked in China**. And [valine](https://valine.js.org/en/) is just fitted. And this is an good [article](https://www.smslit.top/2018/07/08/hugo-valine/) to maintain the build way in Hugo.

### mermaid
[Mermaid](https://mermaidjs.github.io/) is a simple and useful markdown-like script language for generating charts from text via javascript.

{{<mermaid>}}
sequenceDiagram
    participant Alice
    participant Bob
    Alice->>John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
{{< /mermaid >}}


If you want to add to your theme, you can find the repo from this [theme](https://github.com/matcornic/hugo-theme-learn). then copy the `mermaid.html`  from `hugo-theme-learn\layouts\shortcodes` to `your_theme_file\layouts\shortcodes`, mermaid file in `hugo-theme-learn\static\` to `your_theme_file\static\`.  