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

>>>>>>>>>>>>>>>
Creating a repository for your site on github
Creating your site locally using Jekyll

Test local site and Push to github repository



>>>>>>>>>>>>>>>

Add content to Pages site
About content in Jekyll sites. The main type of content for Jekyll sites are pages and posts. the theme(https://jekyllrb.com/docs/themes/#overriding-theme-defaults) includes default layouts, includes, and stylesheets that will automatically be applied to new pages and posts on your site. To set variables and metadata, such as a title and layout, for a page or post on your site, you can add YAML front matter to the top of any Markdown or HTML file.
you can add a new page or post to your Jekyll site on Github pages.
add a new page to your site
add a new post to your site

About github pages and Jekyll, need continue to learn...
- [ ] aount Jekyll
- [ ] Front matter
- [ ] Themes
- [ ] Plugins
- [ ] Syntax highlighting


## Add theme to Pages site
Adding a theme to your GitHub Pages site using Jekyll
_config.yml

```
https://jekyllrb.com/docs/themes/#overriding-theme-defaults

Converting gem-based themes to regular themes
推荐一个好用的themes(https://github.com/simpleyyt/jekyll-theme-next)

The Gemfile and Gemfile.lock files are used by Bundler to keep track of the required gems and gem versions you need to build your Jekyll site.
If you’re publishing on GitHub Pages you should update only your _config.yml as GitHub Pages doesn’t load plugins via Bundler.

```
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
