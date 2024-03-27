

# 1. 变量模板

```cpp
template<typename T>
constexpr T PI = T(3.14159265358979);
int main() {
	cout << PI<int> << endl;      //3
	cout << PI<float> << endl;     //3.14159
}
```

# 2. 结构化绑定-C++17

```cpp
int main() {
    vector<pair<string, int>> vp{ {"pair1",2}, 
                                 {"pair2", 4}, 
                                 {"pair3", 6} };
    for (auto& [p1, p2] : vp) {
        cout << format("{}: {}   ", p1, p2);
    }
    cout << '\n';
    for (auto& [p1, _] : vp) {
        cout << format("{}: {}   ", p1, "null");
    }
}
```

![image-20240229112927827](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240229112927827.png)

# 3 make_heap-C++20

![image-20231223115724534](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231223115724534.png)

```cpp
#include <algorithm>
#include <functional>
#include <iostream>
#include <string_view>
#include <vector>
 
void print(std::string_view text, std::vector<int> const& v = {})
{
    std::cout << text << ": ";
    for (const auto& e : v)
        std::cout << e << ' ';
    std::cout << '\n';
}
 
int main()
{
    print("Max heap");
 
    std::vector<int> v{3, 2, 4, 1, 5, 9};
    print("initially, v", v);
 
    std::make_heap(v.begin(), v.end());
    print("after make_heap, v", v);
 
    std::pop_heap(v.begin(), v.end());
    print("after pop_heap, v", v);
 
    auto top = v.back();
    v.pop_back();
    print("former top element", {top});
    print("after removing the former top element, v", v);
 
    print("\nMin heap");
 
    std::vector<int> v1{3, 2, 4, 1, 5, 9};
    print("initially, v1", v1);
 
    std::make_heap(v1.begin(), v1.end(), std::greater<>{});
    print("after make_heap, v1", v1);
 
    std::pop_heap(v1.begin(), v1.end(), std::greater<>{});
    print("after pop_heap, v1", v1);
 
    auto top1 = v1.back();
    v1.pop_back();
    print("former top element", {top1});
    print("after removing the former top element, v1", v1);
}
```

输出：

```cpp
Max heap:
initially, v: 3 2 4 1 5 9
after make_heap, v: 9 5 4 1 2 3
after pop_heap, v: 5 3 4 1 2 9
former top element: 9
after removing the former top element, v: 5 3 4 1 2
 
Min heap:
initially, v1: 3 2 4 1 5 9
after make_heap, v1: 1 2 4 3 5 9
after pop_heap, v1: 2 3 4 9 5 1
former top element: 1
after removing the former top element, v1: 2 3 4 9 5
```



# 4 range::sort()

![image-20240211211112209](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240211211112209.png)

```cpp
#include <stdio.h>
#include <algorithm>
#include <vector>
#include <string>
using std::string;
int main() {

	std::vector<string> words{ "adg", "adf", 'a', 'ss'};
	std::sort(words.begin(), words.end(), [&](string l, string r) {
		return l.length() < r.length();
		});

	//c++20
	std::ranges::sort(words, [&](string l, string r) {
		return l.length() < r.length();
		});
}
```



# 5 set/map/unordered_set/unordered_map新增方法

## 5.1 `node_type`结点C++17

![image-20240229114617004](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240229114617004.png)



## 5.2 `contains` （类似于python中的`in`）C++20

```cpp
#include <iostream>
#include <unordered_set>
 
int main()
{
    std::unordered_set<int> example{1, 2, 3, 4};
 
    for (int x : {2, 5})
        if (example.contains(x))
            std::cout << x << ": Found\n";
        else
            std::cout << x << ": Not found\n";
}
```

## 5.3 `insert_range`C++23

```cpp
#include <iostream>
#include <unordered_set>
 
void println(auto, auto const& container)
{
    for (const auto& elem : container)
        std::cout << elem << ' ';
    std::cout << '\n';
}
 
int main()
{
    auto container = std::unordered_set{1, 3, 2, 4};
    const auto rg = {-1, 3, -2};
 
#ifdef __cpp_lib_containers_ranges
    container.insert_range(rg);
#else
    container.insert(rg.begin(), rg.end());
#endif
 
    println("{}", container);
}
//output : 4 -1 2 3 -2 1
```

## 5.4 `extract`提取 C++17

