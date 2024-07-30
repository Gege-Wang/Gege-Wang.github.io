---
layout: single
title: "candy test"
date: 2024-07-30
categories: candy, os 
---

## qemu 参数

## trait object
`Fn()` 是一个 
```rust
#[cfg(test)]
fn test_runner(tests: &[&dyn Fn()]) {
    serial_println!("Running {} tests", tests.len());
    for test in tests {
        test();
    }
    exit_qemu(QemuExitCode::Success);
}
```
我们需要为每个 `test` 对象创建一个方法，这个方法能够输出 `test` 的名字，然后调用 `test` 对象。这样的话，我们就不需要在每个 `test` 里面输出测试的名字和是否测试成功。

## pub struct 的成员是不公开的

## 
