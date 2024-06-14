# CPU Cache 数据写入策略

## 写直达



## 写回

![image-20240324172749933](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240324172749933.png)

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

```cpp

atomic<bool> x, y;
atomic<int> z;
//代码最终目的：先x = true  后 y = true， 让z++
void write_x_then_y() {
	x.store(true, std::memory_order_relaxed);
	y.store(true, std::memory_order_release);   //写：先x后y
}

void read_y_then_x() {
	while (!y.load(std::memory_order_acquire));   //读
	if (x.load(std::memory_order_relaxed)) {   //这一句不能优化到acquire的前面
		z++;		// 确保z等于1
	}
}

int main() {
	x = y = false;
	z = 0;
	thread t1(write_x_then_y);
	thread t2(read_y_then_x);
	t1.join();
	t2.join();
	cout << z << endl;
	return 0;
}
```

