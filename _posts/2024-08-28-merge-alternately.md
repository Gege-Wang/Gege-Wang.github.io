---
layout: single
title: "three programming languages to write leetcode 01"
date: 2024-08-28
categories: leetcode
---
> 最近开始写 leetcode ， 突发奇想用三种语言写，其实我就会比较熟许 C/C++/Rust， 所以就用这三种语言写，其实之前一直都是用 C++，用其他的语言还是很有阻碍的，最近可能比较烧包，开始整花活了，就记录一下，以后可能都会尝试用这三种语言。


## 题目描述01
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

## 题目描述02
**题目描述**: 对于字符串 s 和 t，只有在 s = t + t + t + ... + t + t（t 自身连接 1 次或多次）时，我们才认定 “t 能除尽 s”。

给定两个字符串 str1 和 str2 。返回 最长字符串 x，要求满足 x 能除尽 str1 且 x 能除尽 str2 。

 

示例 1：

输入：str1 = "ABCABC", str2 = "ABC"
输出："ABC"
示例 2：

输入：str1 = "ABABAB", str2 = "ABAB"
输出："AB"
示例 3：

输入：str1 = "LEET", str2 = "CODE"
输出：""

**解题方法**：这个问题就是最大公约数问题(Greatest Common Divisor)，我们可以使用辗转相除法，我们可以观察首先符合条件的 x 的长度一定是 str1 和 str2 长度的公约数，我们判断最大公约数， 如果最大公约数不是，会不会是更小的公约数呢，不会了，因为最大公约数已经可以代替这两个字符串了，如果这都不是，就说明再也找不到了。

## C++ 实现
```c++
class Solution {
public:
    bool check(string str, string prefix) {
        int len1 = str.length();
        int len2 = prefix.length();
        if(len1 % len2 ) {
            return false;
        }
        int n = len1 / len2;
        string s = "";
        for(int i = 0; i < n; i++) {
            s += prefix;
        }
        return s == str;
    }
    string gcdOfStrings(string str1, string str2) {
        int len1 = str1.length();
        int len2 = str2.length();
        int gcd_number = gcd(len1, len2);
        string prefix = str1.substr(0, gcd_number);
        if(check(str1, prefix) && check(str2, prefix)) {
            return prefix;
        }else {
            return "";
        }
    }
    int gcd(int a, int b) {
        if(b == 0) return a;
        return gcd(b, a % b);
    }
};

```
## c 实现
> c 实现中有一个必须说明的问题就是字符串的处理，c 里面字符串的末尾必须有 '\0'结尾，任何字符串都是，而且对于 malloc 申请的内存，我机会不会自己手动回收，这是一个非常不好的习惯。
```c

bool check(char* str, char* prefix) {
    int len1 = strlen(str);
    int len2 = strlen(prefix);
    char *s = (char*)malloc((len1 + 1)* sizeof(char));
    s[0]= '\0';
    int n = len1 / len2;
    for(int i = 0; i < n; i++) {
        printf("%s ", prefix);
        strcat(s, prefix);
    }
    return strcmp(s, str) == 0;
}


int gcd1(int a, int b) {
    while(b){
        int temp = a;
        a = b;
        b = temp % b;
    }
    return a;
}

char* gcdOfStrings(char* str1, char* str2) {
    int len1 = strlen(str1);
    int len2 = strlen(str2);
    int gcd = gcd1(len1, len2);
    char* prefix =(char*)malloc((gcd + 1) * sizeof(char));
    strncpy(prefix, str1, gcd);
    prefix[gcd] = '\0';
    if(check(str1, prefix)&&check(str2, prefix)){
        return prefix;
    }else{
        return "\0";
    }
}
```

## Rust 实现
> rust  里面需要特别关注的是字符串的所有权问题，同一时刻可变借用只能有一个，不能既有可变借用同时又转移所有权，这是不允许的。所以我们在写代码的时候要有所有权的意识，否则编译器不会让你通过，并且写 rust 明显感觉到编译器很耐心的告诉你应该怎么改代码，为什么要这么改，这对于一个学习者是非常友好的。并且 rust 能够很好的让我们在编程中对自己的变量有掌控权，在 C 里面就感觉所有的东西都是放任不管的，我们程序员必须有一套非常好的编程习惯才能驾驭。
```rust
impl Solution {
    pub fn gcd_of_strings(str1: String, str2: String) -> String {
        let len1 = str1.len();
        let len2 = str2.len();
        let gcd = Self::gcd(len1, len2);
        let prefix = &str1[0..gcd];
        if Self::check(&str1, prefix) && Self::check(&str2, prefix) {
            prefix.to_string()
        }
        else {
            String::new()

        }
    }
    fn check(str: &str, prefix: &str) -> bool {
        let len1 = str.len();
        let len2 = prefix.len();
        let n = len1 / len2;
        let mut s = String::with_capacity(len1);
        for i in 0..n {
            s.push_str(&prefix);
        }
        s == str

    }
    fn gcd(len1: usize, len2: usize) -> usize {
        if len2 == 0 {return len1;}
        Self::gcd(len2, len1 % len2)
    }
}
```