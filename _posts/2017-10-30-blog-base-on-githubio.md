---
category: books-2017
toc: true
published: true
layout: single
title: win10系统搭建github pages博客
description: the more you read, the more you think, better you'll be.
---

# 技术选择
在github上写博客，有多种可选的方式，[详情介绍](https://github.com/rainzhaojy/blogs/issues/1)：
* [x]Github pages
* [ ]直接提交静态文件到github pages (Hexo等)
* [ ]利用github issue写博客
* [ ]利用github wiki写博客

作为程序员，选择了第一种方式。前期使用模板搭建博客，重点关注博客内容；等博文数量到一定等级后，再优化模板，添加文章分类等功能。

# 环境搭建
个人尝试过3种方式
1. [windows原生](http://silvercodingcat.com/blog/2017/03/13/Ruby-jekyll/)
2. mingw
3. [Jekyll on Windows](https://jekyllrb.com/docs/windows/)

在1和2这2种方式中，都遇到了问题：包括二进制包依赖、字符集编码问题（Invalid GBK character "\xE2"）。由于我的PC平时用于工作，开发环境比较复杂，而且对ruby不太熟悉，不想过多修改引起其他问题。在尝试了方案3之后，问题完美解决，因此环境搭建以“bash on windows10”的方式介绍。

## bash on windows安装
安装方式见https://msdn.microsoft.com/en-us/commandline/wsl/install-win10
1. Open PowerShell as Administrator and run:
> Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux
2. Restart your computer when prompted.

## bash下环境搭建
见[Installation via Bash on Windows 10](https://jekyllrb.com/docs/windows/)，主要是ruby与相关依赖包的安装。
按照该文档安装后，实际使用的时候缺少libz，需要单独安装
> sudo apt-get install libz-dev

# 博客仓库创建
## 建立github io pages
由于github相关软件的安装设置在我的环境中已经就绪，不再详细介绍。主要涉及git软件安装、sshkey设置，有需要的话可以自行google。
在github中新建一个repo，repo的名字必须为 “账号.github.io”，比如我的账号为elemem，我的github page的repo名为“elemem.github.io”。也可以在github中找到自己喜欢的模板，fork过来后，修改repo的名字为“账号.github.io”。

## 博客模板
在github上搜索了jekyll模板，找到了一个得分较高的[minimal-mistakes](https://github.com/mmistakes/minimal-mistakes/)
该模板提供了使用方式介绍，[quick-start-guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
我的使用方式为(不直接fork是希望git仓库较小，下载和上传速度快一些)：
- 建立elemem.github.io的空白repo，并拉到本地
- 克隆[minimal-mistakes](https://github.com/mmistakes/minimal-mistakes/)到本地
- 复制 minimal-mistake下的文件到elemem.github.io目录下（.git文件夹除外）
- 按照说明删除无用文件[Remove the Unnecessary](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)
- 修改根目录下的Gemfile为

```terminal
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
end
```
- 建立_post文件夹，放入一篇文档，注意文档需要满足指定格式。可以在minimal-mistake的docs/_post目录下复制一篇过来。

## 测试博客
依次运行以下命令
```terminal
bundle install
bundle update
bundle exec jekyll server
```
在浏览器中打开http://127.0.0.1:4000, 可以看到自己放在_post目录中的文章，博客搭建成功。将_post目录下的文章，替换成自己需要写的，就可以上传github了。

