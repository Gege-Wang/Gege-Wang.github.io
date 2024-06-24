---
layout: single
title:  "rust 中 build.rs 构建脚本的作用"
date:  2024-06-19
categories: rust
---

# build.rs 
rust 中的特殊构建脚本。它能做的包括生成代码，构建依赖项，编译 C/C++ 代码。
println! 宏在 build.rs 中的作用不仅仅是普通的输出，它实际上是向 Cargo 传递特定的编译指令。这些指令可以控制 Cargo 的行为，比如重新运行构建脚本的条件、设置编译器标志等。
```
fn main() {
    // 通过 `println!` 宏来向 Cargo 输出指令
    println!("cargo:rerun-if-changed=src/my_ffi.c");
    
    // 构建某些非 Rust 依赖项。在编译 rust 项目前， 在 build.rs 中调用 cc crate 来编译 C/C++ 源文件。
    cc::Build::new()
        .file("src/my_ffi.c")
        .compile("my_ffi");
}
```

以下是一些常用的 Cargo 输出指令示例：

重新运行条件：
```
cargo:rerun-if-changed=path/to/file：当指定的文件更改时，重新运行构建脚本。
cargo:rerun-if-env-changed=VAR：当指定的环境变量更改时，重新运行构建脚本。
```
链接库：
```
cargo:rustc-link-lib=static=foo：链接名为 foo 的静态库。
cargo:rustc-link-search=native=path/to/library：添加库搜索路径。
```
编译器标志：
```
cargo:rustc-cfg=foo：向编译器添加 cfg 标志。
```

# build.rs 应用

*TODO*