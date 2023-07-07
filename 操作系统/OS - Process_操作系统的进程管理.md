
# 0. What is Process

系统资源分配的基本单位。是程序运行的一个实例。

# 1. How Process work

进程是怎么运行的

## 1.0 进程状态

三种基本状态： Ready, Running, Blocked

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230707143915.png)

## 1.1 进程调度

### 时间片轮转调度算法

（RR, Round-Robin）

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230707144738.png)
优点： 公平，响应快。

缺点：时间片如果太大，就相当于 FCFS 调度算法；时间片如果太小，处理机切换频繁，开销增大。

# 2. How Processes Work TOGERTER

进程之间是如何协作的

## 2.0 进程通信

各进程内存空间彼此独立，一个进程不能随意访问其他进程的地址空间。

### 2.0.0 共享存储 Shared-Memory

有基于共享数据结构的通信方式和基于共享存储区（同一块内存空间）的通信方式。

### 2.0.1 消息传递 Message-Passing

直接通信：点到点发送。发送和接收时指明双方进程的 ID。每个进程维护一个消息缓冲队列（留言）。

间接通信：广播信箱。以信箱为媒介。

### 2.0.2 管道通信 Pipe

管道的本质是一个共享文件，pipe 文件。是内存中固定大小的缓冲区。只能单向通信。以**流**的形式读写。

## 2.1 进程同步

### 2.1.0 Overview

广义上，进程同步包括两种情况：互斥和同步。互斥是排他地访问共享资源，同步是进程合作，如管道通信。

并发的进程对共享资源存在竞争。进程通信中发的信号叫**消息**和**事件**。

### 2.1.1 临界区互斥

临界区即临界资源即共享资源。

#### 信号量 Semaphore

PV 操作，即等待唤醒机制。

# 3. Dead Lock