---
title: 记录一下思源服务端的安装过程
date: 2022-03-15 00:37:00

tags: []
    - 服务器
    - docker
urlname: siyuan-install
---
第一步，先在本地wsl上逝一逝  
首先安装docker，然后docker服务起不来，给我报  
```
Err :connection error: desc = "transport: Error while dialing dial unix:///var/run/docker/containerd/containerd.sock: timeout". Reconnecting...  module=grpc
failed to start daemon: Error initializing network controller: error obtaining controller instance: unable to add return rule in DOCKER-ISOLATION-STAGE-1 chain:  (iptables failed: iptables --wait -A DOCKER-ISOLATION-STAGE-1 -j RETURN: iptables v1.8.7 (nf_tables):  RULE_APPEND failed (No such file or directory): rule in chain DOCKER-ISOLATION-STAGE-1
 (exit status 4))
```
经过我的精心搜索，需要这样：  
`sudo update-alternatives --set iptables /usr/sbin/iptables-legacy`  
然后就好了  
接着跑思源  
要保留数据需要映射文件夹，比如这样  
`docker run -v /root/siyuan/:/siyuan/workspace -p 6806:6806 -u 1000:1000 b3log/siyuan --workspace=/siyuan/workspace/`  
但是如果报  
`create conf folder [/siyuan/workspace/conf] failed: mkdir /siyuan/workspace/conf: permission denied`  
说明你容器外面的那个映射的文件夹权限不够（我也不知道为什么，不是root吗？），chmod 777就可以了。  
再放上我那30一年的服务器：
```
docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: process_linux.go:545: container init caused: rootfs_linux.go:75: mounting "proc" to rootfs at "/proc" caused: mount through procfd: permission denied: unknown.
ERRO[0020] error waiting for container: context canceled
```
我靠，这服务器跑在lxc上的跑不了docker
