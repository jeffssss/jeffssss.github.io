---
published: true
title: 这大概是以最低成本搭建饥荒独立服务器的方法
layout: post
author: Jeffssss 
comments: true
category: 游戏
tags:
- 游戏
---

# 如何以低成本搭建饥荒独立服务器

## 背景
小伙伴最近买了饥荒，于是我这阵子重新开始沉迷饥荒无法自拔。

我作为主机，使用的是长城宽带，另外的小伙伴有使用电信的，有使用校园网的。联机体验较差，且偶尔会出现连不上主机的问题(长宽网络质量真的不咋地)

于是，我考虑使用云服务搭建饥荒的独立服务器。

## 方案决策

考虑一下两种方案：

1. 按月租服务器。
	
2. 按需租服务器。

因为我在游戏前会和小伙伴约定游戏时间，且游戏时间较短(一般都是两三个小时)，同时游戏比较吃内存，而高配置的云服务器的月租较贵，所以使用按月的云服务器不合算。最终我决定按需租服务器。

按需租云服务器的时候需要注意两点：

1. 云服务提供商需要支持系统盘快照的功能，否则你每次结束游戏后，都需要把游戏存档拷贝到别的地方，然后下次启动时重新安装饥荒服务端。
2. 游戏完毕之后，需要备份系统盘，然后手动的删除云服务器，否则会一直扣费。

因为我在美团云上有一些余额(内部福利哈哈哈哈哈)，所以我优先考虑了美团云。

## 参考

一些有帮助的信息：

