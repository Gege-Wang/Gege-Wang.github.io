---
layout: single
title: "three programming languages to write leetcode 01"
date: 2024-08-28
categories: leetcode
---
> 最近开始写 leetcode ， 突发奇想用三种语言写，其实我就会比较熟许 C/C++/Rust， 所以就用这三种语言写，其实之前一直都是用 C++，用其他的语言还是很有阻碍的，最近可能比较烧包，开始整花活了，就记录一下，以后可能都会尝试用这三种语言。


## 题目描述
**题目描述**: 给你两个字符串 word1 和 word2 。请你从 word1 开始，通过交替添加字母来合并字符串。如果一个字符串比另一个字符串长，就将多出来的字母追加到合并后字符串的末尾。返回合并后的字符串
示例 1：

输入：word1 = "abc", word2 = "pqr"
输出："apbqcr"
解释：字符串合并情况如下所示：
word1：  a   b   c
word2：    p   q   r
合并后：  a p b q c r 

## C++ 实现
```c++
class Solution {
public:
    string mergeAlternately(string word1, string word2) {
        int len1 = word1.size();
        int len2 = word2.size();
        string s;
        int n = max(len1, len2);
        for(int i = 0; i < n; i++) {
            if(i < len1){
                s.push_back(word1[i]);
            }
            if(i < len2) {
                s.push_back(word2[i]);
            }
        }
        return s;
        
    }
};
```
## C 实现
```c

char * mergeAlternately(char * word1, char * word2){
    int len1 = strlen(word1);
    int len2 = strlen(word2);
    int n = len1 > len2 ? len1 : len2;
    // c 需要用 malloc 申请内存
    char * s = (char*)malloc((len1 + len2 + 1) * sizeof(char));
    int l = 0;
    for(int i = 0; i < n; i++) {
        if(i < len1) {
            s[l++] = word1[i];

        }
        if(i < len2) {
            s[l++] = word2[i];
        }
    }
    // C 字符串结尾必须是 ‘\0' ，否则会报错
    s[l] = '\0';
    return s;

}

```

## Rust 实现
```Rust
use core::cmp::max;
impl Solution {
    pub fn merge_alternately(word1: String, word2: String) -> String {
        let len1 = word1.len();
        let len2 = word2.len();
        let  n = max(len1, len2);
        // rust 的 String 可以用 with_capacity 来指定初始容量
        let mut res = String::with_capacity(len1 + len2);
        let l = 0;
        for i in 0..n {
            if i < len1 {
                // rust 的 String 可以用 push 来添加字符 而且可以用 nth 来获取字符
                res.push(word1.chars().nth(i).unwrap());
            }
            if i < len2 {

                res.push(word2.chars().nth(i).unwrap());
            }
        }
        res
    }
}
```