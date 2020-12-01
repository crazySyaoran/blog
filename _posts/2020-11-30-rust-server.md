---
layout: post
title: 以RUST为例的游戏服务器搭建教程
date: 2020-11-30
Author: Syaoran
tags: [server, rust]
comments: true
---

本鸽王回来了！ ~~咕咕咕！~~

作为一名单机游戏玩家，搭建游戏服务器应该是一项基本功，通过这篇文章，想以RUST为例讲解和总结一下这么久以来，在公网搭建游戏服务器的过程中踩过的坑和积累到的经验。

**我尽量讲的详细一些，这样让没有计算机网络相关知识的同学也能理解来龙去脉**

实际上，公网搭建服务器和局域网相比最大的困难在于内网穿透。简单来说，我们如果用自己的计算机做服务器一般是没有公网IP的，这样客户端就没办法访问到你的服务，要解决这件事情有四个方案：

- 购买公网IP
- 购买云服务器
- 使用ipv6协议
- 购买内网穿透服务

废话少说，前两种都不推荐，本文主要介绍第四种方案

- 第一种虽然最简单且一目了然，但基本不可行，现在普通用户想从各种运营商那里买到公网IP基本是不可能的，据说公司或者团体可以以极高的价格购买专线来获得公网IP。
- 第二种是现在大多数教程提供的方法，不推荐。原因是云服务器的价格太高，性能又太差。这种方案就相当于你从云服务商那里租赁一台有公网IP的计算机（但实际上这也是一种虚拟化技术），把服务器程序跑在这个计算机上，但是现在便宜的云服务器算力堪忧，无法承担大多数游戏服务器程序的计算量，算力够用的云服务器又贵的离谱。
- 第三种方案理论上可行但受到游戏的局限较大。64位的ipv6地址具有远超ipv4协议32位的地址表示域，可以做到每台设备一个全球ip，而且通常具有超过ipv4的传输速度。现在大多数的宽度默认都是有ipv6的（电信、移动、联通...），实际上现在已经有了许多基于ipv6的P2P通信工具。但可惜的是，绝大多数游戏底层还是只支持ipv4通信的，或许在不久的将来，这种方案将变得十分普遍。

这篇文章我们着重介绍第四种方案。

第四种方案常见的是购买类似花生壳这样的内网穿透工具，它的作用就是在你的机器和它的一个有公网IP的机器之间建立一个信息传输通路，然后将它收到的信息原封不动转发给你，并将你发送给他的数据包转发出去，从而解决你无法被外界访问的问题。这种相比较于方案二，转发数据包的云服务器配置要求要远小于运行服务器程序的云服务器，因此价格要低得多。

