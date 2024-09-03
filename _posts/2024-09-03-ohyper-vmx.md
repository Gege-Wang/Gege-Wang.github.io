---
layout: single
title: "ohyper VMX"
date: 2024-09-03
categories: ohyper
---
## 相关指令
CPUID: 查看 CPU 支持的硬件特性的指令。
## github workflow 的写法
```yaml
name: Build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        arch: [x86_64]
        os: [ubuntu-latest]
    steps:
    - uses: actions/checkout@v2
    - uses: action-rs/toolchain@v1
      with:
        profile: minimal
        toolchain: stable
        components: rustfmt, clippy, rust-src
    - name: Build ohyper
      run: cargo build --release
  Clippy:
    runs-on: ubuntu-latest
    strategy:
      fail-fast: false
      matrix:
        arch: [x86_64]
        os: [ubuntu-latest]
    steps:
    - uses: actions/checkout@v2
    - uses: action-rs/toolchain@v1
      with:
        profile: minimal
        toolchain: stable
        components: clippy
        - name: Run Clippy
        run: cargo clippy --all-targets --all-features -- -D warnings
```

