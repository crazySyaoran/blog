---
layout: post
title: windows下为terminal设置代理
date: 2020-07-12
Author: Syaoran
tags: [windows, proxy, terminal]
comments: true
---

最近换了pc所以重新开始学着用windows（是真的难用）

因为要用SBT所以要给terminal设置代理：
```
set HTTP_PROXY=http://192.168.31.5:10809
set HTTPS_PROXY=http://192.168.31.5:10809
```

要注意几点：
- ping命令是ICMP协议，不是TCP/UDP协议，因此设置成功也不代表能ping通google
- `http://`不能省略
- `192.168.31.5`替换成`127.0.0.1`似乎会出问题
- 端口应与ss或v2ray的HTTP代理监听端口保持一致

正常来说SBT不报代理相关的warn应该就是对了













