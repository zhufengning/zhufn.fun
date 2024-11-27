---
title: F# bigint类型范围随机数
urlname: F# bigint类型范围随机数
date: December 24, 2022 3:21 AM
tags: [F#]
---
```fsharp
let rnd x y =
    let r = Random()
    let mutable res = 0I
    let mutable first = true

    while not first && not (res >= x && res <= y) do
        first <- false
        let length = BigInteger.Log(y, 2) |> Math.Ceiling
        let numBytes = length / 8.0 |> Math.Ceiling |> Convert.ToInt32
        let data: byte[] = Array.zeroCreate numBytes
        r.NextBytes(data)
        res <- new BigInteger(data)

    res
```
