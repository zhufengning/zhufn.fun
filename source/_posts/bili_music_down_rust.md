---
title: 学习rust后，写出下载B站视频的音频的GUI应用
date: 2021-09-08 13:30:00
tags: [YouKnow]
urlname: bili_music_down_rust

---

从Bilibili的视频收藏夹中下载音乐，仅下载音频。  
Download music from video lists of bilibili, audio only.  
目前~只有终端版本，之后可能搞个gui+二维码登录~有GUI，但只能用Cookie登录  

首先我们看一下截图 

![截图1](bili_music_down_rust/086be60c.png)

![截图2](bili_music_down_rust/127a029f.png)

![截图3](bili_music_down_rust/3405b239.png)

![截图4](bili_music_down_rust/f46f3ef8.png)

![截图5](bili_music_down_rust/20cdf1e1.png)

没错，就这么小一个东西我写了一个多月才写出来，我果然还是那么菜。  

地址：<https://gitlab.com/zhufengning/bili_music_download>

## Release Note  

### 2   

+ 加入GUI  
+ 加入视频选择  

### 1  

只能下载整个收藏夹，不会跳过已下载文件（不过下载完成后才会写入文件所以下到一半可以放心Ctrl+C）