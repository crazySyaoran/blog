---
layout: post
title: 以饥荒为例的传统游戏服务器搭建教程
date: 2021-02-09
Author: Syaoran
tags: [server, dst]
comments: true
---

## 前言

在[上一篇文章](https://crazysyaoran.github.io/blog/rust-server/)中我们叙述了一种服务器搭建的方式：在本机上运行服务器程序，然后通过云服务器的端口转发将服务暴露到公网。这种方法相比较于传统的直接把服务器程序跑到云服务器上的方式有以下优点：

- 大大减少了对云服务器的算力和内存要求。国内服务器的单位性能价格过于昂贵，而大多数游戏对服务器的算力和内存要求是很高的，比如MC、RUST、7 days to die、forest等等这种3D沙盒游戏。像是一般靠着薅羊毛从腾讯云或者阿里云嫖到的70~99块钱一年的2G1M服务器是无法胜任服务程序运算工作的。而端口转发这种方式使得运算压力被转移到了用户本机，仅仅转发数据包消耗的服务器资源是完全可以接受的。
- 同时减少了服务器的带宽压力，因为其中一个人直接访问本机地址就可以获取服务。通常我们都是2~3个人一起玩，这样减去一个人实际上节省的带宽压力就还是非常可观的了。
- 另外我个人也感觉在本机的操作系统上直接进行操作毕竟还是比ssh服务器要方便的，可以用鼠标不说，还可以调用本机上各种应用程序或者docker提供的服务。

那么有没有有没有一种情况下我们只能选择把服务器直接搭到云服务器上呢？答案是有的。比如饥荒这款~~垃圾~~游戏。

现在为了保护版权和共享steam还有信息等等各种考虑，steam上越来越多的游戏开始强制玩家通过steam的服务器进行部分数据包的中转，同时关闭了服务器直连的功能。客户端无法再通过类似`client.connect([IP:PORT])`的方式直接与服务器建立连接了，这样一来我们上述的方法也就失效了，这种情况下我们只能选择将服务程序跑到云服务器了。不过幸运的是饥荒的服务器程序这两年进行过优化，实测2G1M的小霸王服务器也能跑起来。

> 这个时候你可能就会问，有没有哪个游戏服务器程序写的跟屎一样死耗资源然后还强行不让玩家端口直连呢？是的，这个垃圾游戏就是**Space Engineer**。当时我们三个人折腾到凌晨两三点人均气成脑溢血了也没找出一个可行的方案，对于这种开发商我就想跟他说一句：
>
> ![nmbwcnm](../post_images/nmbwcnm.jpg)

好了，接下来我们以饥荒为例，补充讲解传统的在云服务器上直接搭建服务器的方法。

> 除非特殊说明，本文余下内容均在cenos上测试完成。

## 准备工作

首先连接服务器并在服务上安装包管理工具，steamcmd和各种依赖库：

### 安装依赖库

ssh连接服务器：

```
ssh [USER]@[HOST]
```

> 或者直接使用云服务器的控制台

依赖库参考了[迹流](https://zhuanlan.zhihu.com/p/105219404?utm_source=qq)的文章，涉及的操作系统比较全面一些。（我个人只有centos64bit，测试是通过的，你们如果和我不一致出现了问题请直接去喷原作者233）

- **Ubuntu 32位环境**

```text
sudo apt-get update
sudo apt-get install libstdc++6:i386 libgcc1:i386 libcurl4-gnutls-dev:i386
```

- **Ubuntu 64位环境**

```text
sudo apt-get update
sudo apt-get install libstdc++6 libgcc1 libcurl4-gnutls-dev
```

如果是Centos或其他基于RHEL的Linux系统，安装依赖使用yum，方法类似。以下是参考：

- **Centos 32位环境**

```text
// 我当时在64位系统上运行了这个命令，导致后续要新建一个软链接把组件库连接过来。详见下一节。
yum -y install glibc.i686 libstdc++.i686 libcurl4-gnutls-dev.i686 libcurl.i686 
```

- **Centos 64位环境**

```text
yum -y install glibc libstdc++ libcurl4-gnutls-dev libcurl
```

### 安装screen

```
sudo yum -y install screen
```

### 安装steamsmd

```
cd /home && mkdir steamcmd && cd steamcmd
wget https://steamcdn-a.akamaihd.net/client/installer/steamcmd_linux.tar.gz
tar -xvzf steamcmd_linux.tar.gz
```

## 服务程序安装与运行

运行解压后的steamcmd程序

```
./steamcmd.sh
```

进入steamcmd后匿名登录下载饥荒的服务器程序

```
login anonymous

force_install_dir /home/dstserver

app_update 343050 validate
```

以上的命令同样可以用于更新服务器程序。游戏更新后记得更新服务器程序，否则大厅是搜索不到旧版本的服务器的。这一点适用于所有steamcmd建立的服务器。

我当时在64位系统上安装了32位的依赖库，所以提示缺少libcurl-gnutls.so.4。因此ln把32位的依赖链接到过来。

> 居然好使了我也是没想到的

```
ln -s /usr/lib/libcurl.so.4 /home/dstserver/bin/lib32/libcurl-gnutls.so.4
```

接着新建两个批处理命令

```
cd /home/dstserver/bin

echo "./dontstarve_dedicated_server_nullrenderer -console -persistent_storage_root /home/dstsave -conf_dir dst -cluster World1 -shard Master" > master_start.sh

echo "./dontstarve_dedicated_server_nullrenderer -console -persistent_storage_root /home/dstsave -conf_dir dst -cluster World1 -shard Caves" > cave_start.sh

chmod +x master_start.sh cave_start.sh
```

此时可以运行`./master_start.sh`启动主世界。出现以下提示后按`ctrl+c`或者输入`quit`退出

```
[200] Account Failed (6): "E_INVALID_TOKEN"
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
!!!! Your Server Will Not Start !!!!
!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
```

访问KLei官网，进入[个人信息](https://accounts.klei.com/account/info)，选择上方菜单栏的“游戏”，进入后点击[饥荒：联机版的游戏服务器](https://accounts.klei.com/account/game/servers?game=DontStarveTogether)。在下方输入名字后点击添加新服务器，对服务器进行配置后点击下载设置。将下载下来的压缩包解压，将文件名改为之前在批处理命令中我们写下的服务器名字`World1`。

通过scp命令将World1文件夹传到服务器上：

```
scp -r World1 [USER]@[HOST]:/home/dstsave/dst
```

现在可以去启动服务器了，ssh连接回服务器并利用screen来运行服务器：

```
cd /home/dstserver/bin
screen -S master 
./master_start.sh
```

启动完成后`ctrl+a`然后`ctrl+d`切回原来的页面，同样的方式启动洞穴：

```
screen -S caves
./cave_start.sh
```

至此为止，纯净版的服务器启动完毕。查看10998和10999端口上是否已经运行了服务：

```
netstat -nlp |grep :10999
netstat -nlp |grep :10998
```

记得在安全组中开放10999端口和10998端口。

## Mod配置

编辑`/home/dstserver/mods`下的`dedicated_server_mods_setup.lua`来添加mod

```
--There are two functions that will install mods, ServerModSetup and ServerModCollectionSetup. Put the calls to the functions in this file and they will be executed on boot.

--ServerModSetup takes a string of a specific mod's Workshop id. It will download and install the mod to your mod directory on boot.
    --The Workshop id can be found at the end of the url to the mod's Workshop page.
    --Example: http://steamcommunity.com/sharedfiles/filedetails/?id=350811795
    --ServerModSetup("350811795")

--ServerModCollectionSetup takes a string of a specific mod's Workshop id. It will download all the mods in the collection and install them to the mod directory on boot.
    --The Workshop id can be found at the end of the url to the collection's Workshop page.
    --Example: http://steamcommunity.com/sharedfiles/filedetails/?id=379114180
    --ServerModCollectionSetup("379114180")

ServerModSetup("XXXXXXXX")
ServerModSetup("YYYYYYYY")

--ServerModCollectionSetup("id")
```

ServerModSetup("XXXXXXXXX")中XXXXXXXX对应的就是mod的id。想找到对应mod的id可以到你本机上新建一个服务器，选好需要的mod，然后到`文档\KLei\DoNotStarveTogether\服务器名\Master`下打开`modoverrides.lua`文件，每一条记录的`["workshop-xxxxxxxx"]={...}`中的`xxxxxxxx`就是mod的id，填入服务器的`dedicated_server_mods_setup.lua`即可。

## 相关设置

### 设置管理员

在`/home/dstsave/dst/World1/`下新建`adminlist.txt`，将管理员的KLEI_ID写入，换行间隔每一个管理员。

### 设置暴露端口

如主世界，进入Master文件夹，修改`server.ini`可以改变端口。

## 结语

Enjoy gaming!

