> 1) 解除含 position 所指向元素的结点的链接并返回拥有它的[结点句柄](https://zh.cppreference.com/w/cpp/container/node_handle)。
> 2) 若容器拥有键等于 k 的元素，则从容器解除该元素的节点并返回拥有它的[结点句柄](https://zh.cppreference.com/w/cpp/container/node_handle)。否则，返回空结点句柄。
> 3) 同 (2)。此重载只有在限定标识 Compare::is_transparent 合法并指代类型，且 `iterator` 与 `const_iterator` 均不可从 `K` 隐式转换时才会参与重载决议。它允许调用此函数时无需构造 `Key` 的实例。
>
> **任何情况下，均不复制或移动元素，只重指向容器结点的内部指针**（可能发生再平衡，和 [erase()](https://zh.cppreference.com/w/cpp/container/set/erase) 一样）。提取结点只会使指向被提取元素的迭代器失效。指向被提取元素的指针和引用保持有效，但在结点句柄拥有该元素时不能使用：一旦元素被插入容器，就能使用它们。

```cpp
#include <algorithm>
#include <iostream>
#include <string_view>
#include <set>
 
void print(std::string_view comment, const auto& data)
{
    std::cout << comment;
    for (auto datum : data)
        std::cout << ' ' << datum;
 
    std::cout << '\n';
}
 
int main()
{
    std::set<int> cont{1, 2, 3};
 
    print("Start:", cont);
 
    // 提取节点句柄并改变键
    auto nh = cont.extract(1);
    nh.value() = 4;  
 
    print("After extract and before insert:", cont);
 
    // 将节点句柄插回去
    cont.insert(std::move(nh));
 
    print("End:", cont);
}

/*
output:
    Start: 1 2 3
    After extract and before insert: 2 3
    End: 2 3 4
*/
```



## 5.5 `erase_if`

![image-20240229115202368](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240229115202368.png)

## 5.6 `merge`

> 尝试提取（“接合”）`source` 中的每个元素，并用 *this 的比较对象插入到 *this。 若 *this 中有元素的键等价于来自 source 中某元素的键，则不从 source 提取该元素。 不复制或移动元素，只会重指向容器结点的内部指针。指向被转移元素的所有指针和引用保持有效，但现在指代到 *this 中而非到 source 中。
>
> 若 get_allocator() != source.get_allocator() 则行为未定义。

```cpp
#include <iostream>
#include <set>
 
// 打印出容器
template<class Os, class K>
Os& operator<<(Os& os, const std::set<K>& v)
{
    os << '[' << v.size() << "] {";
    bool o{};
    for (const auto& e : v)
        os << (o ? ", " : (o = 1, " ")) << e;
    return os << " }\n";
}
 
int main()
{
    std::set<char>
        p{'C', 'B', 'B', 'A'}, 
        q{'E', 'D', 'E', 'C'};
 
    std::cout << "p: " << p << "q: " << q;
 
    p.merge(q);
 
    std::cout << "p.merge(q);\n" << "p: " << p << "q: " << q;
}
/*
output:
    p: [3] { A, B, C }
    q: [3] { C, D, E }
    p.merge(q);
    p: [5] { A, B, C, D, E }
    q: [1] { C }
*/
```



# 6 管道-C++20

C++20中的管道（`std::ranges::views::pipeline`）是用于构建管道式操作的工具，类似于Unix shell中的管道符号（`|`），允许将多个操作链接在一起以处理数据序列。下面是一个简单的示例，展示了如何使用C++20中的管道：

```cpp
/*eg1*/
cppCopy code#include <iostream>
#include <vector>
#include <ranges>

int main() {
    std::vector<int> vec{1, 2, 3, 4, 5};

    auto pipeline = vec | std::views::transform([](int x) { return x * 2; })
                        | std::views::filter([](int x) { return x % 3 == 0; });

    for (int x : pipeline) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}

/*eg2*/
int main() {
	vector<int> arr{ 1,2,3,4,5,6,7,8 };
	for (auto x : arr | views::drop(2) | views::take(5)) {
		cout << x;
	}
}
```

在这个示例中，我们首先创建了一个包含整数的向量`vec`，然后使用管道操作符`|`将`vec`链接到了一个管道中。管道中的第一个操作是`std::views::transform`，用于将每个元素乘以2。接着，`|`右边是一个`std::views::filter`操作，用于过滤出能被3整除的元素。最后，我们使用`for`循环遍历管道中的结果并输出。

需要注意的是，C++20中的管道操作是惰性求值的，只有在遍历结果时才会执行实际的计算。



# 7 chrono库

```cpp
auto start_time = chrono::steady_clock::now();

///


auto end_time = chrono::steady_clock::now();
auto ms = chrono::duration_cast<chrono::milliseconds>(end_time - start_time).count();
cout << "Serail task finish, " << ms << " ms consumed, Result: " << sum << endl;
```



# 8 any



# 9 Coroutines

## 线程与协程的区别

> C++中的协程和线程都是并发编程的概念，但它们有不同的执行方式和调度方式：
>
> 1. **执行方式：**
>    - 线程是由操作系统调度的独立执行单元，一个进程可以包含多个线程，每个线程有自己的堆栈和程序计数器，可以并发执行不同的代码路径。
>    - 协程是一种轻量级的执行单元，可以在一个线程内多个协程之间切换执行，但同一时刻只有一个协程处于执行状态。
> 2. **调度方式：**
>    - 线程的调度由操作系统负责，**<u>线程之间的切换需要进行上下文切换，涉及到内核态和用户态的切换</u>**，开销较大。
>    - <u>**协程的调度由程序员控制，可以在协程之间自由切换，切换时不涉及内核态和用户态的切换，开销较小。**</u>
> 3. **资源消耗：**
>    - 线程的创建和销毁需要操作系统参与，涉及到堆栈和寄存器的分配和释放，资源消耗较大。
>    - 协程是在用户空间管理的，创建和销毁开销较小，不需要操作系统参与。
> 4. **适用场景：**
>    - 线程适用于需要真正的并发执行和利用多核处理器的场景，例如需要同时处理大量IO操作或者进行密集计算的情况。
>    - 协程适用于IO密集型的应用，可以提高程序的响应性能，减少线程切换带来的开销，同时也适用于需要管理大量并发任务的场景。
>
> 在实际应用中，选择线程还是协程取决于具体的需求和性能要求。

## 同步与异步的区别

> 同步（Synchronous）和异步（Asynchronous）是描述程序执行方式的两个概念，主要涉及到任务的触发、执行和完成时机。
>
> 1. **同步**：在同步操作中，任务是按顺序依次执行的，每个任务需要等待上一个任务完成后才能开始执行。在一个任务执行的过程中，如果需要等待某个操作完成（比如等待IO操作完成或者等待另一个任务的结果），该任务会被阻塞，直到操作完成才能继续执行。
> 2. **异步**：在异步操作中，任务的触发和执行是独立于主程序流程的，任务可以在后台或者并行执行，不需要等待上一个任务完成。异步操作通常会在任务完成时通知主程序，主程序可以继续执行其他任务，不需要等待异步操作完成。
>
> 在实际编程中，同步和异步的选择取决于应用的需求和性能要求。同步操作通常会导致程序阻塞，降低了程序的响应速度，但编写起来相对简单；而异步操作可以提高程序的响应速度和并发性，但编写起来可能会更加复杂，需要处理异步操作的完成通知和结果处理逻辑。





# 10 类型转换相关

## 10.1 禁止隐式类型转换

```cpp
void f(int) {}

template<typename T>
void f(T) = delete;

struct X {
	operator int()const { return 0; }
};

int main() {
	f(1);

	//以下类型转换报错
	//f(1u);
	//f(1lu);
	//f(1.);
	//f(1.f);
	//f(X());
}
```

## 10.2 `dynamic_cast` 和 `static_cast`的区别

### 10.2.1 多态类类型

![image-20240327205351459](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240327205351459.png)



# 11 在<type_traits>里的

## 11.1 `is_same`

![image-20240327225608984](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240327225608984.png)

```cpp
#include <cstdint>
#include <iostream>
#include <type_traits>
 
int main()
{
    std::cout << std::boolalpha;
 
    // 一些由实现定义的状况
 
    // 若 'int' 为 32 位则通常为 true
    std::cout << std::is_same<int, std::int32_t>::value << '\n'; // 可能为 true
    // 若使用 ILP64 数据模型则可能为 true
    std::cout << std::is_same<int, std::int64_t>::value << '\n'; // 可能为 false
 
    // 与上面相同的测试，但使用了 C++17 的 std::is_same_v<T, U> 格式
    std::cout << std::is_same_v<int, std::int32_t> << ' ';  // 可能为 true
    std::cout << std::is_same_v<int, std::int64_t> << '\n'; // 可能为 false
 
    // 比较一对变量的类型
    long double num1 = 1.0;
    long double num2 = 2.0;
    static_assert( std::is_same_v<decltype(num1), decltype(num2)> == true );
 
    // 'float' 决非整数类型
    static_assert( std::is_same<float, std::int32_t>::value == false );
 
    // 'int' 为隐式的 'signed'
    static_assert( std::is_same_v<int, int> == true );
    static_assert( std::is_same_v<int, unsigned int> == false );
    static_assert( std::is_same_v<int, signed int> == true );
 
    // 不同于其他类型，'char' 既非 'unsigned' 亦非 'signed'
    static_assert( std::is_same_v<char, char> == true );
    static_assert( std::is_same_v<char, unsigned char> == false );
    static_assert( std::is_same_v<char, signed char> == false );
 
    // const 限定的类型 T 与非 const T 不同
    static_assert( !std::is_same<const int, int>() );
}
```



# 12 cv限定符 / [存储类型说明符](https://zh.cppreference.com/w/cpp/language/storage_duration)

![image-20240327231357916](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240327231357916.png)



# 13 [数组声明](https://zh.cppreference.com/w/cpp/language/array)

![image-20240328000236913](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240328000236913.png)

![image-20240327233705627](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240327233705627.png)
