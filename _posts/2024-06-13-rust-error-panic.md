---
layout: single
title:  "rust 错误处理"
date:  2024-06-13
categories: rust
---
> 写这篇文章的目的是我没有建立好的错误处理思路，我风格混乱，我是个疯子。

1. ？
在错误的时候传播会调用者，并且立刻终止当前函数的运行， 不会运行下面的逻辑
2. unwrap()
在错误的时候会 panic, 用于绝对不会出错的场景
在 result 类型是获得 值或者在错误的时候 panic。
3. with_context(|| format!("this is the info {}", node))
包装错误信息
4. wrap_err_with(|| format!("this is the info: {:?}", node))
捕捉上下文 包装错误信息
5. wrap_err("this is the info");
在 result 类型时获得值或者在出错时在错误上下文中增加信息
6. bail!("this is the error")
7. context("cannot find sonething") 
包装错误信息
8. expect("this is the error")
这个也是在发生错误时 `panic`


Err(eyre!("..."))
- Err(eyre!("...")) 是直接创建一个包含错误信息的 EyreError 类型的错误，并将其作为一个 Result 的 Err 返回。
- 这种方式可以在任意位置创建并返回一个错误，不受宏的限制。
由于是直接创建一个错误并返回，可以进行更加灵活的错误处理，包括在错误发生后执行一些额外的逻辑或处理。
bail!
- bail! 是一个宏，用于创建一个 EyreError 并立即返回该错误，类似于使用 Err(eyre!("..."))，但更为简洁。
- 它在写代码时可以让错误处理更具表达力，提高了代码的可读性，尤其是对于一些简单的错误返回。它相当于一个简化版的 Err(eyre!("..."))。

选择使用哪个？
如果你需要在处理错误时进行一些额外的操作或逻辑处理，可能更适合使用 Err(eyre!("..."))，因为它更加灵活。
如果只是简单地创建一个带有错误信息的 EyreError 并立即返回，可以选择使用 bail! 宏，因为它更简洁、易读。