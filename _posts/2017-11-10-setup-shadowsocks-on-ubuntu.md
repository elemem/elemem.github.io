---
category: 2017
published: true
layout: splash
title: 在aws上搭建shadowsocks服务
description: the more you read, the more you think, better you'll be.
---

## aws账号申请
国际版aws EC2 12个月免费，可以用来搭建VPN免费翻墙。aws账号申请，根据注册提示一步步操作，填写需要信息即可。网上找了一下，有图文教程,详情见：[AWS EC2入门篇](https://blog.waterstrong.me/aws-ec2-basic/)

有2点需要注意：
- 最后一步会有电话验证，电话过来时输入注册页面的验证码，注册才能成功。
- 注册成功后，信用卡会扣取1美金

## EC2实例创建
在EC2创建界面，先选择区域。用[http://www.cloudping.info/](http://www.cloudping.info/)
测试网速，选择速度最好的一个节点即可，我选择了东京节点，但首尔、新加坡节点速度也都不错。

###步骤 1: 选择一个 Amazon 系统映像(AMI)
我选择了 **Ubuntu Server 16.04 LTS (HVM)**

###步骤 2: 选择一个实例类型
**t2.micro** 是免费的类型，选择的时候有文字提示，选错类型的话，估计就要收费了。在创建实例前，最好先创建密钥。刚注册结束时，在 **EC2控制面板** 的密钥对中，可以看到只有0个密钥。我第一次创建实例时，没有密钥对，跳过了选择密钥这个过程，创建结束后不知道怎么连接服务器。最后的解决方案是先删除实例，创建密钥对后重新创建实例。

###步骤 3: 连接服务器
我是在windows下使用msys2来连接远程服务器的，在创建密钥结束后，根据提示将key.pem文件保存在msys2用户目录，然后根据官方[文档](http://docs.aws.amazon.com/zh_cn/AWSEC2/latest/UserGuide/AccessingInstancesLinux.html)中 **连接到 Linux 实例** 这一节的操作提示进行操作:

使用 ssh 命令连接到实例。您将指定私有密钥 (.pem) 文件和 user_name@public_dns_name。 对于 Amazon Linux，用户名为 ec2-user。对于 RHEL，用户名称是 ec2-user 或 root。对于 Ubuntu，用户名称是 ubuntu 或 root。对于 Centos，用户名称是 centos。对于 Fedora，用户名称是 ec2-user。对于 SUSE，用户名称是 ec2-user 或 root。另外，如果 ec2-user 和 root 无法使用，请与您的 AMI 供应商核实。

经过测试，ubuntu用户可用，root用户无法登录
```terminal
ssh -i /path/my-key-pair.pem ubuntur@ec2-198-51-100-1.compute-1.amazonaws.com
```

经过这一步，远程主机就创建好了

## shadowsocks 服务搭建
shadowsocks服务，需要通过pip安装
```terminal
sudo apt-get update
sudo apt-get install python-dev build-essential
sudo apt-get install python-pip
```

检查pip是否安装成功
```terminal
pip --version
```

可以查找有哪些版本
```
pip search shadowsocks
```
在我的主机上，显示以下内容：
```terminal
auto-ss (0.1.3)                - Auto acquire ss-link free shadowsocks accounts and test connection speed
shadowsocks-c (2.8.2)          - shadowsocks clone with linux pluggable congestion control support
shadowsocks-gilgamesh (0.2.5)  - a simple way to use mongo db, let db like dict
shadowsocks-gtk (0.1.1)        - ShadowSocks Gtk Client
shadowsocks-py (2.9.1)         - A fast tunnel proxy that help you get through firewalls, the original pypi source is not maintained since version 2.8.2, this is a
                                 newly maintained pypi source by SilverLining.
shadowsocks-sdk (0.1.0.dev0)   - shadowsocks sdk and cli
shadowdns (0.1.3)              - A DNS forwarder using Shadowsocks as the server
shadowproxy (0.2.3)            - A proxy server that implements Socks5/Shadowsocks/Redirect/HTTP (tcp) and Shadowsocks/TProxy/Tunnel (udp) protocols.
shadowsocks (2.8.2)            - A fast tunnel proxy that help you get through firewalls
  INSTALLED: 2.8.2 (latest)
sscrypto (0.1.1)               - shadowsocks crypto libs.
```

我选择了shadowsocks (2.8.2)
```terminal
sudo pip install shadowsocks
```

建立配置文件
```terminal
vim shadowsocks.json
```

添加以下内容
```
{
    "server":"0.0.0.0",
    "server_port":8388,
    "local_port":1080,
    "password":"shadowsockspasswordforyou",
    "timeout":600,
    "method":"aes-256-cfb"
}
```
说明如下：
- server：监听的IP，必须是"0.0.0.0"
- server_port：shadowsocks服务端口号，可以在1080~65000修改，默认是8388
- local_port：不知道干嘛用的，没改过
- password：登陆密码，不要太短
- timeout：超时，默认600
- method：加密算法，推荐"aes-256-cfb"

## 防火墙开启
默认情况下，aws只开放22端口用于ssh登录。需要开启服务器的外部访问端口，你才能使用shadowsocks服务。

在EC2控制面板的"实例->实例"中，先看一下你实例的"安全组"具体选了哪个。在EC2控制面板的“网络与安全->安全组”中，勾选你实例使用的"安全组"，选择“入站”tab，点击“编辑”，在弹出的“编辑入站规则”界面，点击“添加规则” 。规则如下：

- 类型：自定义TCP规则
- 协议：TCP
- 端口范围：**server_port** 中使用的值
- 来源：如果你需要在多个地点访问，选择“自定义”，0.0.0.0/0。如果你只在一个地点访问，选择“我的 IP”。

编辑结束后，保存配置。

## 客户端配置
接下来，就可以用shadowsocks客户端来测试能否翻墙了。windows平台有图形化客户端，可以在github[下载界面](https://github.com/shadowsocks/shadowsocks-windows)中 **Download the latest release** 提示的下载链接中，下载最新版本。

打开软件后，右键点击任务栏图标，选择"服务器-》编辑服务器"，弹出编辑界面，填写以下信息：
- 服务器地址：你服务器的公网IP，在EC2控制面板的实例中，查看**IPv4 公有 IP**
- 服务器端口：与服务器配置中的**server_port**相同
- 密码：与服务器配置中的**password**相同
- 加密：与服务器json配置中的**method**相同
- 代理端口：默认1080

如果是第一次翻墙，需要右键点击shadowsocks的图标，选择"启动系统代理"以开启全局代理。

在浏览器中设置全局代理为“127.0.0.1:1080”，尝试一下是否能访问“www.youtube.com”，如果能访问，说明代理成功。

### 浏览器插件
如果所有流量都走代理，相当于在国外的服务器访问你需要访问的内容，这样访问国内资源的时候会很慢。chrome浏览器的话，可以通过插件"Proxy SwitchyOmega"，决定哪些网站要走代理哪些不走，chrome应用商店中可以下载该插件。其他浏览器应该也有相应的插件，既然已经能翻墙了，我想应该难不倒你。
