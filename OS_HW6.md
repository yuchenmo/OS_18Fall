# lec6 SPOC思考题

## 与视频相关思考题

### 6.1	非连续内存分配的需求背景
1. 为什么要设计非连续内存分配机制？

存在要用户需要一块区域，而内存中没有满足大小需求的连续区域这种情况需求。

2. 非连续内存分配中内存分块大小有哪些可能的选择？大小与大小是否可变?

段式（较大），页式（较小），段页式；分配的内存大小可以改变。

3. 为什么在大块时要设计大小可变，而在小块时要设计成固定大小？小块时的固定大小可以提供多种选择吗？



### 6.2	段式存储管理
 1. 什么是段、段基址和段内偏移？

 数据类型和访问方式相同的一段连续的地址空间；
 段在物理内存中的起始地址；
 访问的内容相对基址的位置；

 1. 段式存储管理机制的地址转换流程是什么？为什么在段式存储管理中，各段的存储位置可以不连续？这种做法有什么好处和麻烦？

给定逻辑地址（段号、偏移），根据段号在进程的段表中查段描述符，在描述符中获得段基址和段长度，MMU根据长度和偏移判读是否越界，再将基址加上偏移得到物理地址进行访问；
因为段的定义需要访问方式相同，且不会通过一个段（基址）访问另一个段；
管理内存更加灵活，内存映射略显麻烦。

### 6.3	页式存储管理
 1. 什么是页（page）、帧（frame）、页表（page table）、存储管理单元（MMU）、快表（TLB, Translation Lookaside Buffer）和高速缓存（cache）？

 页：逻辑页面
 帧：物理页帧
 页表：逻辑和物理页号转换的LUT
 MMU：进行物理和逻辑地址转换的部件
 TLB：页表的Cache
 Cache：缓存，提升访存速度
 
 1. 页式存储管理机制的地址转换流程是什么？为什么在页式存储管理中，各页的存储位置可以不连续？这种做法有什么好处和麻烦？

 给定逻辑地址（页号和页内偏移），根据页号转换物理地址+页内偏移；其逻辑中页表的映射不一定是连续的，因而该存储管理允许不连续；
 好处依然是灵活和方便内存管理，麻烦是内存映射较为复杂，开销较大。

### 6.4	页表概述
 1. 每个页表项有些什么内容？有哪些标志位？它们起什么作用？

 标志位：存在位（是否有对应物理帧）、修改位（是否被修改过，涉及Cache的读写）、引用位（过去一段时间是否有过对它的引用，涉及Cache的更新）
 页号、帧号
 
 1. 页表大小受哪些因素影响？
物理地址空间大小、页大小、进程数

### 6.5	快表和多级页表
 1. 快表（TLB）与高速缓存（cache）有什么不同？

  Cache缓存内存，TLB只缓存页表
 
 1. 为什么快表中查找物理地址的速度非常快？它是如何实现的？为什么它的的容量很小？

减少了访存次数；把近期访问的页表项缓存进CPU；在CPU里，成本高功耗大
 
 1. 什么是多级页表？多级页表中的地址转换流程是什么？多级页表有什么好处和麻烦？

页表组织成树状，页号分级，间接引用寻址；每级访问页表查找到下一级页表中的位置，在最后一级找到物理页号；访存次数太多，但由于不会用到所有的页表，事实上可以节省页表大小。

### 6.6	反置页表
 1. 页寄存器机制的地址转换流程是什么？

逻辑地址Hash，（TLB，）页寄存器，解决冲突
 
 1. 反置页表机制的地址转换流程是什么？

逻辑地址和PID一起Hash，找页寄存器，解决冲突
 
 1. 反置页表项有些什么内容？

PID、逻辑地址、标志位

### 6.7	段页式存储管理
 1. 段页式存储管理机制的地址转换流程是什么？这种做法有什么好处和麻烦？

给定段号、页号和页内偏移，进程段基址->段表基址，段表基址+段号->段表项（段长度、段基址），段基址->页表基址，页表基址+页号->页表项（页帧号），页帧号+页内偏移->物理地址；
对段/页式的cpu兼容性好，其它到处都是麻烦。
 
 1. 如何实现基于段式存储管理的内存共享？

 两个进程的段表项指向同一块区域
 
 1. 如何实现基于页式存储管理的内存共享？

 两个进程的页表项指向同一块区域

## 个人思考题
（1） (w3l2) 请简要分析64bit CPU体系结构下的分页机制是如何实现的



## 小组思考题
（1）(spoc) 某系统使用请求分页存储管理，若页在内存中，满足一个内存请求需要150ns (10^-9s)。若缺页率是10%，为使有效访问时间达到0.5us(10^-6s),求不在内存的页面的平均访问时间。请给出计算步骤。

	90% * 150ns + 10% * x = 500
	x = 3650ns

