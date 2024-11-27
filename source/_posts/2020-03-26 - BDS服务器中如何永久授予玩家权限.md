---
title: BDS服务器中如何永久授予玩家权限
date: 2020-03-26 10:48:56
tags: [YouKnow]
urlname: 88

---
<!--markdown-->
1.登录服务器，
2.screen -d -r mc然后找到要改权限的人的xuid。xuid会在一个人连接或断开时会以下图形式记录
![GprYuQ.png](BDS%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%AD%E5%A6%82%E4%BD%95%E6%B0%B8%E4%B9%85%E6%8E%88%E4%BA%88%E7%8E%A9%E5%AE%B6%E6%9D%83%E9%99%90/GprYuQ.png)
3.Ctrl + a然后按d退出screen
4.nano permissions.json
5.![GprdNq.png](BDS%E6%9C%8D%E5%8A%A1%E5%99%A8%E4%B8%AD%E5%A6%82%E4%BD%95%E6%B0%B8%E4%B9%85%E6%8E%88%E4%BA%88%E7%8E%A9%E5%AE%B6%E6%9D%83%E9%99%90/GprdNq.png)
按这个格式写。加入一条新规则时，在最后一个大括号后面先写一个逗号，然后在后面添加一对大括号，在新加的大括号里面写"permission":"xx","xuid":"xxxxxxx"
permission后面成员对应member，访客对应visitor，op对应operator
可以不加空格和换行。但是为了看得清楚。。。。
6.Ctrl+X 按 Y 按回车退出nano
7.screen -d -r mc
8.permission reload
9.Ctrl + a然后按d退出screen
10.安全地退出ssh