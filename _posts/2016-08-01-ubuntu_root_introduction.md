---
layout:     post
title:      "Ubuntu 操作篇"
subtitle:   "Ubuntu修改为root账号远程登录"
date:       2016-08-01 07:00:00
author:     "DavidWang"
header-img: "img/post-bg-e2e-ux.jpg"
catalog: true
tags:
    - Ubuntu
--- 

### 1. 修改 root 密码

```
sudo passwd root
```

### 2. 以其他账户登录修改内容

```
xxx@ubuntu14:~$ su - root
Password:
root@ubuntu14:~# vi /etc/ssh/sshd_config
```

### 3. 注释注解

```
# Authentication:
LoginGraceTime 120
#PermitRootLogin without-password
PermitRootLogin yes
StrictModes yes
```

### 4. 重启ssh服务

```
root@ubuntu14:~# sudo service ssh restart
ssh stop/waiting
ssh start/running, process 1499
root@ubuntu14:~#
```
