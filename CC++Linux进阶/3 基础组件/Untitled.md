# 缓存命中-写回策略



# 缓存不一致

总线嗅探，事务串行化（锁总线）

- 锁M和E的状态

# MESI一致性协议

![image-20240323234311437](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240323234311437.png)

![image-20240323234250237](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240323234250237.png)



# 原子操作

![image-20240324001224021](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240324001224021.png)

# 内存序

## `memory_order_relaxed`

- 读/写，效率高，只保证原子性

没有同步性，没有指导优化



## `memory_order_release` 

写操作， 后面的可以优化到前面来，反之则不行

写操作是“果”，因此前面的“因”不能优化到“果”的后面

![image-20240324001048785](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240324001048785.png)

## `memory_order_acquire`

读操作， 前面的可以优化到后面去，反之则不行

读操作是“因”，因此前面的“果”可以优化到“因”的后面

![image-20240324001031481](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240324001031481.png)

## `memory_order_sec_cst`

在被这个修饰的原子操作的前后都不能交换位置