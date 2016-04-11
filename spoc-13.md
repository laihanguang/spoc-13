# lab4 spoc 思考题

- 有"spoc"标记的题是要求拿清华学分的同学要在实体课上完成，并按时提交到学生对应的ucore_code和os_exercises的git repo上。

## 个人思考题

### 13.1 总体介绍

(1) ucore的线程控制块数据结构是什么？

    struct proc_struct {
    enum proc_state state;                      // Process state
    int pid;                                    // Process ID
    int runs;                                   // the running times of Proces
    uintptr_t kstack;                           // Process kernel stack
    volatile bool need_resched;                 // bool value: need to be rescheduled to release CPU?
    struct proc_struct *parent;                 // the parent process
    struct mm_struct *mm;                       // Process's memory management field
    struct context context;                     // Switch here to run process
    struct trapframe *tf;                       // Trap frame for current interrupt
    uintptr_t cr3;                              // CR3 register: the base addr of Page Directroy Table(PDT)
    uint32_t flags;                             // Process flag
    char name[PROC_NAME_LEN + 1];               // Process name
    list_entry_t list_link;                     // Process link list 
    list_entry_t hash_link;                     // Process hash list
    };

### 13.2 关键数据结构

(2) 如何知道ucore的两个线程同在一个进程？

根据proc_struct中的parent是否为同一个parent来判断

(3) context和trapframe分别在什么时候用到？

context：线程切换时用到；
trapframe：中断/异常/系统调用时用到。

(4) 用户态或内核态下的中断处理有什么区别？在trapframe中有什么体现？

在用户态时发生中断，trapframe中还需要额外保存cs/esp/ss，以完成特权级切换。

### 13.3 执行流程

(5) do_fork中的内核线程执行的第一条指令是什么？它是如何过渡到内核线程对应的函数的？
```
tf.tf_eip = (uint32_t) kernel_thread_entry;
/kern-ucore/arch/i386/init/entry.S
/kern/process/entry.S
```

    pushl %edx              # push arg
    call *%ebx              # call fn
    pushl %eax              # save the return value of fn(arg)
    call do_exit            # call do_exit to terminate current thread
    
通过`call *%ebx`来调用内核线程对应的函数fn

(6)内核线程的堆栈初始化在哪？

```
tf和context中的esp
```

在`do_fork`函数中调用了`copy_thread`会完成tf和context的初始化

(7)fork()父子进程的返回值是不同的。这在源代码中的体现中哪？

父进程的返回值在do_fork()中，设置返回值为子进程的pid`ret = proc->pid`

子进程的返回值在`copy_thread()`中设置寄存器`eax`的值为0

(8)内核线程initproc的第一次执行流程是什么样的？能跟踪出来吗？

在kernel_thread_entry中 call *%ebx 调用了fn，打印字符串信息
