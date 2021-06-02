---
title: 写OS遇到的让人印象深刻的Bug（RISC-V）
date: 2021-02-23 23:26:22
tags:
categories:
  - - OS
---

**常在河边走，哪有不湿鞋**
<!-- more -->

#### 我来分享两个我在写OS过程中遇到的Bug

##### 折腾Trap遇到的玄学Bug

*众所周知，玄学Bug总归还是找得到原因的。* 

当Trap发生时，会进入`stvec`寄存器所存的地址的位置（Direct Mode），这个位置一般是Trap处理函数。我将我的测试函数写入`stvec`寄存器，然后通过`ebreak`指令产生中断，发现测试正常，之后就把无用的`println!`删除了。不删不要紧一删就跑不起来了，当时给我郁闷的。

既然Bug出现了，那就得排除修复，第一个检查的就是地址有没有写入`stvec`，经过GDB调试，发现地址写入后随即变成0x0，但是加个`println!`又修复了，这是为啥呢？没有思路的我开始破罐子破摔，开始写入一些随机值（1, 2, 3, 4 .....），一写我就发现了，只有4的倍数的值可以被成功写入，接着我就想难道有对其要求？翻出riscv-privileged，查了查，发现这一句话：

*The BASE field in stvec is a WARL field that can hold any valid virtual or physical address,*
*subject to the following alignment constraints: the address **must always be at least 4-byte aligned**,*
*and the MODE setting may impose additional alignment constraints on the value in the BASE*
*field*

~~妈的，还是得好好学习。~~写入的地址必须要是4字节对齐的，所以那行`println!`引发的Bug只是影响了地址值，加上恰好是4个字节。我的解决办法是在`trap entry`前加入对齐指令：

```
# stvec need align 4 byte
.align 4

_trap_entry: 
......
......
```



##### 自以为写的很好的代码引发的不易发现的Bug

*众所周知，不易发现的问题代码常常是自己觉得写的不错的代码*

**RAII**，全称**资源获取即初始化**（英语：**R**esource **A**cquisition **I**s **I**nitialization）。RAII要求，资源的有效期与持有资源的[对象的生命期](https://zh.wikipedia.org/w/index.php?title=对象的生命期&action=edit&redlink=1)严格绑定，即由对象的[构造函数](https://zh.wikipedia.org/wiki/构造函数)完成[资源的分配](https://zh.wikipedia.org/w/index.php?title=资源的分配&action=edit&redlink=1)（获取），同时由[析构函数](https://zh.wikipedia.org/wiki/析构函数)完成资源的释放。在这种要求下，只要对象能正确地析构，就不会出现[资源泄露](https://zh.wikipedia.org/w/index.php?title=资源泄露&action=edit&redlink=1)问题。——[来自维基百科RAII](https://zh.wikipedia.org/wiki/RAII)

在写虚拟地址模块时，写了个```FrameTracker```结构，目的是记录每一个被分配的物理页号（PPN）,同时为其实现了`Drop Trait`，在其生命周期结束的时候会自动释放资源。

```rust
impl Drop for FrameTracker {
    fn drop(&mut self) {
        stack_frame_dealloc(self.ppn)
    }
}
```

当代码写的差不多时，我进行测试，发现直接`LoadPageFault`，发现在进行虚拟页转换的时候，二级页表指向的指针居然是某个虚拟地址的物理地址，以至于访问次虚拟地址的时候直接`LoadPageFault`，调试2天毫无头绪。就在刚刚才想到可能原本的地址被释放了，然后再次分配了`Frame`，这本不应该发生。顺着这思路马上就找到了Bug，而我记得在写这段代码时十分满意。。。

```rust
pub fn map_one(&mut self, vpn: VirtPageNum, page_table: &mut PageTable) {
    let ppn = match self.map_type {
        MapType::Identical => PhysPageNum(vpn.0),
        MapType::Framed => {
            let f = stack_frame_alloc().unwrap();
            self.data_frames.insert(vpn, f.clone());
            f.ppn
        }
    };

    let flags = PTEFlags::from_bits(self.map_permission.bits).unwrap();
    page_table.map(vpn, ppn, flags);
}
```

这里可以看到Bug发生的原因是：`FrameTracker`实现了`Drop Trait`，这是一种RAII，在其生命周期结束的时候会自动释放资源，我直接使用了`clone()`，而导致生命周期在函数执行完时结束，接着释放了资源，对应的`PPN`即被标记为未分配的状态。

修复自然简单，不Clone不就完了：

```rust
pub fn map_one(&mut self, vpn: VirtPageNum, page_table: &mut PageTable) {
    let ppn = match self.map_type {
        MapType::Identical => PhysPageNum(vpn.0),
        MapType::Framed => {
            let f = stack_frame_alloc().unwrap();
            let ppn = f.ppn;
            self.data_frames.insert(vpn, f);
            ppn
        }
    };

    let flags = PTEFlags::from_bits(self.map_permission.bits).unwrap();
    page_table.map(vpn, ppn, flags);
}
```



