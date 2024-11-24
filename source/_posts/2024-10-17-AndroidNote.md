---
title: CamDetectRe
urlname: CamDetectRe
date: 2024-10-17 22:02:19
typora-copy-images-to: 2024-10-17-CamDetectRe
tags: ["Android"]
---

## 经验

- 找旧版，旧版可能没加固，没检测
- 遇到uni-app勇敢改html/js文件，很多人没有防护意识的

## 抓某个进程的log

使用命令：`adb logcat --pid=$(adb shell pidof -s hiddencamdetector.futureapps.com.hiddencamdetector)`

## 加固

### 360加固

找到篇分析文章<https://oacia.dev/360-jiagu/> 一看字数$6.2\times 10^4$还是先放一下慢慢看吧

使用工具`DITOR`

### 检测hook的加固

比如网易易盾

需要使用改了art源码的Android系统

有个提供在线服务的网站：<https://nop.gs/>

### jadx打不开脱出的dex

checksum失败，修复dex头。

mt或<https://github.com/luoyesiqiu/DexRepair>

## frida

### 启动指定包名的应用

```
frida -l xxx.js -U -f package_name"
```

### 启动 工作资料/指定用户 的应用

```
~: adb shell pm list users                                11/04/2024 11:13:08 PM
Users:
        UserInfo{0:机主:4c13} running
        UserInfo{10: 壶中界 :1030} running
        UserInfo{95:DUAL_APP:20001010} running
```

```
frida -l xxx.js -U -f package_name --aux uid="(int)10"
```

### frida获取不到类

~~首先看看是不是没有加`Java.perform`~~

查找所有`dalvik.system.PathClassLoader`，然后一个一个地用来尝试查找类，写出来大概是这样

```javascript
Java.perform(function () {
  Java.choose("dalvik.system.PathClassLoader", {
    onMatch: function (instance) {
      // console.log(instance)
      // console.log(Java.ClassFactory)
      var factory = Java.ClassFactory.get(instance)
      try {
        var myClass = factory.use("d.a.a.a.c.d$a$b")
        var methodArr = myClass.class.getMethods();
        for (var m in methodArr) {
          console.log(methodArr[m]);
        }
        console.log(myClass["b"])
        myClass["b"].implementation = function (a) {
          console.log(`b is called`); 
          var stack = Java.use("java.lang.Exception").$new().getStackTrace();
          for (var i = 0; i < stack.length; i++) {
            console.log(stack[i].toString());
          }
          this["b"](a);
        };
        console.log("stop")
      } catch (e) {
        // console.log("next")
        // console.log(e)
      }
    },
    onComplete: function () {
      // console.log("Done")
    }
  })
})

```

## 魔改Android导致的问题

### scrcpy在小米的usb设置bug的情况下如何启用触摸

又名[不登陆小米账号启用 MIUI 的 ADB 调试（安全设置）和 ADB 应用安装（需 Root）](https://zhuanlan.zhihu.com/p/603628922)

```
setprop persist.security.adbinstall 1
setprop persist.security.adbinput 1
setprop persist.fastboot.enable 1

am force-stop com.miui.securitycenter
```

编辑`/data/data/com.miui.securitycenter/shared_prefs/remote_provider_preferences.xml`，加入
```
<boolean name="permcenter_install_intercept_enabled" value="false" />
<boolean name="security_adb_install_enable" value="true" />
```


## UI相关

### 获取当前Activity

```
adb shell "dumpsys window | grep mCurrentFocus"
```

### 获取窗口堆栈

```
adb shell dumpsys window windows |grep "Window #"
```

### 整个Activity的详细信息

```
adb shell dumpsys activity [package_name]
```