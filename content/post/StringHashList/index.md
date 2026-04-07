---
title: Rust日记:字符串哈希实现
description: 一篇用rust实现字符串哈希的记录
date: 2026-01-30 17:30:00
tags:
  - Rust
  - 算法
categories:
  - Rust
---

# 字符串哈希

# 初版代码:

```rust
use std::cmp::Ordering;
use std::io::Read;
struct StringHash {
    list: Vec<Vec<String>>,
}

impl StringHash {
    pub fn new() -> Self {
        Self {
            list: vec![Vec::<String>::new(); 1000003],
        }
    }

    pub fn insert(&mut self, input: String) -> Result<(), ()> {
        if let Some(idx) = StringHash::get_hash(&input) {
            self.list[idx].push(input);
            Ok(())
        } else {
            Err(())
        }
    }

    pub fn query(&self, input: &str) -> Option<()> {
        if let Some(idx) = StringHash::get_hash(input) {
            for item in &self.list[idx] {
                if let Ordering::Equal = input.cmp(item) {
                    return Some(());
                }
            }
        }
        None
    }

    fn get_hash(input: &str) -> Option<usize> {
        const MOD: usize = 1_000_003;
        const P: usize = 131;
        let mut hash: usize = 0;
        for item in input.bytes() {
            hash *= P;
            hash += item as usize;
            hash %= MOD;
        }
        Some(hash)
    }
}

fn main() {
    let mut input = String::new();
    std::io::stdin()
        .read_to_string(&mut input)
        .expect("Failed to read!");
    let mut iter = input.split_whitespace();
    macro_rules! next {
        ($t:ty) => {
            iter.next().unwrap().parse::<$t>().unwrap()
        };
    }
    let n = next!(usize);
    let mut cnt = 0;
    let mut string_hash = StringHash::new();
    for _ in 0..n {
        let input = next!(String).clone();
        if string_hash.query(&input).is_none() {
            string_hash.insert(input).expect("Failed to insert!");
            cnt += 1;
        }
    }

    println!("{cnt}");
}
```