1. 美团云官网 [https://www.mtyun.com/](https://www.mtyun.com/) (当然，换成别的云服务也可以)
2. 饥荒在Steam的指南 [How to setup dedicated server with cave on Linux](https://steamcommunity.com/sharedfiles/filedetails/?id=590565473) 后续部分图片来自该指南
3. SteamCMD的文档[https://developer.valvesoftware.com/wiki/SteamCMD](https://developer.valvesoftware.com/wiki/SteamCMD)

## 步骤

### 1. 申请云服务器

在我的印象中，需要有一定的账户余额才可以使用按需的服务器(阿里云是100元，不知道美团云是多少钱)

![美团云](/img/mtyun1.png)

申请云服务器页面有如下参数：

地域：服务器所在的区域，如果都在国内的话，选啥区别都不太大。可用区随便选一个即可。

主机配置：用来架设游戏的服务端程序，重要的是CPU和内存，不需要使用GPU，所以此处选择普通机型即可。CPU和内存，我选择的是4核4G，这个配置理论上可以同时跑饥荒世界和洞穴世界。

镜像： 第一次创建的时候，选择公共镜像，如果已有饥荒的系统盘的话，可以选自定义镜像。我选择的是Ubuntu 16.04 64位系统。(我选过CentOS系统，但是yum的支持并不太好，所以换了Ubuntu)

存储： 系统盘就已经够用了，数据盘随便多少都行，反正用不到。

网络配置：网络类型使用基础网络即可，浮动IP必须要购买，不然没有公网的IP，计费方式推荐选择按流量计费，因为游戏消耗的流量并不多，所以你可以选择非常大的带宽，这样游戏就不会因为带宽不够而出现连接上的问题。我选择的是按流量计费，带宽是100Mbps，防火墙选择”开放“(因为游戏需要开放一些不常用的端口，比如10999)

安全设置方式：决定你如何连接上服务器的终端。推荐使用密码登录。

基本设置：业务组不选即可，主机名随意，计费模式选择按需计费。

这样下来，你的预估花费(不包含外网流量)是0.34元/小时。

### 2. 安装饥荒的服务端

安装饥荒服务端可以见参考2中的链接 或者[饥荒联机独立服务器搭建教程（二）：Linux篇](http://blog.ttionya.com/article-1233.html)

第一步，登录服务器终端。这个过程中，如果你是windows系统，使用putty软件。其余操作系统可直接在终端中使用ssh命令登录。(感觉说了等于没说→_→ 会的人自然懂，不会的人看了也还是不懂)

第二步，安装SteamCMD。见参考3的链接,Ubuntu可以直接使用以下命令安装:

`sudo apt-get install steamcmd`

然后将/usr/games/steamcmd 加入到系统的path中(命令略)

第三步，安装饥荒的服务端。文档写到这里，发现完全可以直接看参考2一步一步来。于是不再赘述了(我是真的懒)。

第四步，创建存档。这里有两种情况。

1. 你之前在本地有联机存档。此时需要将原存档拷贝到服务器。

	a. 你需要前往本地Klei文件夹里找到你的存档。Klei文件夹在不同操作系统里有不同的路径。比如在`/Klei/DoNotStarveTogether/`文件夹下，找到你的存档，例如"Cluster_1",其中包含`cluster.ini`,`Master/*`。将其复制到服务器的`/root/.klei/DoNotStarveTogether/`下.
	
	b. 拷贝人物存档。进入`/Klei/DoNotStarveTogether/client_save`下，除了带`temp`、`cache`字样的文件或文件夹外，其他的全都复制到服务器的`/root/.klei/DoNotStarveTogether/Cluster_1/Master`下. 最终文件夹应该类似这样（我的存档名叫`JeffssssServer`，等同于`Cluster_1`）：
	
	```
	root@jeffssss:~/.klei# ll
	total 12
	drwxr-xr-x 3 root root 4096 Apr 21 12:27 ./
	drwx------ 9 root root 4096 Apr 28 08:50 ../
	drwxr-xr-x 3 root root 4096 Apr 21 13:29 DoNotStarveTogether/
	root@jeffssss:~/.klei# cd DoNotStarveTogether/JeffssssServer/ && ll
	total 24
	drwxr-xr-x 3 root root 4096 Apr 28 08:50 ./
	drwxr-xr-x 3 root root 4096 Apr 21 13:29 ../
	drwxr-xr-x 4 root root 4096 Apr 28 08:52 Master/
	-rw-r--r-- 1 root root  144 Apr 25 11:43 blocklist.txt
	-rw-r--r-- 1 root root  437 Apr 28 08:50 cluster.ini
	-rw-r--r-- 1 root root   63 Apr 21 12:35 cluster_token.txt
	root@jeffssss:~/.klei/DoNotStarveTogether/JeffssssServer# cd Master/ && ll
	total 4884
	drwxr-xr-x 4 root root    4096 Apr 28 08:52 ./
	drwxr-xr-x 3 root root    4096 Apr 28 08:50 ../
	drwxr-xr-x 4 root root    4096 Apr 21 13:29 backup/
	-rw-r--r-- 1 root root    2545 Apr 21 14:07 leveldataoverride.lua
	-rw-r--r-- 1 root root    9075 Apr 21 14:07 modoverrides.lua
	drwxr-xr-x 7 root root    4096 Apr 21 13:29 save/
	-rw-r--r-- 1 root root      93 Apr 28 08:45 server.ini
	-rw-r--r-- 1 root root     130 Apr 28 09:56 server_chat_log.txt
	-rw-r--r-- 1 root root 4956927 Apr 28 09:57 server_log.txt
	root@jeffssss:~/.klei/DoNotStarveTogether/JeffssssServer/Master# cd save/ && ll
	total 72
	drwxr-xr-x 7 root root  4096 Apr 21 13:29 ./
	drwxr-xr-x 4 root root  4096 Apr 28 08:52 ../
	-rw-r--r-- 1 root root    15 Apr 28 08:52 boot_modindex
	drwxr-xr-x 2 root root  4096 Apr 21 14:09 client_temp/
	drwxr-xr-x 2 root root  4096 Apr 21 13:29 forge_stats/
	drwxr-xr-x 2 root root  4096 Apr 21 14:08 mod_config_data/
	-rw-r--r-- 1 root root   759 Apr 28 08:52 modindex
	-rw-r--r-- 1 root root  1527 Apr 28 08:51 profile
	-rw-r--r-- 1 root root 31541 Apr 28 09:56 saveindex
	drwxr-xr-x 2 root root  4096 Apr 21 13:29 server_temp/
	drwxr-xr-x 3 root root  4096 Apr 21 14:30 session/
	root@jeffssss:~/.klei/DoNotStarveTogether/JeffssssServer/Master/save#
	```
	c. 申请cluster_token（申请方法见参考2）. 并保存为`~/.klei/DoNotStarveTogether/JeffssssServer/cluster_token.txt`
	
	d. 加载mod。 在`~/dst/mods/dedicated_server_mods_setup.lua`中填入需要加载的服务端mod。 举例如下
		
	```
	ServerModSetup("1079538195") #数字是mod的id
	ServerModSetup("345692228")
	ServerModSetup("350811795")
	```
	
2. 直接在独立服务器上创建世界。此情况看参考2的步骤

第五步，启动游戏。

按照参考2做的话，你会编写一个启动游戏的shell脚本。启动shell脚本后，即是启动了服务端程序。等`sim pause`出现后，启动完毕，这个时候就可以enjoy your game。

小伙伴如何直接连接你的服务器？

在饥荒客户端联机上Klei的服务器后，按control + `~` 打开控制台，输入`c_connect("服务器ip",端口号)`即可直接连接到你的服务器，跳过了搜索主机的过程。假设我的服务器外网ip是'127.0.0.1'，端口是10999，则输入`c_connect("127.0.0.1",10999)`

第六步，关闭游戏，删除服务器。

当游戏过程结束之后，需要关闭服务器。你需要做以下步骤。

a. 关闭服务器端的程序：在服务器端的控制台按control+`c`退出服务端程序，此时会进行存档等必要操作。

b. 备份系统盘。在美团云的控制台，点击创建系统盘(如果创建云服务实例的时候是加载的已有的景象，此处点`更新系统盘`)

![mtyun2](/img/mtyun2.png)

c. 备份完成后，删除云服务器实例，注意：需要同时删除对应的浮动IP(否则会一直收你钱)

## 总结

从实际使用来看，假设每次游戏时间是4h的话，服务器开销的花费在2元以内。重点是，你享受的至少是4核4G，100Mbps的云服务器啊！

当然，我也犯过错，比如服务器好几天没关，或者浮动IP忘记删除。这让我白白损失了好些钱→_→(心痛)。但是，这也比一个月好几百的月租费用要少得多。

## 优化点

后期可以用脚本控制整个过程：

启动过程：

1. 利用云服务商的API，自动创建服务器实例，使用特定的配置参数、特定的镜像进行创建
2. 在云服务器的镜像中，添加开机自动运行饥荒服务端的脚本

关闭过程：

1. 远程登录云服务器，关闭饥荒服务端的进程
2. 利用云服务商的API，对当前云服务器实例进行系统盘镜像的更新
3. 利用云服务商的API，删除当前云服务器实例以及浮动IP服务

当然，想法总是美好的，通过查阅文档我发现目前美团云并不支持通过API进行系统盘更新→_→  我不知道阿里云或者腾讯云是否支持，如果支持，可以通过脚本控制整个流程。(当然得花一些时间写脚本啦)