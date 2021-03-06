### 进程概述

- 进程是正在运行的程序的实例，是基本的分配单元也是基本的执行单元。
- 可以用一个程序来创建多个进程，进程是由内核定义的抽象实体，并为该实体分配用以执行程序的各项系统资源。
- 单道程序：计算机内存中只允许一个程序运行
- 多道程序：计算机内存中同时放几道相互独立的程序，使它们在调度下，相互穿插运行。

- 时间片：操作系统分配给每个正在运行的进程微观上的一段时间。
- 并发：指在同一时刻，有多条指令在多个处理器上同时执行。
- 指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。
- 内核为每个进程分配一个 `PCB` 进程控制块，维护进程相关的信息，`Linux` 内核的进程控制块是 `task_struct` 结构体（`/usr/src/linux-headers-xxx/include/linux/sched.h` 中）

### 进程状态转换

- [新建态]、运行态、就绪态、阻塞态、[终止态]
- 查看进程 `ps aux / ajx`
  - a：显示终端上的所有进程，包括其他用户的进程
  - u：显示进程的详细信息
  - x：显示没有控制终端的进程
  - j：列出与作业控制相关的信息
- STAT 参数的意义：
  - D：不可中断
  - R：正在运行或在队列中的进程
  - S：处于休眠状态
  - T：停止或被追踪
  - Z：僵尸进程
  - W：进入内存交换
  - X：死掉的进程
  - <：高优先级
  - N：低优先级
  - S：包含子进程
  - +：位于前台的进程组
- 实时显示进程动态 `top [-d 5]`
  - -d 来指定信息更新的时间间隔
  - M 根据内存使用量排序
  - P 根据 CPU 占有率排序
  - T 根据进程运行时间排序
  - U 根据用户名排序
  - K 输入指定的 PID 杀死进程
- 杀死进程 `kill [-signal] pid`
  - `kill -l` 列出所有信号
  - `kill –SIGKILL pid`
  - `kill -9 pid`
  - `killall name xxx` 根据进程名杀死进程
- 进程号和相关函数
  - 每个进程都对应唯一一个进程号，类型为 `pid`
  - 任何进程（除 `init` 进程）都由一个进程创建，该进程称为被创建进程的父进程 `PPID`
  - 进程组是一个或多个进程的集合，有个进程组号，为 `PGID` 。
  - `pid_t getpid(void)` 和 `pid_t getppid(void)` 和 `pid_t getpgid(pid_t pid)`   

### 创建进程

```c
pid_t fork(void);
// 创建一个和父进程一模一样的子进程，子进程从这行开始执行
// 子进程返回 0，父进程返回子进程的 pid，失败返回 -1
```

完全的复制操作，可以认为只有 `pid` 不同， 相互之间几乎没有联系。（准确来说，`Linux` 的 `fork()` 使用的是写时拷贝，`fork` 时并不复制整个地址空间，而是共享，只有当需要写入时才会复制。这样效率高）

### 父子进程关系和 GDB 调试

**父子进程关系**

- `fork` 函数的返回值不同
- `PCB` 中的进程号和信号集等不同
- 数据读时共享，写时拷贝

**GDB 调试**

- 设置调试子进程，`set follow-fork-mode child` ，默认是父进程
- 设置调试模式，`set detch-on-fork on/off`，默认为 `on` ，表示调试当前进程时其他进程继续运行；否则，挂起。
- 查看调试的进程，`info inferiors`
- 切换当前调试的进程：`inferior id`
- 使进程脱离 GDB 的调试：`detach inferiors id`