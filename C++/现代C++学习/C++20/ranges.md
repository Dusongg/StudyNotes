# 1 管道

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



# 2 常用的范围适配器和操作

```cpp
#include <algorithm>
#include <iostream>
#include <ranges>
#include <vector>

int main() {
    std::vector<int> nums{1,2,3,4,5};
    std::pmr::vector<int> pmr_nums{1,2,3,4,5};
    //1. filter: 筛选
    auto even_nums = nums | std::ranges::views::filter([](int x) {return x % 2 == 0;});
    for (auto x : even_nums) {
        std::cout << x << ' ';
    }
    std::cout << std::endl;

    //2. transform: 用于转换每个元素
    auto equared_nums = nums | std::ranges::views::transform([](int x) {return x * x;});
    for (auto x : equared_nums) {
        std::cout << x << ' ';
    }
    std::cout << std::endl;

    //3. take: 获取某个范围里的前n个元素  ; drop : 丢弃前n个元素
    for (auto x : std::vector<int>{1,2,3,4,5,6,7,8,9,10} | std::ranges::views::take(5) | std::ranges::views::drop(2)) {
        std::cout << x  << ' ';
    }

    //4. reverse
    auto reversed_nums = nums | std::views::reverse;
    //std::reverse()
    // std::reverse(nums.begin(), nums.end());

    //5. sort
    std::ranges::sort(nums);

}

```

