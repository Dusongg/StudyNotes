https://juejin.cn/post/7194435171633299513

# 单元测试框架

1. googletest

## [doctest](https://github.com/doctest/doctest)

- 分组执行：

  1. `TEST_CASE`,`SUBCASE`, `TEST_SUITE`

  2. ```
     TEST_CASE_TEMPLATE()   //测试模板
     TEST_CASE_FIXTURE()    //测试类的继承？
     ```

- 断言：`REQUIRE`, `CHECK`, `WARN`

- 其他方法。。。



# benchmark框架——`nanobench`

用处：

- 计时函数、
- 大量测试->每次测试时间的波动，
-  指令条数，指令有多少次分支预判，有多少次分支预判成功了
- 计算某个操作的时间复杂度
- 输出可视化图

```cpp
		int y = 0;
		std::atomic<int> x(0);
		ankerl::nanobench::Bench().run("testing Tarjan", [&] {
			x.compare_exchange_strong(y, 0);
		});
		//防止被优化
		int z = 1;
		ankerl::nanobench::Bench().run("++z", [&]() {
			ankerl::nanobench::doNotOptimizeAway(++z);
		});
```

- 计算时间复杂度

```cpp
TEST_CASE("nanobench test") {
		ankerl::nanobench::Bench bench;
		ankerl::nanobench::Rng rng;
		std::set<uint64_t> set;
		for (auto set_size : {10U, 20U, 50U, 100U, 500U, 1000U, 2000U, 5000U, 10000U}) {
			while(set.size() < set_size) {
				set.insert(rng());
			}

			bench.complexityN(set.size()).run("stt::set find", [&] {
				ankerl::nanobench::doNotOptimizeAway(set.find(rng()));
			});
		}
		std::cout << bench.complexityBigO() << std::endl;

	}
```



# 测试覆盖率

- 测试覆盖率：gcc:`--converage`

![image-20240605004744913](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240605004744913.png)

- clang/gcc编译器，编译选项

  - `-fsanitizer=address`

  - `-fsanitizer=leak` :检测内存泄漏

    ```cmake
    #set(CMAKE_EXE_LINKER_FLAGS "-static")   使用时不能带-static选项
    
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=leak")  #检测内存泄露
    ```

    ![image-20240605010603902](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240605010603902.png)

  - `-fsanitizer=undefined`：检测未定义行为（例如函数返回值） 

  - `-fsanitizer=thread`：检测多线程的数据竞争 

    ```cmake
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=thread")  #检测线程间的资源竞争
    ```

    

  ![image-20240605011705311](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240605011705311.png)

# profile分析性能瓶颈

















