---
title: Rust 宏(Macro)
tags: []
id: '793'
categories:
  - - Rust
date: 2021-01-16 21:25:12
---

## Rust 宏(Macro)

​ 众所周知, _Rust_提供了一个强大的宏系统，可进行元编程(metaprogramming)。_Rust_ 中的宏几乎无处不在，其实你们写的第一个_Rust_ 程序里面就已经用到了宏，比如`println!`，宏看起来和函数很像，只不过名称末尾有一个感叹号 `!` 。宏并不产生函数调用，而是展开成源码，并和程序的其余部分一起被编译。 ​ 由于才疏学浅，不敢班门弄斧，所以主要从例子出发，不深究原理，文章适合有_Rust_基础的同学看，官方文档 [The Rust Programming Language](https://doc.rust-lang.org/book/ch19-06-macros.html) ​ 我的Github: [上铺小哥Spxg](https://github.com/Spxg)

### 与C的宏有什么不同

_C_的宏：

```c
define ONE_PLUE_ONE 2
// ....

// output: 2
printf("%d", ONE_PLUS_ONE);
```

_Rust_的宏：

```rust
macro_rules! ONE_PLUS_ONE {
    () => { 2 };
}
// ....

// output: 2
println!("{}", ONE_PLUS_ONE!());
```

可能有同学要问了，_Rust_的宏与_C_的相比有什么不同。不同的是，_Rust_的宏会展开为抽象语法树(AST，abstract syntax tree)，而不是像字符串预处理那样直接替换成代码，这样就不会产生无法预料的优先权错误。

### 动手整个map!

了解_Rust_的同学，相比对`vec!`不陌生，`vec![1, 2, 3]`相当于

```rust
let mut v = Vec::new();
v.push(1);
v.push(2);
v.push(3);
```

可见宏使用可以带来便捷，那我们可不可以为_HashMap_实现相似的功能呢，当然可以，考虑下面几行代码：

```rust
// {
//     "me" => "laji",
//     "you" => "good"  
// }
let mut map = HashMap::new();
map.insert("me", "laji");
map.insert("you", "good");
```

我们不妨设计成_key_ => _value_的模式，像Ruby一样，将其插入到一个Map中，如下代码：

```rust
marco_rule! map {
    // $(...), + will be expanded for each matches 
    // Within $() is $x:expr, which matches any Rust expression and gives the expression the name $x
    ($($key: expr => $value: expr), +) => { {
        let mut map = HashMap::new();
        $(map.insert($key, $value);), +
        map
    } }
}
```

这样就可以使用map!来生成：

```rust
let map = map!(
    "me" => "laji",
    "you" => "good"
);
```

不难发现，其精髓还是对号入座，可以在脑子里替换进去

### 玩个大点的

同学们都知道`fibonacci`数，我们都知道这列值可以永远持续下去，定义一个`fibonacci`的求值函数略显困难。显然，返回一整列值并不实际。我们真正需要的，应是某种具有惰性求值性质的东西——只在必要的时候才进行运算求值。 在Rust中，这样的需求表明，是`Iterator`派上用场的时候了。实现迭代器并不十分困难，但比较繁琐：你得自定义一个类型，弄明白该在其中存储什么，然后为它实现`Iterator` trait。 其实，递推关系足够简单；几乎所有的递推关系都可被抽象出来，变成一小段由宏驱动的代码生成机制。 但是我们不写`fibonacci`，写了就有点大材小用~~脱裤子放屁~~了，就为了一个`fibonacci`写宏多不值啊，编译器展开和自己写还不是一样。所以，我们整个比较通用的。

#### 设计模型

同学们都是学过来的，想必都遇到过这些东西： `a = 1, 2, ..., n` `a = 1.0, ..., n * a[n - 1]` `fib = 0, 1, ..., fib[n - 1] + fib[n - 2]` 我们通过自己设计的宏假装实现一下：

```rust
recurrence!(a[n]: u32 = 0, 1 ... n);
recurrence!(a[n]: f64 = 1.0 ... n * a[n - 1]);
recurrence!(fib[n]: u32 = 0, 1 ... fib[n - 1] + fib[n - 2]);
```

#### 列出方程

以`fibonacci`为例， `fib[0] = 0` `fib[1] = 1` `fib[2] = fib[2 - 1] + fib[2 - 2] = 1` `fib[3] = fib[3 - 1] + fib[3 - 2] = 2` `......` 可以知道，我们只要有两个初始值就可以通过迭代算出任意一个`fibonacci`数，所以只需定义一个数组，列出方程：

```rust
// pseudocode
let fib = [0, 1];
let length = fib.length();
if n < length {
    fib[n]
} else {
    // offset is the times of iter
    fib[n - 1 - offset + length] + fib[n - 2 - offset + length]
}
```

#### 设计程序

有了方程后，我们开始做更细化的操作，解决问题，如`fib[n - offset + length]`。有的同学就要问了：这有啥问题，不是很符合逻辑吗？的确很符合，但是有大坑啊，`n - 2 - offset` 在`n`为2开始的时候程序就panic了。原因是：在_Rust_中， 数组的Index是`usize`类型，必是大于等于0，`n`为2开始，其中间运算小于0，直接爆炸，所以我们需要改改逻辑：

```rust
let a = n as i32 - 1 - offset as i32 + length as i32;
let b = n as i32 - 2 - offset as i32 + length as i32;
fib[a as usize] + fib[b as usize];
```

看了以后，可能有些同学觉得这~~尼玛是啥~~太复杂了，很不美观，所以我们再想想。在_Rust_中，有个`std::num::Wrapping`类型，它就是干这事的，话不多说，看代码

```rust
// Provides intentionally-wrapped arithmetic on T.

// Operations like + on u32 values are intended to never overflow, and in some debug configurations overflow is detected and results in a panic. While most arithmetic falls into this category, some code explicitly expects and relies upon modular arithmetic (e.g., hashing).

// Wrapping arithmetic can be achieved either through methods like wrapping_add, or through the Wrapping<T> type, which says that all standard arithmetic operations on the underlying value are intended to have wrapping semantics.

// The underlying value can be retrieved through the .0 index of the Wrapping tuple

use std::num::Wrapping;
let n = Wrapping(n);
let offset = Wrapping(offset);
let length = Wrapping(length);
let real_index = n - offset + length;
fib[real_index.0]
```

总算比上面一坨好看多了。

#### 开始操作

_Rust_ 提供了Iterator trait，使我们更好使用，而且还有好多实现了Iterator类型的方法，用它就完了，trait 长这样：

```rust
trait Iterator {
    type Item
//  The type of the elements being iterated over.

//  Required methods
fn next(&mut self) -> Option<Self::Item>
//  Advances the iterator and returns the next value.

// Returns None when iteration is finished. Individual iterator implementations may choose to resume iteration, and so calling next() again may or may not eventually start returning Some(Item) again at some point.
}
```

在_Rust_中，`[]`操作来源于`Index<T>` trait, 其中的`T`就是Index的类型，在这里就是`usize`，`Index<T>` trait 长这样：

```rust
trait Index<T> {
    type Output: ?Sized
    // The returned type after indexing.

    // Required methods
    fn index(&self, index: Idx) -> &Self::Output
    // Performs the indexing (container[index]) operation.
}
```

直接开始写：

```rust
use std::ops::Index;
const MEM_SIZE: usize = 2;

struct Solution {
    mem: [u32; MEM_SIZE],
    pos: usize,
}

impl Solution {
    fn new() -> Self {
        Self {
            mem: [0, 1],
            pos: 0,
        }
    }
}

struct IndexOffset<'a> {
    slice: &'a [u32; MEM_SIZE],
    offset: usize,
}

impl<'a> Index<usize> for IndexOffset<'a> {
    type Output = u32;

    #[inline(always)]
    fn index<'b>(&'b self, index: usize) -> &'b Self::Output {
        use std::num::Wrapping;
        let index = Wrapping(index);
        let offset = Wrapping(self.offset);
        let window = Wrapping(MEM_SIZE);
        let real_index = index - offset + window;
        &self.slice[real_index.0]
    }
}

impl Iterator for Solution {
    type Item = u32;

    #[inline]
    fn next(&mut self) -> Option<Self::Item> {
        if self.pos < MEM_SIZE {
            let value = self.mem[self.pos];
            self.pos += 1;
            Some(value)
        } else {
            let next_val = {
                let offset = self.pos;
                let fib = IndexOffset { slice: &self.mem, offset };
                fib[offset - 1] + fib[offset - 2]
            };

            {
                // move forward
                use std::mem::swap;
                let mut swap_tmp = next_val;
                for i in (0..MEM_SIZE).rev() {
                    swap(&mut swap_tmp, &mut self.mem[i]);
                }
            }

            self.pos += 1;
            Some(next_val)
        }
    }
}

fn main() {
    let fib = Solution::new();

    // iter 10 times
    for f in fib.take(10) {
        println!("{}", f);
    }
}
```

编译运行一把梭，轻松秒杀：

```rust
0
1
1
2
3
5
8
13
21
34
```

有了这个经验，写宏就很简单了，首先要解决的是，怎么让宏知道你初始有几个元素？套娃就对了！直接上代码：

```rust
macro_rules! count_exprs {
    () => { 0 };
    ($head: expr) => { 1 };
    ($head: expr, $($tail: expr), *) => { 1 + count_exprs!($($tail), *) }
}
```

```rust
macro_rules! recurrence {
    ($seq: ident[$ind: ident]: $sty: ty = $($inits: expr), + ... $recur: expr) => { {
        use std::ops::Index;
        const MEM_SIZE: usize = count_exprs!($($inits: expr), +);

        struct Solution {
            mem: [$sty; MEM_SIZE],
            pos: usize,
        }

        struct IndexOffset<'a> {
            slice: &'a [$sty; MEM_SIZE],
            offset: usize,
        }

        impl<'a> Index<usize> for IndexOffset<'a> {
            type Output = $sty;

            #[inline(always)]
            fn index<'b>(&'b self, index: usize) -> &'b Self::Output {
                use std::num::Wrapping;
                let index = Wrapping(index);
                let offset = Wrapping(self.offset);
                let window = Wrapping(MEM_SIZE);
                let real_index = index - offset + window;
                &self.slice[real_index.0]
            }
        }

        impl Iterator for Solution {
            type Item = $sty;

            #[inline]
            fn next(&mut self) -> Option<Self::Item> {
                if self.pos < MEM_SIZE {
                    let value = self.mem[self.pos];
                    self.pos += 1;
                    Some(value)
                } else {
                    let next_val = {
                        let $ind = self.pos;
                        let $seq = IndexOffset { slice: &self.mem, offset: $ind };
                        $recur
                    };

                    {
                        use std::mem::swap;
                        let mut swap_tmp = next_val;
                        for i in (0..MEM_SIZE).rev() {
                            swap(&mut swap_tmp, &mut self.mem[i]);
                        }
                    }

                    self.pos += 1;
                    Some(next_val)
                }
            }
        }

        Solution { mem: [$($inits), +], pos: 0 }
    } }
}
```

我们测试一波：

```rust
fn main() {
    recurrence!(fib[n]: u32 = 0, 1 ... fib[n - 1] * fib[n - 2]);
    return;
}
```

报错了：

```rust
error: `$inits:expr` may be followed by `...`, which is not allowed for `expr` fragments
 --> src\main.rs:8:62
  
8      ($seq: ident[$ind: ident]: $sty: ty = $($inits: expr), + ... $recur: expr) => { {
                                                                ^^^ not allowed after `expr` fragments
  
  = note: allowed there are: `=>`, `,` or `;`
```

看来是不能用`...`了，那我们换成`=>`

#### 完整代码

```rust
macro_rules! count_exprs {
    () => { 0 };
    ($head: expr) => { 1 };
    ($head: expr, $($tail: expr), *) => { 1 + count_exprs!($($tail), *) }
}

macro_rules! recurrence {
    ($seq: ident[$ind: ident]: $sty: ty = $($inits: expr), + => $recur: expr) => { {
        use std::ops::Index;
        const MEM_SIZE: usize = count_exprs!($($inits: expr), +);

        struct Solution {
            mem: [$sty; MEM_SIZE],
            pos: usize,
        }

        struct IndexOffset<'a> {
            slice: &'a [$sty; MEM_SIZE],
            offset: usize,
        }

        impl<'a> Index<usize> for IndexOffset<'a> {
            type Output = $sty;

            #[inline(always)]
            fn index<'b>(&'b self, index: usize) -> &'b Self::Output {
                use std::num::Wrapping;
                let index = Wrapping(index);
                let offset = Wrapping(self.offset);
                let window = Wrapping(MEM_SIZE);
                let real_index = index - offset + window;
                &self.slice[real_index.0]
            }
        }

        impl Iterator for Solution {
            type Item = $sty;

            #[inline]
            fn next(&mut self) -> Option<Self::Item> {
                if self.pos < MEM_SIZE {
                    let value = self.mem[self.pos];
                    self.pos += 1;
                    Some(value)
                } else {
                    let next_val = {
                        #[allow(unused)]
                        let $ind = self.pos;
                        #[allow(unused)]
                        let $seq = IndexOffset { slice: &self.mem, offset: $ind };
                        ($recur)
                    };

                    {
                        use std::mem::swap;
                        let mut swap_tmp = next_val;
                        for i in (0..MEM_SIZE).rev() {
                            swap(&mut swap_tmp, &mut self.mem[i]);
                        }
                    }

                    self.pos += 1;
                    Some(next_val)
                }
            }
        }

        Solution { mem: [$($inits), +], pos: 0 }
    } }
}

fn main() {
    let s1 = recurrence!(s1[n]: usize = 0, 1 => n);
    println!("s1.take(6) is");
    for i in s1.take(6) {
        println!("{}", i);
    }

    println!("-------------");

    let s2 = recurrence!(s2[n]: f64 = 1.0 => s2[n - 1] * n as f64);
    let sum = s2.take(3).fold(0.0, acc, x acc + x);
    println!("s2.take(3)'s sum is {}", sum);

    println!("-------------");

    let s3 = recurrence!(s3[n]: usize = 1 => n + 1);
    let v = s3.take(5).collect::<Vec<usize>>();
    println!("s3.take(5) is {:?}", v);

    println!("-------------");

    let mut fib = recurrence!(fib[n]: u32 = 0, 1 => fib[n - 1] + fib[n - 2]);
    let sixth = fib.nth(6).unwrap();
    println!("the sixth of fibonacci is {}", sixth);

    println!("-------------");

    println!("RUST NB!");
}
```

输出：

```rust
s1.take(6) is
0
1
2
3
4
5
-------------
s2.take(3)'s sum is 4
-------------
s3.take(5) is [1, 2, 3, 4, 5]
-------------
the sixth of fibonacci is 8
-------------
RUST NB!
```

### 引用文章

*   [The Little Book of Rust Macros](https://danielkeep.github.io/tlborm/)
    
*   [The Rust Programming Language](https://doc.rust-lang.org/book/title-page.html)