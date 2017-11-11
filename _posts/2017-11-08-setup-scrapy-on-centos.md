---
category: books-2017
toc: true
published: true
layout: single
title: centos下搭建scrapy运行环境
description: the more you read, the more you think, better you'll be.
---

##环境说明
开发环境：目前我在windows上做开发，需要安装scrapy和scrapyd-client(用于部署)

运行环境：在linux下运行，需要安装scrapy、scrapyd、scrapyd-client（可选）、spiderkeeper(job调度与控制)。我使用的服务器的IP为172.16.12.3

官方文档：
- [scrapy](https://doc.scrapy.org/en/latest/index.html)
- [scrapyd](https://scrapyd.readthedocs.io/en/latest/index.html)
- [spiderkeeper](https://github.com/DormyMo/SpiderKeeper)

## centos下环境搭建

### scrapyd安装
由于在同一台服务器下有多个python服务，为了避免环境相互冲突，需要用虚拟环境隔离。我选择的是conda，使用普通权限安装在用户的home目录下，不影响整个服务器本身的python配置。

scrapy和scrapyd紧密配合，使用同一个虚拟环境；spiderkeeper使用另外一个虚拟环境。

安装minoconda，在控制台下运行以下命令：
```terminal
wget https://repo.continuum.io/miniconda/Miniconda2-latest-Linux-x86_64.sh
chmod +x Miniconda2-latest-Linux-x86_64.sh
Miniconda2-latest-Linux-x86_64.sh
source .bashrc
```
在安装过程中，提示你是否要将miniconda加入到.bashrc的PATH环境变量中，记得选yes

使用conda安装scrapy
```terminal
conda install -c conda-forge scrapy
```

conda下没有scrapyd，直接使用pip安装
```terminal
pip install scrapyd
pip install scrapyd-client
```

### scrapyd配置
在用户目录下新建.scrapyd.conf文件，内容如下：
```
[scrapyd]
eggs_dir = eggs
logs_dir = logs
dbs_dir = dbs
jobs_to_keep = 5
max_proc = 0
max_proc_per_cpu = 2
finished_to_keep = 100
poll_interval = 5.0
bind_address = 0.0.0.0
http_port = 6800
debug = off
runner = scrapyd.runner
application = scrapyd.app.application
launcher = scrapyd.launcher.Launcher
webroot = scrapyd.website.Root
[services]
schedule.json     = scrapyd.webservice.Schedule
cancel.json       = scrapyd.webservice.Cancel
addversion.json   = scrapyd.webservice.AddVersion
listprojects.json = scrapyd.webservice.ListProjects
listversions.json = scrapyd.webservice.ListVersions
listspiders.json  = scrapyd.webservice.ListSpiders
delproject.json   = scrapyd.webservice.DeleteProject
delversion.json   = scrapyd.webservice.DeleteVersion
listjobs.json     = scrapyd.webservice.ListJobs
```
其中， bind_address = 0.0.0.0是为了便于除自身外其他机器可以访问，默认监听接口为“127.0.0.1”。

启动服务，查看运行结果“ Scrapyd web console available at http://0.0.0.0:6800/ ”说明配置运用成功
```terminal
[panjian@transcode3 ~]$ scrapyd
2017-10-23T13:15:50+0800 [-] Loading /usr/lib/python2.7/site-packages/scrapyd/txapp.py...
2017-10-23T13:15:51+0800 [-] Scrapyd web console available at http://0.0.0.0:6800/
2017-10-23T13:15:51+0800 [-] Loaded.
2017-10-23T13:15:51+0800 [twisted.scripts._twistd_unix.UnixAppLogger#info] twistd 16.4.1 (/usr/bin/python2 2.7.5) starting up.
2017-10-23T13:15:51+0800 [twisted.scripts._twistd_unix.UnixAppLogger#info] reactor class: twisted.internet.epollreactor.EPollReactor.
2017-10-23T13:15:51+0800 [-] Site starting on 6800
2017-10-23T13:15:51+0800 [twisted.web.server.Site#info] Starting factory <twisted.web.server.Site instance at 0x3887d88>
2017-10-23T13:15:51+0800 [Launcher] Scrapyd 1.2.0 started: max_proc=6400, runner='scrapyd.runner'
```

这时你可以在你本地，通过“http://172.16.12.3:6800”访问scrapyd服务

### spiderkeeper安装与配置
创建spiderkeeper的虚拟环境，在虚拟环境中安装spiderKeeper
```terminal
conda create -n spiderkeeper
source activate spiderkeeper
pip install spiderkeeper
```

运行spiderkeeper，需要指定scrapyd的地址
```
spiderkeeper --server=http://172.16.12.3:6800
```

这时你可以在你本地，通过“http://172.16.12.3:5000”访问spiderkeeper

## 部署爬虫
我是在windows下部署爬虫的，基本步骤如下：

修改爬虫工程目录下的scrapy.cfg，把URL改成scrapyd服务器的IP与端口
```
url = http://172.16.12.3:6800/
```

cd到爬虫目录下，部署爬虫
```terminal
[panjian@transcode3 geeksInfo]$ scrapyd-deploy default
Packing version 1508740534
Deploying to project "geeksInfo" in http://172.16.12.3:6800/addversion.json
Server response (200):
{"status": "ok", "project": "geeksInfo", "version": "1508740534", "spiders": 1, "node_name": "transcode3"}
```

## 启动爬虫
通过“http://172.16.12.3:5000”访问spiderkeeper，在“Job Dashboard”中，可以单次运行爬虫。

通过“http://172.16.12.3:6800”访问scrapyd，可以查看运行日志
