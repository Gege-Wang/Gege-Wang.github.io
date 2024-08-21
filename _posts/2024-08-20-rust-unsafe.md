---
layout: single
title: "rust 语言特性"
date: 2024-08-20
categories: candy
---

> 写这篇文章的目的在于记录在页表转换过程中使用 `unsafe` 造成的一些问题，以及 `rust` 在升级过程中产生的一些不同。

1. 首先我们有一个 unsafe 的地址翻译的函数，然后返回的是一个 Option 的值。我们想要在函数调用之后，输出返回值，那么我们可以有以下的代码
```rust
for &address in &addresses {
        let virt = VirtAddr::new(address);
        let phys = unsafe { translate_addr(virt, phys_mem_offset); };   // unsafe 里面加了一个分号，这样类型判断就完全不一样了
        println!("{:?} -> {:?}", virt, phys);
}

```
但是其实这样我们可以看到，rust 的自动类型判断 phys 的类型是 () ，而不是一个 Option<PhysAddr> 所以我们直接打印的结果并不是物理地址而是() 。 
而我们使用下面的代码就是正确的。

```rust
        for &address in &addresses {
            let virt = VirtAddr::new(address);
            unsafe{
                let phys = translate_addr(virt, physical_mem_offset);
                println!("{:?} -> {:?}", virt, phys);
            };
        }
```

## 使用 hlt_loop 降低功耗

## 条件编译中的 与或非
```rust
#[cfg(all(target_arch = "x86_64", target_os = "linux"))]
fn linux_specific() {}

#[cfg(any(feature = "instructions", target_arch = "x86_64"))]
fn feature_or_arch() {}

#[cfg(not(feature = "instructions"))]
```
## 关于 trait 对象的使用
- &mut impl FrameAllocator<Size4KiB>
静态分发，接受任何实现了 FrameAllocator 的可变引用，会在编译时确定具体的类型。没有运行时开销。
- &mut dyn FrameAllocator<Size4KiB>
动态分发，接受一个指向实现 FrameAllocator 的 trait 对象的可变引用，调用函数会在运行时通过虚函数表动态进行。会有一定的运行时开销。

## 如何分辨 static 和 Box 这类分配在堆上的数据结构
我们知道在内核的初期阶段是没有堆的，但是我们还是需要像 idt 这样的数据结构，仅仅分配在栈上生命周期是不够的，所以我们需要静态变量，但是如果不是在堆上，那么静态变量是分配在哪里的呢？
静态变量是在以区别于栈的固定的地方。rust 规定静态变量是只读的，因为我们要避免数据竞争，也就是避免多个地方同时改动导致数据不一致。但是如果我们要改静态变量需要用 `Mutex` 包一层，这样我们可以保证同一时间只有一个可变引用。

## 我们为什么需要堆（动态数据）
- 局部变量：函数调用结束后，局部变量会被销毁
- 静态变量：整个程序运行过程中静态变量都不会被释放，即便我们不再需要使用它们。它们会一直占用固定的内存。而且它们可以被任何一个函数调用。
  局部变量和静态变量都有各自的限制，并且它们都是固定大小的。并不能动态的扩展大小。

为什么我们的内核需要堆？   
我们需要在两种地方使用堆：一种是动态生命周期，另一种是动态尺寸的数据类型。
我们将会在进程管理中使用动态大小的数据结构来管理进程。