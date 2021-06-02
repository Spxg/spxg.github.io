---
title: Rust生命周期学习
tags: []
categories:
  - - Rust
date: 2021-01-31 20:38:41
---

## 生命周期学习

### 出个题目：

```rust
#[derive(Debug)]
struct NumRef<'a>(&'a i32);

impl<'a> NumRef<'a> {
    fn change(&'a mut self, num: &'a i32) -> i32 {
        self.0 = num;
        *self.0
    }
}

fn main() {
    let mut num_ref = NumRef(&5);
    let a = num_ref.change(&6);
    println!("{:?}", a);
}
```

* 问题一：*以上代码编译是否会出错,为什么?如果会出错,怎么修改让它不出错？*
* 问题二： *假如你已经解决了第一题，那代码这么写的会有什么局限性?* 
* 问题三：*添加一行代码，证明第二题你所认为的局限性*
* 问题四：*按照第二题你所认为的局限性,修改代码,消除局限性*
* 问题五：*本题你学到了什么*



### 答案

* 一，CV一把梭，就知道能编译过去了，所以答案是不会出错。

* 二，根据生命周期分析，`NumRef`和`change`方法中的`self`(注：`&'a mut self`是`self: &'a mut Self` 的简写)标记的生命周期都是`'a`，可以理解为这个可变引用的生命周期和结构体的生命周期一样长，根据可变引用的独占性，`num_ref`一但调用`change`方法，之后其任何引用的使用都会触发借用检查的报错，所以局限性在于只能调用一次`change`方法(这样的应用很少)

* 三，添加`let _ = num_ref.change(&7);`  触发报错：

``` rust
error[E0499]: cannot borrow `num_ref` as mutable more than once at a time
  --> src/main.rs:14:13
   |
13 |     let a = num_ref.change(&6);
   |             ------- first mutable borrow occurs here
14 |     let _ = num_ref.change(&7);
   |             ^^^^^^^
   |             |
   |             second mutable borrow occurs here
   |             first borrow later used here
```

添加` println!("{:?}", num_ref);`   触发报错(`println!`使用了引用)：

```rust
error[E0502]: cannot borrow `num_ref` as immutable because it is also borrowed as mutable
  --> src/main.rs:15:22
   |
13 |     let a = num_ref.change(&6);
   |             ------- mutable borrow occurs here
14 |     println!("{:?}", a);
15 |     println!("{:?}", num_ref);
   |                      ^^^^^^^
   |                      |
   |                      immutable borrow occurs here
   |                      mutable borrow later used here
```

* 四，删掉`&'a mut self`中的`'a`即可
* 五，别瞎写生命周期参数
