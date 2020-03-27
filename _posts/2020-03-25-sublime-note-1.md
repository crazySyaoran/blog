---
layout: post
title: SublimeText笔记
date: 2020-03-25
Author: Syaoran 
tags: [sublime, note, markdown]
comments: true
---

### - 调出命令面板  
`ctrl + shift + P`  
(MAC请使用 `command + shift + P`)

### - 调出控制台
```
ctrl + `  
```
(MAC用户同上)  

### - 安装插件
调出命令面板后输入`Package Control: Install Package`
> 或`pcip`快速查询  
> 安装时左下角会提示后台正在运行，稍等片刻，无需重复操作  
> 如需卸载已安装的插件请使用`Package Control: Remove Package`

### - 禁用已安装的插件
- 调出命令面板，使用`Package Control: Disable Package`
- 或在配置文件中一次性添加：  
Perferences -> Setting ->   
```	
"ignored_packages":   
[  
    "XXXX",   
],  
```

