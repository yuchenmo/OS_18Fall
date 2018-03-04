# lab0 SPOC思考题

---
## 思考题

- 你理解的对于类似ucore这样需要进程/虚存/文件系统的操作系统，在硬件设计上至少需要有哪些直接的支持？至少应该提供哪些功能的特权指令？

    CPU应有MMU，硬盘应支持对应的文件系统等等；产生中断和使能/屏蔽中断。

- 你理解的x86的实模式和保护模式有什么区别？物理地址、线性地址、逻辑地址的含义分别是什么？
    
    保护模式下用户程序不能直接根据物理地址访存，而要根据虚拟地址经过操作系统映射到物理地址访问，避免了程序错误的修改内存（其他程序占用的内存，乃至程序代码自身所在的内存）。实模式仅仅将物理地址分段，对各个系统/用户程序所能占用的区域没有区分。因为这个本质的不同，实模式在CPU启动时使用，最多寻址1MB（16bits），而保护模式之后启动，可以寻址4GB

    逻辑地址：程序产生和使用的地址，是相对段基址的偏移
    线性地址：逻辑地址加上段偏移
    物理地址：地址总线上寻址物理内存的信号，通过线性地址查页表的得到

- 理解list_entry双向链表数据结构及其4个基本操作函数和ucore中一些基于它的代码实现（此题不用填写内容）

- 对于如下的代码段，请说明":"后面的数字是什么含义
```
 /* Gate descriptors for interrupts and traps */
 struct gatedesc {
    unsigned gd_off_15_0 : 16;        // low 16 bits of offset in segment
    unsigned gd_ss : 16;            // segment selector
    unsigned gd_args : 5;            // # args, 0 for interrupt/trap gates
    unsigned gd_rsv1 : 3;            // reserved(should be zero I guess)
    unsigned gd_type : 4;            // type(STS_{TG,IG32,TG32})
    unsigned gd_s : 1;                // must be 0 (system)
    unsigned gd_dpl : 2;            // descriptor(meaning new) privilege level
    unsigned gd_p : 1;                // Present
    unsigned gd_off_31_16 : 16;        // high bits of offset in segment
 };
```

    “：”后为C语言中的位域，在这里可以指定结构体的每个成员占用多少bits空间


- 对于如下的代码段，

```
#define SETGATE(gate, istrap, sel, off, dpl) {            \
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;        \
    (gate).gd_ss = (sel);                                \
    (gate).gd_args = 0;                                    \
    (gate).gd_rsv1 = 0;                                    \
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;    \
    (gate).gd_s = 0;                                    \
    (gate).gd_dpl = (dpl);                                \
    (gate).gd_p = 1;                                    \
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;        \
}
```
如果在其他代码段中有如下语句，
```
unsigned intr;
intr=8;
SETGATE(intr, 1,2,3,0);
```
请问执行上述指令后， intr的值是多少？


假设有SETGATE前有强转类型，亦即gate是一个起始地址和intr一样的gatedesc
在github上找到的STS_IG32=0xE, STS_TG32=0xF
初始gate: 0x???? 0x???? 0x???? 0x0008

```

#define SETGATE(gate, istrap, sel, off, dpl) {           // istrap = 0, sel = 1, off = 2, dpl = 3
    (gate).gd_off_15_0 = (uint32_t)(off) & 0xffff;       // 0x???? 0x???? 0x???? 0x0002 
    (gate).gd_ss = (sel);                                // 0x???? 0x???? 0x0001 0x0002
    (gate).gd_args = 0;                                  // 0x???? 0x??t0 0x0001 0x0002 t最后一位为0
    (gate).gd_rsv1 = 0;                                  // 0x???? 0x??00 0x0001 0x0002
    (gate).gd_type = (istrap) ? STS_TG32 : STS_IG32;     // 0x???? 0x?E00 0x0001 0x0002
    (gate).gd_s = 0;                                     // 0x???? 0xtE00 0x0001 0x0002 t最后一位为0
    (gate).gd_dpl = (dpl);                               // 0x???? 0xrE00 0x0001 0x0002 r最后三位为110
    (gate).gd_p = 1;                                     // 0x???? 0xEE00 0x0001 0x0002
    (gate).gd_off_31_16 = (uint32_t)(off) >> 16;         // 0x0000 0xEE00 0x0001 0x0002
}
```
因此，intr（即*(&intr)）值为0x10002


### 课堂实践练习

#### 练习一

请在ucore中找一段你认为难度适当的AT&T格式X86汇编代码，尝试解释其含义。

  - [Intel格式和AT&T格式汇编区别](http://www.cnblogs.com/hdk1993/p/4820353.html)

  - ##### [x86汇编指令集  ](http://hiyyp1234.blog.163.com/blog/static/67786373200981811422948/)

  - ##### [PC Assembly Language, Paul A. Carter, November 2003.](https://pdos.csail.mit.edu/6.828/2016/readings/pcasm-book.pdf)

  - ##### [*Intel 80386 Programmer's Reference Manual*, 1987](https://pdos.csail.mit.edu/6.828/2016/readings/i386/toc.htm)

  - ##### [[IA-32 Intel Architecture Software Developer's Manuals](http://www.intel.com/content/www/us/en/processors/architectures-software-developer-manuals.html)]

#### 练习二

宏定义和引用在内核代码中很常用。请枚举ucore中宏定义的用途，并举例描述其含义。

 > 利用宏进行复杂数据结构中的数据访问；
    #define WT_CHILD                    (0x00000001 | WT_INTERRUPTED)  // wait child process
    #define WT_KSEM                      0x00000100                    // wait kernel semaphore 
    #define WT_TIMER                    (0x00000002 | WT_INTERRUPTED)  // wait timer
    #define WT_KBD                      (0x00000004 | WT_INTERRUPTED)  // wait the input of keyboard
 > 利用宏进行数据类型转换；如 to_struct, 
    #define le2vma(le, member) to_struct((le), struct vma_struct, member)
 > 常用功能的代码片段优化；如  ROUNDDOWN, SetPageDirty
    #define pid_hashfn(x)       (hash32(x, HASH_SHIFT))
    #define ALIGN(addr,size)   (((addr)+(size)-1)&(~((size)-1))) 
