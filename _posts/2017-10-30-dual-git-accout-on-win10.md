---
category: books-2017
toc: true
published: true
layout: single
title: win10多git账号设置sshkey
description: the more you read, the more you think, better you'll be.
---

## 多sshkey设置的必要性
同一台PC，需要同时管理公司的girrit、公司的gitlab和gitbub的仓库。其中公司仓库使用同一个公司邮箱，github使用自己的gmail邮箱。由于有多个邮箱，而sshkey又与邮箱绑定，因此必须设置多个sshkey。

## 多sshkey设置方法
单个sshkey的配置，可以查看以下流程：
- http://www.cnblogs.com/tinyphp/p/5025311.html
- https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/

多个sshkey生成时，每个sshkey的生成方式与单个sshkey生成方式基本相同。生成sshkey平时我们都是直接回车，默认生成id_rsa和id_rsa.pub。在为多个邮箱建立多个sshkey时，特别需要注意，出现提示输入文件名的时候**(Enter file in which to save the key (~/.ssh/id_rsa): id_rsa_new)**要输入与默认配置不一样的文件名。而且最好输入绝对地址，否则根据bash当前目录的位置，会生成在当前目录下，而不是windows的默认位置。

在初始情况下，我电脑里已经有sshkey，与公司的2个仓库匹配。为了便于区分，我将默认的key重命名为**id_rsa_gitlab**，新创建key命名为**id_rsa_github**，并且将**id_rsa_github**的公钥添加到github。

将新创建的sshkey加入到ssh-agent中，命令
```terminal
ssh-agent bash
ssh-add ~/.ssh/id_rsa_github
ssh-add ~/.ssh/id_rsa_gitlab
```

在.ssh目录下，新建config文件
```terminal
touch config
```

修改为以下内容
```terminal
#github网站使用User=git
Host github.com
HostName github.com
User git
IdentityFile ~/.ssh/id_rsa_github

#公司gitlab
Host git.moretv.cn
HostName git.moretv.cn
User git
IdentityFile ~/.ssh/id_rsa_gitlab
	 
#公司girrit
Host gerrit.moretv.cn
HostName gerrit.moretv.cn
User pan.jian
IdentityFile ~/.ssh/id_rsa_gitlab
```

如果remote pull和push有问题，需要清除全局的name和email，并且在每个仓库下设置单独的name和email。

1.取消global
```terminal
git config --global --unset user.name
git config --global --unset user.email
```

2.设置每个repo的自己的user.email
```terminal
git config  user.email "xxxx@xx.com"
git config  user.name "xxxx"
```
由于local的设置会覆盖全局的，因此也可以保留全局的设置，只是在需要设置的repo中设置自己的name和email。比如我大部分项目都用公司的配置，那我就只需要在gitbug的repo中设置单独的name和email。

参考：
- https://gist.github.com/suziewong/4378434
- http://www.jianshu.com/p/89cb26e5c3e8