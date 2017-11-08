---
category: books-2017
toc: true
published: true
layout: single
title: centos下搭建scrapy运行环境
description: the more you read, the more you think, better you'll be.
---

安装epel扩展源
```terminal
 sudo yum -y install epel-release
```

安装python-pip，升级到最新
```terminal
sudo yum -y install python-pip
pip install --upgrade pip
```

安装开发工具包
```terminal
yum groupinstall "Development Tools" -y
```
安装Scrapy依赖包
```terminal
yum install -y python-devel openssl-devel libxslt-devel libxml2-devel
```
安装scrapy
```terminal
pip install Scrapy
```

安装scrapyd
```terminal
pip install Scrapyd
```

安装scrapyd-client
```terminal
pip install scrapyd-client
```
在用户目录下新建scrapyd.conf文件，内容如下：
```
[scrapyd]
eggs_dir = /usr/scrapyd/eggs
logs_dir = /usr/scrapyd/logs
jobs_to_keep = 100
dbs_dir = /usr/scrapyd/dbs
max_proc = 0
max_proc_per_cpu = 800
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
schedule.json = scrapyd.webservice.Schedule
cancel.json = scrapyd.webservice.Cancel
addversion.json = scrapyd.webservice.AddVersion
listprojects.json = scrapyd.webservice.ListProjects
listversions.json = scrapyd.webservice.ListVersions
listspiders.json = scrapyd.webservice.ListSpiders
delproject.json = scrapyd.webservice.DeleteProject
delversion.json = scrapyd.webservice.DeleteVersion
listjobs.json = scrapyd.webservice.ListJobs
```
其中， bind_address = 0.0.0.0是为了便于除自身外其他机器可以访问,默认监听接口为“127.0.0.1”。
启动服务，查看运行结果“ Scrapyd web console available at http://0.0.0.0:6800/ ”说明配置运用成功
```terminal
[panjian@transcode3 ~]$ sudo scrapyd
2017-10-23T13:15:50+0800 [-] Loading /usr/lib/python2.7/site-packages/scrapyd/txapp.py...
2017-10-23T13:15:51+0800 [-] Scrapyd web console available at http://0.0.0.0:6800/
2017-10-23T13:15:51+0800 [-] Loaded.
2017-10-23T13:15:51+0800 [twisted.scripts._twistd_unix.UnixAppLogger#info] twistd 16.4.1 (/usr/bin/python2 2.7.5) starting up.
2017-10-23T13:15:51+0800 [twisted.scripts._twistd_unix.UnixAppLogger#info] reactor class: twisted.internet.epollreactor.EPollReactor.
2017-10-23T13:15:51+0800 [-] Site starting on 6800
2017-10-23T13:15:51+0800 [twisted.web.server.Site#info] Starting factory <twisted.web.server.Site instance at 0x3887d88>
2017-10-23T13:15:51+0800 [Launcher] Scrapyd 1.2.0 started: max_proc=6400, runner='scrapyd.runner'
```
修改爬虫工程目录下的scrapy.cfg

```
# Automatically created by: scrapy startproject
#
# For more information about the [deploy] section see:
# https://scrapyd.readthedocs.org/en/latest/deploy.html

[settings]
default = geeksInfo.settings

[deploy]
url = http://172.16.12.3:6800/
project = geeksInfo
```

在爬虫目录下，部署爬虫
```terminal
[panjian@transcode3 geeksInfo]$ scrapyd-deploy default
Packing version 1508740534
Deploying to project "geeksInfo" in http://172.16.12.3:6800/addversion.json
Server response (200):
{"status": "ok", "project": "geeksInfo", "version": "1508740534", "spiders": 1, "node_name": "transcode3"}
```

启动爬虫
```terminal
[panjian@transcode3 geeksInfo]$ curl http://172.16.12.3:6800/schedule.json -d project=geeksInfo -d spider=ithome
{"status": "ok", "jobid": "53bc7656b7be11e7bfe82c56dc3c031d", "node_name": "transcode3"}
```
