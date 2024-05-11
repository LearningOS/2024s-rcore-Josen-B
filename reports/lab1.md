# 1.编码作业功能描述

1. 在task.rs中的TaskControlBlock结构体中新增了一个sys_call_times的数组，用于记录当前任务中各个系统调用的次数。
2. 每次执行系统调用时，会自动将全局变量TASK_MANAGER中当前任务current_task对应的TaskControlBlock结构体的相应系统调用计数自增。
3. TaskManager定义了get_sys_call_times方法，用于获取当前任务current_task对应的TaskControlBlock结构体的系统调用数组的拷贝。
4. 在完成process.rs中的sys_task_info函数时，调用了get_sys_call_times和get_time_ms方法，以获取TaskInfo结构体的syscall_times和time部分，并将status部分设为Running。

# 2 简答作业

# 2.1 正确进入 U 态后，程序的特征还应有：使用 S 态特权指令，访问 S 态寄存器后会报错。 

请同学们可以自行测试这些内容 (运行 Rust 三个 bad 测例 (ch2b_bad_*.rs) ， 注意在编译时至少需要指定 LOG=ERROR 才能观察到内核的报错信息) ， 描述程序出错行为，同时注意注明你使用的 sbi 及其版本。

Rustsbi 版本为: 0.2.0-alpha.2

出现以下报错: 
[kernel] PageFault in application, bad addr = 0x0, bad instruction = 0x804003c4, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.
[kernel] IllegalInstruction in application, kernel killed it.

ch2b_bad_address.rs 由于除0错误触发异常退出
ch2b_bad_instructions.rs 在用户态非法使用指令sretch2b_bad_register.rs 在用户态非法使用指令csrr

# 2.2 深入理解 trap.S 中两个函数 __alltraps 和 __restore 的作用，并回答如下问题:

# Q1: L40：刚进入 __restore 时，a0 代表了什么值。请指出 __restore 的两种使用情景。

A1: 刚进入 __restore 时，a0 代表了系统调用的第一个参数

__restore 的两种使用情景：

1. 从用户态进入内核态
2. 从内核态返回用户态

# Q2 L43-L48：这几行汇编代码特殊处理了哪些寄存器？这些寄存器的的值对于进入用户态有何意义？请分别解释。

ld t0, 32*8(sp) # 内核栈 32*8(sp) 处存储了原 sstatus 寄存器的值, 将其读取到 t0
ld t1, 33*8(sp) # 内核栈 32*8(sp) 处存储了原 sepc 寄存器的值, 将其读取到 t1
ld t2, 2*8(sp) # 内核栈 32*8(sp) 处存储了原 sscratch 寄存器的值, 将其读取到 t2
csrw sstatus, t0 # 将 t0中原 sstatus 寄存器的值读取到 sstatus
csrw sepc, t1 # 将 t0中原 sepc 寄存器的值读取到 sepc
csrw sscratch, t2 # 将 t0中原 sscratch 寄存器的值读取到 sscratch

# 2.3 L50-L56：为何跳过了 x2 和 x4？

1. 跳过`x2`是因为`x2`对应的用户栈指针保存到了sscratch寄存器, 不需要从内核栈中进行恢复
2. 跳过`x4`是因为并没有使用它, 所以无需恢复

# 2.4 L60：该指令之后，sp 和 sscratch 中的值分别有什么意义？

`sp`指向用户栈, `sscratch`指向内核栈

# 2.5 __restore：中发生状态切换在哪一条指令？为何该指令执行之后会进入用户态？

`sret`后发生了状态切换, 执行该指令后, PC设置为 `sepc` 寄存器的值。
`sepc` 存储着产生中断或异常前的指令地址，因此这实现了到原始代码的返回。

# 2.6 L13：该指令之后，sp 和 sscratch 中的值分别有什么意义？

`sp`, `sscratch`寄存器的内容被交换, `sp`保存了原`sscratch`中的内核栈指针, `sscratch`保存了原`sp`中的用户栈栈指针

# 2.7 从 U 态进入 S 态是哪一条指令发生的？

在`ecall`指令时发生