（2）(spoc) 有一台假想的计算机，页大小（page size）为32 Bytes，支持32KB的虚拟地址空间（virtual address space）,有4KB的物理内存空间（physical memory），采用二级页表，一个页目录项（page directory entry ，PDE）大小为1 Byte,一个页表项（page-table entries
PTEs）大小为1 Byte，1个页目录表大小为32 Bytes，1个页表大小为32 Bytes。页目录基址寄存器（page directory base register，PDBR）保存了页目录表的物理地址（按页对齐）。

PTE格式（8 bit） :
```
  VALID | PFN6 ... PFN0
```
PDE格式（8 bit） :
```
  VALID | PT6 ... PT0
```
其
```
VALID==1表示，表示映射存在；VALID==0表示，表示映射不存在。
PFN6..0:页帧号
PT6..0:页表的物理基址>>5
```
在[物理内存模拟数据文件](./03-2-spoc-testdata.md)中，给出了4KB物理内存空间的值，请回答下列虚地址是否有合法对应的物理内存，请给出对应的pde index, pde contents, pte index, pte contents。
```
1) Virtual Address 6c74
   Virtual Address 6b22
2) Virtual Address 03df
   Virtual Address 69dc
3) Virtual Address 317a
   Virtual Address 4546
4) Virtual Address 2c03
   Virtual Address 7fd7
5) Virtual Address 390e
   Virtual Address 748b
```

比如答案可以如下表示： (注意：下面的结果是错的，你需要关注的是如何表示)
```
Virtual Address 7570:
  --> pde index:0x1d  pde contents:(valid 1, pfn 0x33)
    --> pte index:0xb  pte contents:(valid 0, pfn 0x7f)
      --> Fault (page table entry not valid)

Virtual Address 21e1:
  --> pde index:0x8  pde contents:(valid 0, pfn 0x7f)
      --> Fault (page directory entry not valid)

Virtual Address 7268:
  --> pde index:0x1c  pde contents:(valid 1, pfn 0x5e)
    --> pte index:0x13  pte contents:(valid 1, pfn 0x65)
      --> Translates to Physical Address 0xca8 --> Value: 16
```

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，请说明原因。

虚拟地址17位，pde号5位，pte号5位。pte在page0x11。一个page 32Bytes = 0x20 Bytes

0x6c74(110 1100 0111 0100): pde index = 0x1b，pde content = 0xa0（10100000，valid @ pt = 0x20），pte index = 0x3，pte content = 0xe1（11100001, valid @ pfn = 0x61，offset=0x14），physical address = pfn << 5 + offset = 0xc34，value = 0x06

（3）请基于你对原理课二级页表的理解，并参考Lab2建页表的过程，设计一个应用程序（可基于python、ruby、C、C++、LISP、JavaScript等）可模拟实现(2)题中描述的抽象OS，可正确完成二级页表转换。

[链接](https://piazza.com/class/i5j09fnsl7k5x0?cid=664)有上面链接的参考答案。请比较你的结果与参考答案是否一致。如果不一致，提交你的实现，并说明区别。

（4）假设你有一台支持[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)的机器，请问你如何设计操作系统支持这种类型计算机？请给出设计方案。

 (5)[X86的页面结构](http://os.cs.tsinghua.edu.cn/oscourse/OS2018spring/lecture06)
---

## 扩展思考题

阅读64bit IBM Powerpc CPU架构是如何实现[反置页表](http://en.wikipedia.org/wiki/Page_table#Inverted_page_table)，给出分析报告。


## interactive　understand VM

[Virtual Memory with 256 Bytes of RAM](http://blog.robertelder.org/virtual-memory-with-256-bytes-of-ram/)：这是一个只有256字节内存的一个极小计算机系统。按作者的[特征描述](https://github.com/RobertElderSoftware/recc#what-can-this-project-do)，它具备如下的功能。
 - CPU的实现代码不多于500行；
 - 支持14条指令、进程切换、虚拟存储和中断；
 - 用C实现了一个小的操作系统微内核可以在这个CPU上正常运行；
 - 实现了一个ANSI C89编译器，可生成在该CPU上运行代码；
 - 该编译器支持链接功能；
 - 用C89, Python, Java, Javascript这4种语言实现了该CPU的模拟器；
 - 支持交叉编译；
 - 所有这些只依赖标准C库。
 
针对op-cpu的特征描述，请同学们通过代码阅读和执行对自己有兴趣的部分进行分析，给出你的分析结果和评价。