但是花生壳赚到的这笔钱我也想省下来！所以我们购买了云服务器并自己搭建了端口转发服务，本篇教程以我刚购买的88块/年的腾讯云ECS（2M带宽 1G内存）为例。各家云服务器供应商提供的ECS用法大同小异，我之前也分享过[阿里云配置的踩坑记录](https://crazysyaoran.github.io/blog/aliyun/)。

**接下来以RUST为例简述配置过程**

首先购买云服务器并重置root密码。**并在安全组规则中开放服务器的6000,6001,7000端口**

> 在本例子中，6000和6001端口用于暴露出去与游戏客户端通信，6000是TCP，6001是UDP；7000端口用于绑定到游戏服务器所在的计算机，与其进行通信。

下载对应操作系统的[端口转发程序](https://github.com/fatedier/frp/releases)，我的云服务器是centOS 64位，是Linux的发行版，因此我下载的应该是[frp_0.34.3_linux_arm64.tar.gz](https://github.com/fatedier/frp/releases/download/v0.34.3/frp_0.34.3_linux_arm64.tar.gz)

下载好之后传到云服务器上：

```
scp [本地文件] [用户名]@[服务器IP地址]:[目标路径]
// 如 scp frp_0.34.3_linux_arm64.tar.gz root@111.22.3.444:~
```

ssh连接云服务器：

```
$ ssh [用户名]@[IP]
// 如 ssh root@111.22.3.444
```

输入密码后进入云服务器。

解压端口转发程序：

```
tar -zxvf frp_0.34.3_linux_arm64.tar.gz
cd frp_0.34.3_linux_arm64
```

>  可以更改配置文件来改变服务器暴露的端口，默认暴露7000端口：`vim frps.ini`

安装tmux：

```
// yum
yum install tmux

// apt
// apt install tmux
```

新建tmux会话

```
tmux new -s [session_name]
// 如 tmux new -s rust
```

> 此时应该已经进到tmux会话里了，以后再想进入这个会话可以通过`tmux ls` 列出已有回话然后`tmux attach -t [session_name]`进入会话

在tmux会话中运行端口转发程序：

```
./frps -c ./frps.ini
```

**到此为止，云服务器上的事情就做完了**，接下来我们在本机搭建游戏服务器并与云服务器进行通讯。

首先搭建游戏服务器：

下载steamcmd

- windows: [直接点这个链接](https://steamcdn-a.akamaihd.net/client/installer/steamcmd.zip)
- linux: 
  - Ubuntu/Debian: `useradd -m steam && cd /home/steam && sudo apt install steamcmd`
  - RedHat/CentOS: `useradd -m steam && cd /home/steam && yum install steamcmd`
  - Arch Linux: `useradd -m steam && cd /home/steam && git clone https://aur.archlinux.org/steamcmd.git && cd steamcmd && makepkg -si`

> 可以链接 **steamcmd** 可执行文件：`ln -s /usr/games/steamcmd steamcmd`
>
> docker: `docker run -it --name=steamcmd cm2network/steamcmd bash`
>
> 其他问题请参考[官方文档](https://developer.valvesoftware.com/wiki/SteamCMD)

接下来以windows为例：

解压steamcmd，并进入文件夹，建立一个与steamcmd.exe同级的文件`rust.bat`

修改这个文件为：

```
steamcmd +login anonymous +app_update 258550 validate +quit
```

这个命令会匿名登录steam并自动下载或更新RUST的服务器程序，建议以后每次都先运行这个.bat文件来验证现在的服务器程序是否是最新版本。

初次运行这个.bat文件会自动下载RUST服务器程序，等待下载完成后进入steamcmd文件夹下的`steamapps\common\rust_dedicated`目录，再新建一个`run.bat`的文件，写入服务器运行命令和参数：

```
start RustDedicated.exe -batchmode +server.hostname "[HOST_NAME]" +server.port 28015 +server.level "Procedural Map" +server.worldsize 4096 +server.seed 1 +rcon.port 29015 +rcon.password "[RCON_PASSWORD]" +server.saveinterval "300" +server.description "[SERVER_DESCRIPTION]" +bear.population 5 +boar.population 5 +horse.population 5 +stag.population 5 +wolf.population 5 +zombie.population 5 +server.idlekickmode 0 +server.itemdespawn "300" +server.identity server -logfile
```

如：

```
start RustDedicated.exe -batchmode +server.hostname "qindan233" +server.port 28015 +server.level "Procedural Map" +server.worldsize 4096 +server.seed 1 +rcon.port 29015 +rcon.password "qindan" +server.saveinterval "300" +server.description "mc_hit" +bear.population 5 +boar.population 5 +horse.population 5 +stag.population 5 +wolf.population 5 +zombie.population 5 +server.idlekickmode 0 +server.itemdespawn "300" +server.identity server -logfile
```

运行这个.bat文件来启动RUST服务器。实际上现在你已经可以访问本机的28015端口来进行单机的RUST游戏了。

接下来是配置本地的端口转发程序：

同样下载对应操作系统的[端口转发程序](https://github.com/fatedier/frp/releases)，我的电脑是win10 64位所以我这次下载的是[frp_0.34.3_windows_amd64.zip](https://github.com/fatedier/frp/releases/download/v0.34.3/frp_0.34.3_windows_amd64.zip)

更改`frpc.ini`来配置客户端程序：

```
[common]
server_addr = [SERVER_IP] 
server_port = 7000

[tcp]
type = tcp
local_ip = 127.0.0.1
local_port = 28015
remote_port = 6000

[udp]
type = udp
local_ip = 127.0.0.1
local_port = 28015
remote_port = 6001
```

> `[SERVER_IP]`处填写云服务器的IP地址

这里我是配了两个：tcp和udp，但实际上RUST只使用了UDP协议进行通讯，因此在本例子中只有udp部分是有效。有些游戏用的是TCP，也有TCP+UDP的，因此上面这个模板可以保留并重复使用。

> 在本例子中，本机将28015端口与云服务器的7000端口进行绑定并相互通讯，云服务器暴露6001端口对外进行UDP通讯。

运行本机的端口转发程序：

```
frpc -c frpc.ini
```

**大功告成！**

非本机的客户端在游戏中按F1进入控制台并输入`client.connect [云服务器IP]:6001`即可进入游戏。

本机客户端直接连接本机局域网ip地址无需占用云服务器带宽。

过程中有任何问题欢迎留言或发邮件与我讨论~



















