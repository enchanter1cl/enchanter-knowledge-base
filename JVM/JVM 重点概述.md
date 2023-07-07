

出 JVM 的大致流程是把一个 class 文件通过类加载器加载进系统，然后放到不同的区域，通过 编译器编译。 

Class 文件进入内存后，该如何存储不同数据。

![JVM结构.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/JVM%E7%BB%93%E6%9E%84.png)

# 0. Runtime Data Areas

运行时数据区

这其中最复杂的是运行时数据区，它也是 JVM 内存结构最重要的部分。运行时数据区又可以分为方法区、虚拟机栈、本地方法栈、堆以及程序计数器，并且方法区和堆是**线程共享**的，虚拟机栈、本地方法栈、程序计数器是**线程隔离**的。下面详细讲解运行时数据区的各个组成部分。

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230707163505.png)


## 0.0 PC Register

程序计数寄存器（**Program Counter Register**），Register 的命名源于 CPU 的寄存器，寄存器存储指令相关的线程信息，CPU 只有把数据装载到寄存器才能够运行。**JVM 中的 PC 寄存器是对物理 PC 寄存器的一种抽象模拟**。

## 0.1 JVM Stacks

虚拟机栈

每个线程在创建的时候都会创建一个虚拟机栈。

JVM 直接对虚拟机栈的操作只有两个：每个方法执行，伴随着**入栈**（进栈/压栈），方法执行结束**出栈**。

每个线程都有自己的栈，栈中的数据都是以**栈帧（Stack Frame）的格式存在**。在这个线程上正在执行的每个方法都各自有对应的一个栈帧。

## 0.2 Native Method Stack

简单的讲，一个 Native Method 就是一个 Java 调用非 Java 代码的接口。我们知道的 Unsafe 类就有很多本地方法。

## 0.3 Heap

Java 堆是 Java 虚拟机管理的内存中最大的一块，被所有线程共享。此内存区域的唯一目的就是存放对象实例。

## 0.4 Method Area

存储类信息、方法信息、常量池、静态变量等等。

常量池（Constant Pool Table），用于存放编译期生成的各种字面量和符号引用。当然了，运行期间也可能将新的常量放入池中，这种特性被开发人员利用得比较多的是 `String.intern()`方法。