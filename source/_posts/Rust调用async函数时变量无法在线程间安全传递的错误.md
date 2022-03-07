---
title: Rust调用async函数时变量无法在线程间安全传递的错误
date: 2022-03-07 16:29:23
tags:
categories: 多线程
urlname: rust-var-need-drop-before-async
---
这是一段伪代码
```rust
async fn a() {
    let a:T = ...;//T一个没有实现 Send的类型
    let b = foo(a);
    boo(b).await;
}

async fn boo() -> i32 {
    ...
}
```
这种情况下，会有一个`future cannot be sent between threads safely`的错误。
解决方法是，保证`a`在`await`之前`drop`掉
```rust
async fn a() {
    let b;
    {
        let a:T = ...;//T一个没有实现 Send的类型
        b = foo(a);//这里提取的是可以Send的数据
    }
    boo(b).await;
}
```
不过手动`drop`是不行的
```rust
async fn a() {
    let a:T = ...;//T一个没有实现 Send的类型
    let b = foo(a);//这里提取的是可以Send的数据
    drop(a);//编译器不认
    boo(b).await;
}
```
更进一步的解释可以参考<https://blog.csdn.net/wangjun861205/article/details/118084436>