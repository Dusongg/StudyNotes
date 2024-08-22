SFINAE（Substitution Failure Is Not An Error）是 C++ 中的一种编译期特性，允许模板在类型替换失败时不导致编译错误，而是尝试其他模板重载或特化。它主要用于模板编程中，以支持条件编译和选择合适的模板实例。

### SFINAE 的基本概念

SFINAE 的核心思想是：如果在模板实例化过程中某个替换失败，这种失败不会导致编译错误，而是会让编译器继续尝试其他可能的匹配。

### 主要用法

1. **条件模板选择**： 使用 SFINAE 可以根据模板参数的特性选择合适的模板版本。例如，根据类型是否具有某种成员函数来选择不同的实现。
2. **检测类型特性**： 可以利用 SFINAE 检测某个类型是否具有某种特性（如成员函数、嵌套类型等），并根据检测结果选择不同的代码路径。



# `std::enable_if`

- 编译时根据布尔条件启用或禁用模板重载

```cpp
template<typename T>
typename std::enable_if<std::is_integral<T>::value, std::string>::type
printType() {
    std::cout << "int type call" << std::endl;
    return "int";
}

template<typename T>
typename std::enable_if<!std::is_integral<T>::value, std::string>::type
printType() {
    std::cout << "not int type call" << std::endl;
    return "not int";
}
int main() {
    std::cout << printType<double>();
}
```





# `std::declval`

- 检测T类型是否有某一成员函数

```cpp
#include <iostream>
#include <type_traits>

// 使用 std::declval 和 SFINAE 检测类型是否有成员函数 foo
template<typename T>
class has_foo {
private:
    // 如果 T 类型有成员函数 foo()，则 test() 成功
    template<typename U>
    static auto test(int) -> decltype(std::declval<U>().foo(), std::true_type());

    // 如果 T 类型没有成员函数 foo()，则 test() 失败
    template<typename U>
    static std::false_type test(...);

public:
    static constexpr bool value = decltype(test<T>(0))::value;
};

// 函数模板，根据类型是否有 foo() 来选择重载
template<typename T>
typename std::enable_if<has_foo<T>::value, void>::type
callFoo(T& obj) {
    obj.foo(); // 调用 T::foo()
}

template<typename T>
typename std::enable_if<!has_foo<T>::value, void>::type
callFoo(T&) {
    std::cout << "Type does not have foo()" << std::endl;
}

class A {
public:
    void foo() { std::cout << "A::foo()" << std::endl; }
};
class B {}; // 没有 foo()

int main() {
    A a;
    B b;
    callFoo(a); // 输出: A::foo()
    callFoo(b); // 输出: Type does not have foo()
    return 0;
}
```

