---
title: T and 'static
tags: []
id: '794'
categories:
  - - Rust
date: 2021-01-24 21:42:41
---


# T: 'static

## 先看几个误区(直接摘抄自[Rust生命周期常见误区](https://github.com/whfuyn/rust-blog/blob/master/posts/Rust%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F%E7%9A%84%E5%B8%B8%E8%A7%81%E8%AF%AF%E8%A7%A3.md))

### 1) `T` 只包含所有权类型

这个误解比起说生命周期，它和泛型更相关，但在Rust中泛型和生命周期是紧密联系在一起的，不可只谈其一。

当我刚开始学习Rust的时候，我理解`i32`，`&i32`，和`&mut i32`是不同的类型，也明白泛型变量`T`代表着所有可能类型的集合。
但尽管这二者分开都懂，当它们结合在一起的时候我却陷入困惑。在我这个Rust初学者的眼中，泛型是这样的运作的：

| | | | |
|-|-|-|-|
| **类型变量** | `T` | `&T` | `&mut T` |
| **例子** | `i32` | `&i32` | `&mut i32` |

`T` 包含一切所有权类型； `&T` 包含一切不可变借用类型； `&mut T` 包含一切可变借用类型。
`T`， `&T`， 和 `&mut T` 是不相交的有限集。 简洁明了，符合直觉，但却完全错误。
下面这才是泛型真正的运作方式：

| | | | |
|-|-|-|-|
| **类型变量** | `T` | `&T` | `&mut T` |
| **例子** | `i32`, `&i32`, `&mut i32`, `&&i32`, `&mut &mut i32`, ... | `&i32`, `&&i32`, `&&mut i32`, ... | `&mut i32`, `&mut &mut i32`, `&mut &i32`, ... |

`T`, `&T`, 和 `&mut T` 都是无限集, 因为你可以无限借用一个类型。
`T` 是 `&T` 和 `&mut T`的超集. `&T` 和 `&mut T` 是不相交的集合。
让我们用几个例子来检验一下这些概念:

```rust
trait Trait {}

impl<T> Trait for T {}

impl<T> Trait for &T {} // 编译错误

impl<T> Trait for &mut T {} // 编译错误
```

上面的代码并不能如愿编译:

```rust
error[E0119]: conflicting implementations of trait `Trait` for type `&_`:
 --> src/lib.rs:5:1
  |
3 | impl<T> Trait for T {}
  | ------------------- first implementation here
4 |
5 | impl<T> Trait for &T {}
  | ^^^^^^^^^^^^^^^^^^^^ conflicting implementation for `&_`

error[E0119]: conflicting implementations of trait `Trait` for type `&mut _`:
 --> src/lib.rs:7:1
  |
3 | impl<T> Trait for T {}
  | ------------------- first implementation here
...
7 | impl<T> Trait for &mut T {}
  | ^^^^^^^^^^^^^^^^^^^^^^^^ conflicting implementation for `&mut _`
```

编译器不允许我们为`&T`和`&mut T`实现`Trait`，因为这样会与为`T`实现的`Trait`冲突，
`T`本身已经包含了所有`&T`和`&mut T`。下面的代码能够如愿编译，因为`&T`和`&mut T`是不相交的：

```rust
trait Trait {}

impl<T> Trait for &T {} // 编译通过

impl<T> Trait for &mut T {} // 编译通过
```

**要点**
- `T` 是 `&T` 和 `&mut T`的超集
- `&T` 和 `&mut T` 是不相交的集合


### 2) 如果 `T: 'static` 那么 `T` 必须在整个程序运行中都是有效的

**误解推论**
- `T: 'static` 应该被看作 _" `T` 拥有 `'static` 生命周期 "_
- `&'static T` 和 `T: 'static` 没有区别
- 如果 `T: 'static` 那么 `T` 必须为不可变的
- 如果 `T: 'static` 那么 `T` 只能在编译期创建

大部分Rust初学者是从类似下面这个代码示例中接触到 `'static` 生命周期的：

```rust
fn main() {
    let str_literal: &'static str = "str literal";
}
```

他们被告知 `"str literal"` 是硬编码在编译出来的二进制文件中的，
并会在运行时被加载到只读内存，所以必须是不可变的且在整个程序的运行中都是有效的，
这就是它成为 `'static` 的原因。
而这些观念又进一步被用 `static` 关键字来定义静态变量的规则所加强。

```rust
static BYTES: [u8; 3] = [1, 2, 3];
static mut MUT_BYTES: [u8; 3] = [1, 2, 3];

fn main() {
   MUT_BYTES[0] = 99; // 编译错误，修改静态变量是unsafe的

    unsafe {
        MUT_BYTES[0] = 99;
        assert_eq!(99, MUT_BYTES[0]);
    }
}
```

认为静态变量
- 只可以在编译期创建
- 必须是不可变的，修改它们是unsafe的
- 在整个程序的运行过程中都是有效的

