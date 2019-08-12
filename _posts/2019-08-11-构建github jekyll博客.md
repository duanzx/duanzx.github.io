---
layout: default
title:  "使用GitHub Pages结合Jekyll构建博客网站!"
categories: [jekyll]
---
### 写在前面：   

如果只是单纯在本地构建一个私人博客的话，那么只需要使用Jekyll构建就够了。如果想要一个博客网站的话，那么你可以使用GitHub Pages构建博客网站，然后在本地check out你的GitHub Pages
项目，并使用Jekyll在本地进行生产博客文章，然后push到github仓库，然后再次访问GitHub Pages的博客地址，就能看到你的最近更新的博客内容了。

### 什么是Jekyll?		
		
这是来自[Jekyll中文网](http://jekyllcn.com)的定义：可以将纯文本转换为静态博客网站。

### 什么是Github Pages?

[GitHub Pages](https://pages.github.com)基于Jekyll构建，你可以轻而易举地在GitHub上免费发布网站--[自定义域名](https://help.github.com/en/articles/about-supported-custom-domains)等等。

### 构建你的GitHub Pages

**1.** 首先你需要到GitHub注册账户，注册成功后，登录GitHub网站。

**2.** 创建你的Github仓库，并将仓库名称命名为 xxx.github.io , xxx指的是你的github用户名

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_001.png" width="900" height="350" />
</div>

**3.** 为你的Github Pages 选择一个主题。找到你的仓库 xxx.github.io 点击settings标签页，在GitHub Pages下点击 Choose a theme选择一个博客样式主题 

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_002.png" width="750" height="500" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_003.png" width="950" height="500" />
</div>

**4.** 点击 Select theme后，GitHub Pages就会自动帮你生成博客网站，然后在跳转的界面点击Commit changes ，就可以访问网站 https://xxx.github.io

### 使用Jekyll在本地生成博客

**1.** Jekyll 是基于 Ruby 的静态网页生成系统，因此我们首先得[安装 Ruby+Dev 环境](https://jekyllrb.com/docs/installation/)，在本篇文章里我们[使用Win10进行安装Ruby+Dev](https://jekyllrb.com/docs/installation/windows/)

**2.** [下载RubyInstaller](https://rubyinstaller.org/downloads/)

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_004.png" width="700" height="300" />
</div>
**3.** 安装RubyInstaller		
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_005.png" width="500" height="400" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_006.png" width="500" height="400" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_007.png" width="500" height="400" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_008.png" width="500" height="400" />
</div>

* 在控制台输入 1,2,3 然后敲下Enter

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_009.png" width="500" height="400" />
</div>

* 安装组件的时间比较长~，请耐心等待~。安装完成后，命令行里会询问是否所有的组件都已经成功安装了，此时直接回车即可。

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_012.png" width="600" height="500" />
</div>

* 在win10搜索框里输入PowerShell,打开PowerShell命令窗口。因为PowerShell里可以使用linux命令，相对比较方便

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_011.png" width="600" height="600" />
</div>

* 首先确认Ruby是否安装成功，输入命令ruby -v	。然后再确认一下ruby gem的版本,输入命令 gem -v

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_013.png" width="800" height="150" />
</div>

* 使用gem安装jekyll和bundler , 输入命令 gem install jekyll bundler 。你也可以分别执行命令 gem install jekyll 和 gem install bundler

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_014.png" width="850" height="600" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_015.png" width="850" height="320" />
</div>

* jekyll bundler安装完成后，要修改RubyGems的镜像为[https://gems.ruby-china.com/](https://gems.ruby-china.com/),

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_010.png" width="800" height="450" />
</div>

*替换rubygems的源，输入命令 gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/ 和 gem sources -l

* 用 Bundler 的 Gem 源代码镜像命令，输入命令： bundle config mirror.https://rubygems.org https://gems.ruby-china.com

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_016.png" width="860" height="480" />
</div>

* 在这些操作都完成后，我们就可以尝试着用jekyll先在本地生成一个简单的博客网站了，cd到你要创建的博客项目所在文件夹。输入命令： jekyll new myblog 

* 由于在jeky new命令中，执行bundle install的时候会比对bundle版本号，需要耐心等待几十秒

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_017.png" width="850" height="400" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_017-1.png" width="850" height="300" />
</div>
<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_018.png" width="750" height="260" />
</div>

* 在浏览器上查看博客，myblog文件夹下输入命令:bundle exec jekyll serve

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_019.png" width="845" height="200" />
</div>

* 浏览器地址栏输入：localhost:4000,查看博客首页

<div style="display:flex;align-items:center;justify-content:center;">
    <img src="/img/2019-08-11-构建github jekyll博客/githubpages_020.png" width="800" height="610" />
</div>

* 至此，一个本地博客就完成了！。


3 Simple steps to setup Jekyll Categories and Tags

https://blog.webjeda.com/jekyll-categories/
jekyll 语法
http://jekyllcn.com/docs/collections/

jekyll  https://jekyllrb.com/

https://help.github.com/cn/articles/about-jekyll-themes-on-github

https://webdesign.tutsplus.com/zh-hans/tutorials/how-to-set-up-a-jekyll-theme--cms-26332

https://www.zhihu.com/question/30018945?sort=created

https://www.zhihu.com/question/20962496

分类
http://cnitzone.com/blog/2015/01/jekyll-article-category/

https://justjavac.com/jekyll/2012/05/22/use-category-plugin-for-jekyll-blog.html

分页
https://segmentfault.com/a/1190000007709682