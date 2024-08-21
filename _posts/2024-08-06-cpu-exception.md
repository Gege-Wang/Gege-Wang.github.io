---
layout: single
title: "cpu exception handling"
date: 2024-08-06
categories: candy
---

## 为什么需要 CPU exception handling 
在冯诺依曼的架构下，程序是按顺序执行的，也就是说当我们生成一个机器指令流，程序会按照这个顺序依次执行。但是有些执行的正确性是有条件的，比如一个除法指令，他的除数必须是一个非零的数；又比如整数溢出。这些错误指令是 CPU 不能执行的，如果 CPU 执行到了指令会检查这是不是一条正确指令，正确则执行，错误则抛出异常，那么抛出的异常是用什么表示呢？ 操作系统又如何捕捉抛出的异常？这些事情在 [intel ASDM](https://cdrdv2.intel.com/v1/dl/getContent/671447) 中有详细的表述。



## reference
[1] [intel ASDM](https://cdrdv2.intel.com/v1/dl/getContent/671447)
[2] [Three Easy Pieces](https://pages.cs.wisc.edu/~remzi/OSTEP/)
[3] [OSDev wiki](https://wiki.osdev.org/Expanded_Main_Page)