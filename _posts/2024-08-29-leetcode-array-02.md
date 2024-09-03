---
layout: single
title: "leetcode array 02"
date: 2024-08-29
categories: leetcode
---
## 题目描述01
给你一个数组 candies 和一个整数 extraCandies ，其中 candies[i] 代表第 i 个孩子拥有的糖果数目。

对每一个孩子，检查是否存在一种方案，将额外的 extraCandies 个糖果分配给孩子们之后，此孩子有 最多 的糖果。注意，允许有多个孩子同时拥有 最多 的糖果数目。
```c++
class Solution {
public:  
    vector<bool> kidsWithCandies(vector<int>& candies, int extraCandies) {  
        int max = 0;
        vector<bool> res;
        for(int i = 0; i < candies.size(); i++) {
            if(candies[i] > max) {
                max = candies[i];
            }
        } 
        int threshold = max - extraCandies;
        for(int i = 0; i < candies.size(); i++) {
            if(candies[i] >= threshold) {
                res.push_back(true);
            } else {
                res.push_back(false);
            }
        }
        return res;

    }
};  
```
```c
/**
 * Note: The returned array must be malloced, assume caller calls free().
 * 在 C 语言中操作指针是一项危险的操作，程序员必须保证自己的操纵是有效的，否则就会导致未定义的行为，包括程序崩溃或者数据损坏。 C++ 会通过工具减少越界访问的风险。std::vector：C++ 的标准库提供了动态数组类 std::vector，它在访问元素时可以选择使用带边界检查的 at() 方法。at() 方法在访问越界时会抛出 std::out_of_range 异常。 
 * 面向过程编程和面向对象编程在指针上就能体现的很明显。我们的入参使用单纯的数组，实际上并不掺杂其他任何的信息，我们需要另一个参数来表示数组的大小。但是在 C++ 中，一个 vector 就可以代替一个数组，并且可以自动管理数组的大小。程序员可以通过方法来获得数组的大小。
 */

bool* kidsWithCandies(int* candies, int candiesSize, int extraCandies, int* returnSize) {
    bool* res = (bool*)malloc(candiesSize * sizeof(bool));
    int max = candies[0];
    for(int i = 0; i < candiesSize; i++) {
        if(candies[i] > max) {
            max = candies[i];
        }
    }
    int threshold = max - extraCandies;
    for(int i = 0; i < candiesSize;i++) {
        if(candies[i] >= threshold) {
            res[i] = true;
        }else {
            res[i] = false;
        }
    }
    *returnSize = candiesSize;
    return res;
}
```

## Rust 实现

```rust
///  rust 的实现中更多的使用了迭代器，以及函数式编程的思想。
/// 可以观察到在 max 的 match 分支当中，我们其实就不会漏掉 max 为空的时候。可以看到我们之前在 C++ 中，这种特殊情况是需要程序员特殊处理的。但是在 rust  的编程规范中，我们能够用更加优雅的方式处理。
impl Solution {
    pub fn kids_with_candies(candies: Vec<i32>, extra_candies: i32) -> Vec<bool> {
        let max = match candies.iter().max() {
            Some(max_value) => max_value,
            None => return Vec::new(),
        };
        candies.iter().map(|nums| max - nums <= extra_candies).collect();
    }
}
```

## 题目描述02
求二叉树的最大深度
```rust 
// 现在我们用 rust 来解释这道题
// Definition for a binary tree node.
// #[derive(Debug, PartialEq, Eq)]
// pub struct TreeNode {
//   pub val: i32,
//   pub left: Option<Rc<RefCell<TreeNode>>>,
//   pub right: Option<Rc<RefCell<TreeNode>>>,
// }
//
// impl TreeNode {
//   #[inline]
//   pub fn new(val: i32) -> Self {
//     TreeNode {
//       val,
//       left: None,
//       right: None
//     }
//   }
// }
use std::rc::Rc;
use std::cell::RefCell;
use std::cmp::max;
impl Solution {
    pub fn max_depth(root: Option<Rc<RefCell<TreeNode>>>) -> i32 {
        match root {
            Some(node) => {
                let node = node.borrow(); // 对 RefCell<T> 类型的数据借用，需要使用 borrow 方法，而不是直接使用 &node
                let left = node.left.clone(); // 这里使用 clone 方法来增加 Rc<RefCell<T>> 的引用计数 ，而不是直接引用
                let right = node.right.clone();

                let height_left = Self::max_depth(left);
                let height_right = Self::max_depth(right);
                max(height_left, height_right) + 1
            },
            None => 0,
        }
    }
}
```