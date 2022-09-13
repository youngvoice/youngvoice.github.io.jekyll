---
titile: How use github-pages
description: to use github-pages based on Jekyll
categories:
 - tutorial
tags: [operate]
---

# Directly operate on github website. 

# Set up site with Jekyll
GitHub Pages and Jekyll

Jekyll is a static site generator with built-in support for GitHub Pages.

## create site with Jekyll
Prerequisites

install Jekyll and Git

Bundler manages Ruby gem dependencies, reduces Jekyll build errors, and prevents environment-related bugs, so recommend using Bundler to install and run Jekyll. 

1. install Ruby.
2. install Bundler.

`>>>>>>>>>>>>>>>`

Creating a repository for your site on github

Creating your site locally using Jekyll

Test local site and Push to github repository



`>>>>>>>>>>>>>>>`

Add content to Pages site

About content in Jekyll sites. The main type of content for Jekyll sites are pages and posts. the [theme](https://jekyllrb.com/docs/themes/#overriding-theme-defaults) includes default layouts, includes, and stylesheets that will automatically be applied to new pages and posts on your site. To set variables and metadata, such as a title and layout, for a page or post on your site, you can add YAML front matter to the top of any Markdown or HTML file.

you can add a new page or post to your Jekyll site on Github pages.

add a new page to your site

add a new post to your site

About github pages and Jekyll, need continue to learn...
- [ ] aount Jekyll
- [ ] Front matter
- [ ] Themes
- [ ] Plugins
- [ ] Syntax highlighting



In [directory structure](http://jekyllrb.com/docs/structure/), we know that 
> Except for the special cases listed above, every other directory and file -such as css and images folders, favicon.ico files, and so forth -will be copied verbatim to the generated site.

In [引用图片和其它资源](http://jekyllcn.com/docs/posts/#%E5%BC%95%E7%94%A8%E5%9B%BE%E7%89%87%E5%92%8C%E5%85%B6%E5%AE%83%E8%B5%84%E6%BA%90), 可以知道咋样使用变量 `site.url` 来引用站点的文件。
> 一种常用做法是在工程的根目录下创建一个文件夹，命名为　assets 或者 downloads，将图片文件，下载文件或者其它的资源放到这个文件夹下。然后在任何一篇文章中，它们都可以用站点的根目录来进行引用。

引用图片
```
![pic]({{ site.url }}/assets/screenshot.jpg)
```
引用文件
```
[filename.pdf]({{ site.url }}/assets/mydoc.pdf).
```

## Add theme to Pages site
[Adding a theme to your GitHub Pages site using Jekyll](https://jekyllrb.com/docs/themes/#overriding-theme-defaults)

_config.yml

Converting gem-based themes to regular themes
推荐一个好用的[themes](https://github.com/simpleyyt/jekyll-theme-next)

The Gemfile and Gemfile.lock files are used by Bundler to keep track of the required gems and gem versions you need to build your Jekyll site.

If you’re publishing on GitHub Pages you should update only your _config.yml as GitHub Pages doesn’t load plugins via Bundler.


### Themes
three catagory theme:
* gem-based theme
* regular themes
* themes hosted on GitHub

(theme Jekyll... 都是gem)


Github Pages only support some gem-based themes
github pages also supports using any themes hosted on GitHub using the remote_theme configuration as if it were a gem-based theme. 

main reference
https://docs.github.com/en/pages
http://jekyllcn.com/
https://jekyllrb.com/docs/installation/ubuntu/
