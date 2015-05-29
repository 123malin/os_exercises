# lab6 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。


## 个人思考题

### 总体介绍

 - ucore中的调度点在哪里，完成了啥事？
 - 进程控制块中与调度相关的字段有哪些？
 - ucore的就绪队列数据结构在哪定义？在哪进行修改？
 - ucore的等待队列数据结构在哪定义？在哪进行修改？

### 调度算法支撑框架

 - 调度算法支撑框架中的各个函数指针的功能是啥？会被谁在何种情况下调用？
 - 调度函数schedule()的调用函数分析，可以了解进程调度的原因。请分析ucore中所有可能的调度位置，并说明可能的调用原因。
  
### 时间片轮转调度算法

 - 时间片轮转调度算法是如何基于调度算法支撑框架实现的？
 - 时钟中断如何调用RR_proc_tick()的？

### stride调度算法

 - stride调度算法的思路？ 
 - stride算法的特征是什么？
 - stride调度算法是如何避免stride溢出问题的？
 - 无符号数的有符号比较会产生什么效果？
 - 什么是斜堆(skew heap)?

## 小组练习与思考题

### (1)(spoc) 理解调度算法支撑框架的执行过程

即在ucore运行过程中通过`cprintf`函数来完整地展现出来多个进程在调度算法和框架的支撑下，在相关调度点如何动态调度和执行的细节。(越全面细致越好)

请完成如下练习，完成代码填写，并形成spoc练习报告
> 需写练习报告和简单编码，完成后放到git server 对应的git repo中
>>代码见labcodes_answer/lab6_result
>>输出大致如下：

