
现在 CPU 都是多核的，由于 L1/L2 Cache 是多个核心各自独有的，那么会带来多核心的**缓存一致性（_Cache Coherence_）** 的问题。

# 写传播与事务串行化

要解决这一问题，就需要一种机制，来同步两个不同核心里面的缓存数据。要实现的这个机制的话，要保证做到下面这 2 点：

- 第一点，某个 CPU 核心里的 Cache 数据更新时，必须要传播到其他核心的 Cache，这个称为**写传播（_Write Propagation_）**；
- 第二点，某个 CPU 核心里对数据的操作顺序，必须在其他核心看起来顺序是一样的，这个称为**事务的串行化（_Transaction Serialization_）**。

写传播好理解，事务串行化是什么？

如图，是只保证了写传播，没保证串行化的结果：

![image.png](https://image-bed-erato.oss-cn-beijing.aliyuncs.com/obsdian/20230707010717.png)

那接下来我们看看，写传播和事务串行化具体是用什么技术实现的。

---

# 总线嗅探

写传播的原则就是当某个 CPU 核心更新了 Cache 中的数据，要把该事件广播通知到其他核心。最常见实现的方式是**总线嗅探（_Bus Snooping_）**。 但是这样总线会负载很大，很累，于是有了 MESI 协议。

# MESI 协议

MESI 协议四个字母分别是：

- _Modified_，已修改
- _Exclusive_，独占
- _Shared_，共享
- _Invalidated_，已失效

这四个状态来标记 Cache Line 四个不同的状态。

「独占 Exclusive」状态的时候，数据只存储在一个 CPU 核心的 Cache 里，而其他 CPU 核心的 Cache 没有该数据。这个时候，如果要向独占的 Cache 写数据，就可以直接自由地写入，而不需要通知其他 CPU 核心，因为只有你这有这个数据，随便操作。

另外，在「独占」状态下的数据，如果有其他核心从内存 Memory 读取了相同的数据到各自的 Cache ，那么这个时候，独占状态下的数据就会变成共享状态。

那么，「共享 Shared」状态代表着相同的数据在多个 CPU 核心的 Cache 里都有，所以当我们要更新 Cache 里面的数据的时候，不能直接修改，而是要先向所有的其他 CPU 核心广播一个请求，要求先把其他核心的 Cache 中对应的 Cache Line 标记为「无效」状态，然后再更新当前 Cache 里面的数据。

整个 MESI 的状态就是一个有限状态机。

参考资料：

推荐这篇深度长文 [2.4 CPU 缓存一致性 | 小林coding (xiaolincoding.com)](https://www.xiaolincoding.com/os/1_hardware/cpu_mesi.html#%E6%80%BB%E7%BB%93)