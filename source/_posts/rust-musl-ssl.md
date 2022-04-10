---
title: 解决rust编译目标为musl时openssl报错
date: 2022-04-11 00:09:00
categories: YouKnow
urlname: rust-musl-ssl
tags:
---
为了节省容器启动时间，准备把rust写的api编译好后扔进docker里，于是编译到target:x86_64-unknown-linux-musl  
然后openssl炸了，不认libssl-dev了，查了下要重新编译。。。  
但是，我们发现了一个神奇的docker镜象<https://github.com/emk/rust-musl-builder>，它已经配好了openssl的musl环境  
于是只要这样： 
```
alias rust-musl-builder='docker run --rm -it -v "$(pwd)":/home/rust/src ekidd/rust-musl-builder'
rust-musl-builder cargo build --release
```
就可以了

（不过好像也可以把openssl换成rustls来解决）
> However, rustls now works well with most of the Rust ecosystem, including reqwest, tokio, tokio-postgres, sqlx and many others. The only major project which still requires libpq and OpenSSL is Diesel. If you don't need diesel or libpq:

>    See if you can switch away from OpenSSL, typically by using features in Cargo.toml to ask your dependencies to use rustls instead.
>    If you don't need OpenSSL, try cross build --target=x86_64-unknown-linux-musl --release to cross-compile your binaries for libmusl. This supports many more platforms, with less hassle!
