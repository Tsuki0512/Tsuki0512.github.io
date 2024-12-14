# 操作系统期末复习

*参考笔记：[Isshiki修's Notebook](https://note.isshikih.top/cour_note/D3QD_OperatingSystem/Unit0/)*

我的os笔记方式和ads类似，还是将重心放在**知识框架的梳理**上，对于笔记中提到的东西如果有点陌生，还是建议去翻找ppt或者王道~

## 0 - Overview

### 0.1 操作系统定义

- 职能角度 - 资源管理
- 存在角度 - 管理用户程序的软件程序，kernel是计算机开机后一直在运行的程序

### 0.2 操作系统评价方式

1. 可靠性 - 异常处理机制
2. 安全性 - 权限管理机制 - 不同权限模式
3. 易用性 - 系统调用机制
      - 命令接口、程序接口、图形用户接口、命令行接口...
4. 高效性 - 任务执行机制 - 分时系统实现体感上的并行
5. 公平性 - 进程管理机制 - 避免饥饿现象
6. 可拓展性、易维护性等开发者角度

### 0.3 操作系统架构

- 单处理器系统 - 有且仅有一个单核的通用处理器（可以有别的不运行线程的专用处理器）
- 多处理器系统 - 多个单核的通用处理器 - 吞吐量提升，但非线性提升
- 集群系统 - 多个独立的计算机系统作为节点，通过冗余实现高可用服务，通过并行实现高性能计算
    - 对称集群 - 各个节点互相监督
    - 不对称集群 - 替补关系

### 0.4 操作系统任务处理

- 批处理系统：成批、串行，交互性差
    - 单道批处理阶段 - 同一时间一个任务，会导致CPU长时间空闲
    - 多道批处理阶段 - 宏观上并发，微观上串行
- 分时系统：“体感上”并行，按照时间片进行进程间切换
- 实时系统：特定时间完成特定任务
- 分布式系统：多台计算机共同完成一个任务

### 0.5 操作系统结构设计

有中心：

- 宏内核：耦合所有主要功能，效率高，可靠性低
- 微内核：只提供最基本功能，可靠性高，可拓展性高

*混合系统结合了宏内核和微内核的设计思路。*

分层网状：

- 分层设计：类似计网，$i$用$i-1$层接口，开发维护方便，但逐层接口调用导致效率受限
- 模块化设计：非递进式，将操作系统划分成独立模块

### 0.6 操作系统运行原理

引导 $\to$ 用户态 |（中断（计时器）、系统调用）| 内核态

中断：

- 从紧急程度：可屏蔽中断、不可屏蔽中断
- 从触发原因：内中断（异常等程序导致的）、外中断（硬件产生）

## 1 - Process Management

*进程是资源分配和管理的独立单位*

### 1.1 进程基本概念

#### 1.1.1 进程的形式

- 内核态虚拟内存 - 元数据（进程控制块PCB）

  - ![image-20241127232049530](./markdown-img/final_review.assets/image-20241127232049530.png) 

- 用户态虚拟内存 - 实际需要的数据资源

    ![image-20241209211633935](./markdown-img/final_review.assets/image-20241209211633935.png)

- 静态部分
    - text section - 代码
    - data section - 进程全局变量、静态变量
- 动态更新部分
    - heap - 被动态分配的内存
    - stack - 暂时性数据，函数传参、返回值等

#### 1.1.2 进程的状态

进程的状态和转换关系：

![image-20241209212059494](./markdown-img/final_review.assets/image-20241209212059494.png)

实现调度需要维护等待队列（可能有多个）和就绪队列：

![image-20241209212245180](./markdown-img/final_review.assets/image-20241209212245180.png)

### 1.2 进程管理

#### 1.2.1 创建 - `fork()`

1. 复制父进程数据，即直接`fork`
2. 载入新程序并继续执行，`fork`后`execXX()`

相关技术：COW、`vfork()`

#### 1.2.2 终止 - `exit()`

子进程通过`exit()`终止之后（僵尸进程）需要父进程的`wait()`去回收资源，如果此时父进程被终止则变成孤儿进程，交由`init/systemd`来`wait()`或通过级联终止来避免。

#### 1.2.3 通信

- 信号量
- 共享内存 - 通过系统调用建立，更快
- 信息传递 - 不需要处理数据冲突，少量有用，分布式系统更容易实现
- 文件/管道（本质上也是一种文件，单向传输）

### 1.3 进程调度

- 调度分类
    - 抢占式 - 不占有资源的进程索取cpu资源
    - 非抢占式 - 已拥有资源的进程主动释放cpu资源
- 调度过程 - `__switch_to`
    1. 上下文切换 - 寄存器、进程状态、管理信息
    2. `sret`回用户态`sepc`
- 调度算法指标
    - cpu使用率
    - 吞吐量 - 单位时间内完成的进程数
    - 周转时间 -开始建立 $\to$ 进程完成
    - 等待时间 - = 周转时间 - 运行时间
    - 响应时间 - 发出请求 $\to$ 第一次响应
- 调度算法
    - FCFS - 非抢占式、实现简单
    - SJF - 非抢占式、平均等待时间最小，会导致饥饿现象且运行时间不好预估
    - SRTF - 抢占式（剩余运行时间最短）、平均等待时间最小，会导致饥饿现象且运行时间不好预估
    - RR - 抢占式、解决饥饿现象，频繁切换会导致较高的dispatch latency
    - priority scheduling - 和SJF/SRTF一样分成抢占式和非抢占式两种，会导致饥饿现象
        - priority aging - 避免饥饿现象，优先级随着等待时间不断增长
- 调度设计
    - multilevel queue scheduling - 队列间根据优先级抢占式调度，队列内根据需求采取不同的调度算法
    - multilevel feedback queue scheduling - 允许进程在队列间转移（占有太久cpu -- 优先级`--`；等待太久 -- 优先级`++`）

### 1.4 线程

定义：进程内可调度的执行单元

目的：减小fork和切换的开销

进程间线程共享静态资源，各自维护必需的动态资源：便于线程间通信、任务粒度小减少响应时间；进程内出问题会影响所有线程、内存保护问题。

多线程模型：

- 用户级多线程：支持更多线程数，更容易自定义线程调度算法
- 内核级多线程：单线程阻塞不导致整个进程阻塞，可以利用多核实现并行
- 关系（用户-内核）：一对一、多对多、多对一

## 2 同步

### 2.1 同步工具

#### 2.1.1 相关定义

- race condition：两个进程同时访问一个资源，并且需要对资源作出修改（这个操作在汇编级别不是原子的）的情况，如果不加访问限制则会导致使用过时数据从而写入错误结果。

- 临界资源：只能被至多一个用户占有的资源

- critical section（临界区段）：访问临界资源的代码段

  ```
  ┌─────────────────────┐
  │  Entry Section      │ <-- 判断能否进入临界区（不行则等待）
  ├─────────────────────┤
  │  Critical Section   │ <-- 获取、操作临界资源
  ├─────────────────────┤
  │  Exit Section       │ <-- 释放临界资源
  ├─────────────────────┤
  │  Remainder Section  │ <-- other codes
  └─────────────────────┘
  ```

- 临界区问题：如何保证最多只有一个用户在执行临界区段的代码

- 解决方案要求：

  1. 临界互斥
  2. 选择时间（选择下一个进入临界区的进程）有限
  3. 等待时间有限

#### 2.1.2 同步算法

*内核态的同步问题：对于单处理器只需要在中断发生后禁止中断即可；对于多处理器我们通过非抢占式内核实现内核态的进程数量唯一。接下来是一些更普适的解法。*

---

##### 2.1.2.1 Peterson's Algorithm

限制：参与竞争的进程只能有两个

```c
// `i` is 0 or 1, indicating current pid, while `j` is another pid.
process(i) {
    j = 1 - i;
    READY[i] = true;                    // ┐
    TURN = j;                           // │
    while (READY[j] && TURN == j) {}    // ├ entry section
        // i.e. wait until:             // │
        //  (1) j exits,                // │
        //  (2) j is slower, so it      // │
        //      should run now.         // ┘
    /* operate critical resources */    // - critical section
    READY[i] = false;                   // - exit section
    /* other things */                  // - remainder section
}
```

可以通过对各种情况的枚举完成对解决方案三条要求的证明。

但其实这个方法不适用于现代处理器，因为采用的是乱序流水线，`READY[i] = true`和`TURN = j`这两条指令没有严格的执行顺序。

##### 2.1.2.2 Memory Barriers

加入`memory_barrier()`主动禁止指令重排：

```c
// `i` is 0 or 1, indicating current pid, while `j` is another pid.
process(i) {
    j = 1 - i;
    READY[i] = true;                    // ┐
    memory_barrier();                   // │
    TURN = j;                           // │
    while (READY[j] && TURN == j) {}    // ├ entry section
        // i.e. wait until:             // │
        //  (1) j exits,                // │
        //  (2) j is slower, so it      // │
        //      should run now.         // ┘
    /* operate critical resources */    // - critical section
    READY[i] = false;                   // - exit section
    /* other things */                  // - remainder section
}
```

这里还涉及到两个memory model的定义：

1. 强有序(strongly ordered)：进程对内存做的修改立刻对其它处理器可见；
2. 弱有序(weakly ordered)：进程对内存做的修改不立刻对其它处理器可见；

---

##### 2.1.2.3 Hardware Instructions

我的理解是把一些操作在硬件层面包装成原子操作。

**`test_and_set()`**：

```c
// 目标设true并返回旧值
<atomic> test_and_set(bool * target) {
    bool ret = *target;
    *target = true;
    return ret;
}

// 使用test_and_set解决临界问题
process(i) {
    while ( test_and_set(&LOCK) ) {}    // - entry section
    /* operate critical resources */    // - critical section
    LOCK = false;                       // - exit section
    /* other things */                  // - remainder section
}
```

为了实现“有限等待时间”的需求，我们需要采取手动分配锁的调度方式：

```c
// `i` is process id in [0, n), where `n` is the count of related process. 
process(i) {
    WAITING[i] = true;                                  // ┐
    while ( WAITING[i] && test_and_set(&LOCK) ) {}      // ├ entry sec.
                                                        // │
    WAITING[i] = false;                                 // ┘

    /* operate critical resources */                    // - critical sec.

    // i.e. find next waiting process j                 // ┐
    j = (i + 1) % n;                                    // │
    while (i != j && !WAITING[j]) {                     // ├ exit sec.
        j = (j + 1) % n;                                // │
    }                                                   // │
    // release j's LOCK or release whole LOCK           // │
    if (i == j)     LOCK = false;                       // │
    else            WAITING[j] = false;                 // ┘

    /* other things */                                  // - remainder sec.
}
```

由释放锁的进程决定下一个进入临界区的进程，采用从$i + 1$开始的遍历形式保证选择顺序使得不会饥饿。

**`compare_and_swap()`**：

```c
<atomic> compare_and_swap(int * target, int expected, int new_val) {
    int ret = *target;
    // *target = (*target == expected) ? new_val : *target;
    if (*target == expected) {
        *target = new_val;
    }
    return ret;
}
```

和`test_and_set()`功能类似但是更加普适、可拓展性好。

通过这个原子操作，我们可以构造**Atomic Variables**（原子变量）：

```c
increment(atomic_int * v) {
    int tmp;
    do {
        tmp = *v;
    } while ( tmp != compare_and_swap(v, tmp, tmp+1) );
}
```

此时我们直接使用了封装好的原子操作实现了变量修改的原子性，不需要涉及临界区的讨论。

##### 2.1.2.4 Mutex Locks - 互斥锁

是对2.1.2.3的硬件层面原子操作在软件层面的封装：

```c
// `available` means whether the `LOCK` is free, or whether the related 
// resources is available
acquire() {
    while ( !compare_and_swap(&available, true, false) ) {}
}
release() {
    available = true;
}
```

这里也引入了两个概念：

- 忙等待：在`entry`的等待阶段依然占用CPU资源（使用`while`）
- 自旋锁(spinlock)：使用忙等待的互斥锁

可以通过系统调用避免忙等待，但是要权衡等待浪费的cpu资源和调度开销。

---

##### 2.1.2.5 Semaphores - 信号量

提供的两个标准化原子操作接口：

```c
<atomic> wait(reference S) {
    while (S <= 0) {} // 忙等待，表示资源有限
    S--;
}
<atomic> signal(reference S) {
    S++;
}
```

- counting semaphore - 计数信号量
- binary semaphore - 二值信号量：$0 \le S \le 1$，表示资源只有一个

应用：

1. 程序间通信（或手动串行）

   ```c
   semaphore S = 0;
   P0() {
       /* Section A */
       signal(S);
   }
   P1() {
       wait(S);
       /* Section B */
   }
   ```
2. 互斥

   ```c
   // `i` is process id and S is the semaphores
   process(i) {
       wait(S);    // i.e. release the 'LOCK'
       /* critical section */
       signal(S);  // only one process can pass this line at once
   }
   ```

优化：

避免忙等待：

```c
struct semaphore {
    value;
    waiting_list;
};

<atomic> wait(reference S) {
    S->value--;
    if (S->value < 0) {
        S->waiting_list.push(current process);
        sleep();
    }
}

<atomic> signal(reference S) {
    S->value++;
    if (S->value <= 0) {
        p = S->waiting_list.pop();
        wakeup(p);
    }
}
```

### 2.2 同步问题例子

#### 2.2.1 生产者消费者问题 - The Bounded-Buffer Problem

通过信号量保障生产消费对缓冲区使用的互斥：

```c
// suppose the capacity of the buffer is n
semaphore mutex = 1; // 互斥
semaphore empty = n; // 同步1，生产的等待条件
semaphore full  = 0; // 同步2，消费的等待条件
```

#### 2.2.2 读者写者问题 - The Readers-Writers Problem

由**读写冲突**和**写写冲突**引发的问题，通过以下信号量解决（避免饥饿）：

```c
semaphores rw_mutex   = 1; // 读写/写写互斥锁
semaphores mutex      = 1; // 修改read_count的互斥锁
int        read_count = 0; // 读者数量

// writer's code
writer() {
    wait(rw_mutex);
    /* critical section */
    signal(rw_mutex);
}
// reader's code
reader() {
    wait(mutex);            //   ┐
    read_count++;           //   ├ obtain `mutex` to increase
    if (read_count == 1) {  //   │ `read_count`
        wait(rw_mutex);     // ┐ │
    }                       // │ │
    signal(mutex);          // │ ┘
                            // ├── readers share `rw_mutex` to
    /* critical section */  // │   read critical resource
                            // │
    wait(mutex);            // │ ┐
    read_count--;           // │ ├ obtain `mutex` to decrease
    if (read_count == 0) {  // │ │ `read_count`
        signal(rw_mutex);   // ┘ │
    }                       //   │
    signal(mutex);          //   ┘
}
```

其中，由于`read_count`有比较操作，单一信号量不支持这样的原子操作，所以可以理解为`read_count`和`mutex`一起构成了一个原子变量。

#### 2.2.3 哲学家就餐问题 - The Dining Philosophers Problem

由于筷子的互斥我们对每一个筷子维护了一个信号量，但是这种直接的解法可能会导致死锁（每个哲学家都拿了自己右手边的筷子），我们有以下解决方案：

1. 允许最多 4 位哲学家同时获取筷子；
2. 哲学家必须同时获取两个筷子，而不能抓一支等一支；
    - 为了实现这一点，“抓筷子”这件事应当在一个临界段中完成；
3. 奇数哲学家先拿左手的筷子，偶数哲学家先拿右手的筷子，这样不会产生循环等待。

### 2.3 死锁问题

#### 2.3.1 相关定义

- 死锁：一个处于占有资源且等待资源的状态的线程集合，互相等待彼此占有的资源。
- 资源分配图：
    - 存在环 $\to$ 有可能处于死锁**状态**
    - 不存在环 $\to$ 不可能处于死锁状态
    - 环内节点**只有一个实例** $\to$ 一定处于死锁状态
- 安全状态：存在安全序列
    - 安全状态 $\to$ 一定可以**避免**死锁
    - 不安全状态 $\to$ 不一定可以避免死锁

- 死锁条件（必须同时满足）：
    1. 互斥 - mutual exclusion
    2. 持有并等待 - hold and wait
    3. 非抢占 - no preemption
    4. 循环等待 - circular wait
- 死锁处理：
    1. 从程序逻辑上预防 - 交给开发者
    2. 从行为规范上破坏死锁条件去预防 - **死锁预防**
    3. 禁止可能产生的死锁行为执行 - **死锁避免**
    4. 允许死锁出现后再去消除 - **死锁检测**和**死锁解除**

#### 2.3.2 死锁预防

核心思想是规范行为破坏四个死锁条件之一：

1. 互斥 - 无法破坏
2. 持有并等待 - 将申请并获得资源变成一次性的操作
3. 非抢占 - 允许抢占 - 会带来进程被打断可能无法恢复/继续、饥饿问题
4. 循环等待 - 给资源编号，规定资源申请顺序 - 效率低、资源扩展性差

#### 2.3.3 死锁避免

核心思想是通过一些调度算法组织可能导致死锁的行为：

- 资源分配图算法 - 基于资源分配图 - 适用于每个资源类别**只有一个实例**的情况
    1. 连好**所有**相关的 claim edge；
    2. 对于每条claim edge：
        1. 如果（在提出申请的时候）变为assignment edge不会成环，则变成request edge；
        2. 在获得资源的时候变成assignment edge
        3. 在释放资源的时候删除assignment edge

- 银行家算法 - 基于安全状态
    - 数据结构 - 进程数`n` | 资源种类`m`
        - `Available[m]`
        - `Max[n][m]`
        - `Allocation[n][m]`
        - `Need[n][m]` (`Need[i][j] = Max[i][j] - Allocation[i][j]`)
    - 算法
        1. 安全算法 - 寻找安全序列的方式判断是否为安全状态
        2. 资源请求算法 - 维护安全状态，当线程提出资源请求时，模拟资源被分配后的状态并通过安全算法判断是否仍然为安全状态，如果是，则分配。

#### 2.3.4 死锁检测及死锁解除

- 死锁检测：
    - 单实例：维护等待图（也是每个资源一个实例的，将资源分配图改为进程间等待关系）并定期调用环检测算法 - 维护和检测开销大
    - 多实例：类银行家算法（基于安全状态） *修佬说jjm说不考*
- 死锁解除：
    - 杀死所有死锁中的进程
    - 按照顺序（如优先级）逐个杀直到没有死锁
    - 回滚并且抢占之前分配的资源 - 饥饿 - 通过优先级算法考虑被回滚次数

## 3 内存

内存的管理是基于**页**的，相对于进程粒度更小。

### 3.1 基本概念

- 内存管理单元MMU - Memory management unit
    - 实现进程的内存保护 - 检查访存地址是否位于进程的`[base, base + limit]`范围内
    - 实现进程的**虚拟地址空间**（虚拟内存语境下进程的内存结构）到物理地址的映射 - 使用TBL完成
- 静态代码 $\to$ 动态进程过程中的内存转换
    - compile time - 将符号地址（**symbol address**）转换为可重定位地址（**relocatable address**，相对量）/绝对地址（**absolute address**，但是一旦起始地址发生改变，就需要重新编译）
    - load time - 将可重定位地址转换为绝对地址，此时如果起始地址改变只需重新装载
        - 动态装载（不需要操作系统支持）：未被调用的进程的可重定位地址存在磁盘；被调用时装载进内存
        - 动态链接库/共享库（需要操作系统支持）：能被动态链接的库
    - execute time - 如果进程允许被移动，将可重定位地址到绝对地址的转换放在这一步
- 内存分配
    - 可变划分 - 外部碎片
        - First Fit / Best Fit / Worst Fit ...
    - 固定划分 - 内部碎片

// ----- TODO ... ----- 这也太多了，晕字了

## 4 输入/输出

## 5 存储

## 6 文件系统