`'static` 生命周期大概是以静态变量的默认生命周期命名的，对吧？
那么有理由认为`'static`生命周期也应该遵守相同的规则，不是吗？

是的，但拥有`'static`生命周期的类型与`'static`约束的类型是不同的。
后者能在运行时动态分配，可以安全地、自由地修改，可以被drop，
还可以有任意长度的生命周期。

在这个点，很重要的是要区分 `&'static T` 和 `T: 'static`。

`&'static T`是对某个`T`的不可变引用，这个引用可以被无限期地持有直到程序结束。
这只可能发生在`T`本身不可变且不会在引用被创建后移动的情况下。
`T`并不需要在编译期就被创建，因为我们可以在运行时动态生成随机数据，
然后以内存泄漏为代价返回`'static`引用，例如：


```rust
use rand;

// 在运行时生成随机&'static str
fn rand_str_generator() -> &'static str {
    let rand_string = rand::random::<u64>().to_string();
    Box::leak(rand_string.into_boxed_str())
}
```

`T: 'static` 是指`T`可以被无限期安全地持有直到程序结束。
`T: 'static`包括所有`&'static T`，此外还包括所有的所有权类型，比如`String`, `Vec`等。
数据的所有者能够保证数据只要还被持有就不会失效，因此所有者可以无限期安全地持有该数据直到程序结束。
`T: 'static`应该被看作“`T`受`'static`生命周期约束”而非“`T`有着`'static`生命周期”。
这段代码能帮我们阐释这些概念：


```rust
use rand;

fn drop_static<T: 'static>(t: T) {
    std::mem::drop(t);
}

fn main() {
    let mut strings: Vec<String> = Vec::new();
    for _ in 0..10 {
        if rand::random() {
            // 所有字符串都是随机生成的
            // 并且是在运行时动态申请的
            let string = rand::random::<u64>().to_string();
            strings.push(string);
        }
    }

    // 这些字符串都是所有权类型，所以它们满足'static约束
    for mut string in strings {
        // 这些字符串都是可以修改的
        string.push_str("a mutation");
        // 这些字符串都是可以被drop的
        drop_static(string); // 编译通过
    }
    
    // 这些字符串都在程序结束之前失效
    println!("i am the end of the program");
}
```

**要点**
- `T: 'static` 应该被看作 _`T`受`'static`生命周期约束”_
- 如果 `T: 'static` 那么`T`可以是有着`'static`生命周期的借用类型
- 由于 `T: 'static` 包括了所有权类型，这意味着`T`
    - 可以在运行时动态分配
    - 不一定要在整个程序的运行过程中都有效
    - 可以被安全地、自由地修改
    - 可以在运行时被动态drop掉
    - 可以有不同长度的生命周期



## 我对T: ’static的认识

上文中提到了对`T`， `&T`， `&mut T`和`&'static T`，`T: 'static`的误区，其中有这么一句话：*`T: 'static`包括所有`&'static T`，此外还包括所有的所有权类型，比如`String`, `Vec`等。* 所有权类型可以理解成不包含引用的类型，我们来试试是不是这样：

```rust
use std::fmt::Debug;

fn static_test<T: 'static + Debug>(t: T) {
    println!("{:?}", t);
}

fn main() {
    let s_str: &'static str = "t: 'static test";
    let box_i32 = Box::new(1);
    static_test(s_str);
    static_test(box_i32);
}
```

毫无疑问，编译通过了，`s_str`是`&'static`类型，`box_i32`是所有权类型。接下来我们稍微改改：

```rust
use std::fmt::Debug;

fn static_test<T: 'static + Debug>(t: T) {
    println!("{:?}", t);
}

fn main() {
    let box_i32 = Box::new(1);
    static_test(&box_i32);
}
```

```rust_errors
error[E0597]: `box_i32` does not live long enough
  --> src/main.rs:9:17
   |
9  |     static_test(&box_i32);
   |     ------------^^^^^^^^-
   |     |           |
   |     |           borrowed value does not live long enough
   |     argument requires that `box_i32` is borrowed for `'static`
10 | }
   | - `box_i32` dropped here while still borrowed
```

报错了，不是说T是各种类型的集合吗，为什么不行呢？我们再改改：

```rust
use std::fmt::Debug;

fn static_test<T: 'static + Debug>(t: &T) {
    println!("{:?}", t);
}

fn main() {
    let box_i32 = Box::new(1);
    static_test(&box_i32);
}
```

编译通过了，通过思考，恍然大悟，Rust到处存在模式匹配：

```
{
	t: &T,
	&T: &Box<i32>,
	T: Box<i32>
}
```

这就出现有意思的局面了，在`T: 'static`限定下，除了`&'static T`，其类型为，直接迎合了我们`T` 只包含所有权类型的误区，其例子为：

| | | | |
|-|-|-|-|
| **类型变量** | `T` | `&T` | `&mut T` |
| **例子** | `i32` | `&i32`,  `&mut i32` | `&mut i32` |

真有意思

