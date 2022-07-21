---
title: tcp握手包（服务器）在linux与windows下的重传次数默认值
date: 2022-07-21 21:50:00
categories: 网络
urlname: tcp_and_os
tags:
---
建立连接前两步：  
syn->  
syn+ack<-  
在这里客户端超时，服务器会重传。  
默认情况下linux会重传5次，windows：2次  
（可以改）
