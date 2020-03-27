---
layout: post
title: 在Sublime中实现Markdown的实时预览
date: 2020-03-25
Author: Syaoran 
tags: [sublime, note, markdown]
comments: true
---


## 配置Markdown的实时预览
注意，连接SublimeText的服务器需要翻墙，请尽可能保证VPN处于全局状态  

### MarkdownEditing   
类似于SublimeText中的Markdown语法编辑器，会强制更改你Sublime的主题颜色，如果不喜欢可以禁用掉它。  

### MarkdownPreview  
支持在浏览器中预览markdown文件，也可以将md文件导出为html代码  
将md文件用浏览器预览：命令面板中使用`Markdown Preview : Preview in Browser`
> 出现两个选项：`github`和`markdown`。任选其一即可，github是利用GitHub的在线API来解析.md文件，支持在线资源的预览，如在线图片它的解析速度取决于你的联网速度。该方式据说一天只能打开60次。markdown就是传统的本地打开，不支持在线资源的预览。  

也可以设置快捷键：  
Preferences -> Key Bindings ->  
在右侧的User部分加入  
```
{ "keys": ["alt+m"], "command": "markdown_preview", "args": {"target": "browser", "parser":"markdown"}  } 
```
> "alt+m" 可设置为自己喜欢的按键。  
> "parser": "markdown"也可设置为"parser":"github"，改为使用Github在线API解析markdown。

### MarkdownPreview + LiveReload
实时刷新预览  

首先配置好MarkdownPreview  
配置LiveReload：
调出命令面板，使用`Package Control: Disable Package`，安装`LiveReload`  
安装成功后，再次调出命令面板，输入`LiveReload: Enable/disable plug-ins`，在弹出的选项中选择`Simple Reload with delay (400ms)`或者`Simple Reload`
> 两者的区别仅仅在于后者没有延迟。  

### 现在MarkdownPreView的网页会在你每次保存后自动刷新