```
+ cc kern/process/proc.c
+ ld bin/kernel
(THU.CST) os is loading ...

Special kernel symbols:
  entry  0xc010002a (phys)
  etext  0xc010d865 (phys)
  edata  0xc01b1dd4 (phys)
  end    0xc01b4f78 (phys)
Kernel executable memory footprint: 724KB
ebp:0xc012ff38 eip:0xc0101f67 args:0x00010094 0x00000000 0xc012ff68 0xc01000d8 
    kern/debug/kdebug.c:350: print_stackframe+21
ebp:0xc012ff48 eip:0xc0102256 args:0x00000000 0x00000000 0x00000000 0xc012ffb8 
    kern/debug/kmonitor.c:129: mon_backtrace+10
ebp:0xc012ff68 eip:0xc01000d8 args:0x00000000 0xc012ff90 0xffff0000 0xc012ff94 
    kern/init/init.c:58: grade_backtrace2+33
ebp:0xc012ff88 eip:0xc0100101 args:0x00000000 0xffff0000 0xc012ffb4 0x0000002a 
    kern/init/init.c:63: grade_backtrace1+38
ebp:0xc012ffa8 eip:0xc010011f args:0x00000000 0xc010002a 0xffff0000 0x0000001d 
    kern/init/init.c:68: grade_backtrace0+23
ebp:0xc012ffc8 eip:0xc0100144 args:0xc010d89c 0xc010d880 0x000031a4 0x00000000 
    kern/init/init.c:73: grade_backtrace+34
ebp:0xc012fff8 eip:0xc010007f args:0x00000000 0x00000000 0x0000ffff 0x40cf9a00 
    kern/init/init.c:32: kern_init+84
memory management: default_pmm_manager
e820map:
  memory: 0009fc00, [00000000, 0009fbff], type = 1.
  memory: 00000400, [0009fc00, 0009ffff], type = 2.
  memory: 00010000, [000f0000, 000fffff], type = 2.
  memory: 07efe000, [00100000, 07ffdfff], type = 1.
  memory: 00002000, [07ffe000, 07ffffff], type = 2.
  memory: 00040000, [fffc0000, ffffffff], type = 2.
check_alloc_page() succeeded!
check_pgdir() succeeded!
check_boot_pgdir() succeeded!
-------------------- BEGIN --------------------
PDE(0e0) c0000000-f8000000 38000000 urw
  |-- PTE(38000) c0000000-f8000000 38000000 -rw
PDE(001) fac00000-fb000000 00400000 -rw
  |-- PTE(000e0) faf00000-fafe0000 000e0000 urw
  |-- PTE(00001) fafeb000-fafec000 00001000 -rw
--------------------- END ---------------------
use SLOB allocator
check_slab() success
kmalloc_init() succeeded!
check_vma_struct() succeeded!
page fault at 0x00000100: K/W [no page found].
check_pgfault() succeeded!
check_vmm() succeeded.
sched class: stride_scheduler
push process 1 into wait queue!
ide 0:      10000(sectors), 'QEMU HARDDISK'.
ide 1:     262144(sectors), 'QEMU HARDDISK'.
SWAP: manager = fifo swap manager
BEGIN check_swap: count 31813, total 31813
setup Page Table for vaddr 0X1000, so alloc a page
setup Page Table vaddr 0~4MB OVER!
set up init env for check_swap begin!
page fault at 0x00001000: K/W [no page found].
page fault at 0x00002000: K/W [no page found].
page fault at 0x00003000: K/W [no page found].
page fault at 0x00004000: K/W [no page found].
set up init env for check_swap over!
write Virt Page c in fifo_check_swap
write Virt Page a in fifo_check_swap
write Virt Page d in fifo_check_swap
write Virt Page b in fifo_check_swap
write Virt Page e in fifo_check_swap
page fault at 0x00005000: K/W [no page found].
swap_out: i 0, store page in vaddr 0x1000 to disk swap entry 2
write Virt Page b in fifo_check_swap
write Virt Page a in fifo_check_swap
page fault at 0x00001000: K/W [no page found].
do pgfault: ptep c03b9004, pte 200
swap_out: i 0, store page in vaddr 0x2000 to disk swap entry 3
swap_in: load disk swap entry 2 with swap_page in vadr 0x1000
write Virt Page b in fifo_check_swap
page fault at 0x00002000: K/W [no page found].
do pgfault: ptep c03b9008, pte 300
swap_out: i 0, store page in vaddr 0x3000 to disk swap entry 4
swap_in: load disk swap entry 3 with swap_page in vadr 0x2000
write Virt Page c in fifo_check_swap
page fault at 0x00003000: K/W [no page found].
do pgfault: ptep c03b900c, pte 400
swap_out: i 0, store page in vaddr 0x4000 to disk swap entry 5
swap_in: load disk swap entry 4 with swap_page in vadr 0x3000
write Virt Page d in fifo_check_swap
page fault at 0x00004000: K/W [no page found].
do pgfault: ptep c03b9010, pte 500
swap_out: i 0, store page in vaddr 0x5000 to disk swap entry 6
swap_in: load disk swap entry 5 with swap_page in vadr 0x4000
count is 5, total is 5
check_swap() succeeded!
++ setup timer interrupts
pick up process 1 to run!
pop process 1 out of the queue!
push process 2 into wait queue!
pick up process 2 to run!
pop process 2 out of the queue!
kernel_execve: pid = 2, name = "priority".
push process 3 into wait queue!
push process 4 into wait queue!
push process 5 into wait queue!
push process 6 into wait queue!
push process 7 into wait queue!
main: fork ok,now need to wait pids.
pick up process 7 to run!
pop process 7 out of the queue!
child pid 7, acc 1108000, time 1001
push process 2 into wait queue!
pick up process 6 to run!
pop process 6 out of the queue!
child pid 6, acc 4000, time 1005
pick up process 5 to run!
pop process 5 out of the queue!
child pid 5, acc 4000, time 1010
pick up process 4 to run!
pop process 4 out of the queue!
child pid 4, acc 4000, time 1014
pick up process 3 to run!
pop process 3 out of the queue!
child pid 3, acc 4000, time 1018
pick up process 2 to run!
pop process 2 out of the queue!
main: pid 3, acc 4000, time 1019
main: pid 4, acc 4000, time 1020
main: pid 5, acc 4000, time 1020
main: pid 6, acc 4000, time 1020
main: pid 7, acc 1108000, time 1020
main: wait pids over
stride sched correct result: 1 1 1 1 277
push process 1 into wait queue!
pick up process 1 to run!
pop process 1 out of the queue!
push process 1 into wait queue!
pick up process 1 to run!
pop process 1 out of the queue!
all user-mode processes have quit.
init check memory pass.
kernel panic at kern/process/proc.c:460:
    initproc exit.

Welcome to the kernel debug monitor!!
Type 'help' for a list of commands.
K> 
```


### 练习用的[lab6 spoc exercise project source code](https://github.com/chyyuu/ucore_lab/tree/master/labcodes_answer/lab6_result)


