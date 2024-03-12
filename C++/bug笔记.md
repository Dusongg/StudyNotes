# 1 union的构造函数默认删除

![image-20240307144823789](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240307144823789.png)

C++11标准引入了对联合体（union）的新规定，要求联合体中如果含有非平凡（non-trivial）的成员（如自定义的构造函数、拷贝构造函数、析构函数等），则该联合体的默认构造函数会被删除。这是因为联合体的特性决定了只能同时使用一个成员，而非平凡的构造函数可能会导致程序员误解联合体的使用方式，可能会造成不符合预期的行为

```cpp
class test {
public:
    union testunion {
        testunion() {}    //不能用default
        ~testunion() {}
        function<int(int)> f1;
        function<int(int)> f2;
    };

    testunion func;
    test() {}         // 默认构造函数
    ~test() {}        // 析构函数
};

int main() {
    test t1; // 创建一个 test 对象
    return 0;
}
```

# 2 懒汉模式静态局部变量初始化为线程安全的

> 单例模式是一种常见的设计模式，用于确保一个类只有一个实例，并提供一个全局访问点。在 C++ 中，可以通过以下方式实现单例模式：
>
> ```cpp
> cppCopy code#include <iostream>
> 
> class Singleton {
> public:
>     // 静态成员函数，返回类的唯一实例
>     static Singleton& getInstance() {
>         // 使用局部静态变量保证线程安全的懒汉式单例模式
>         static Singleton instance;
>         return instance;
>     }
> 
>     // 删除拷贝构造函数和赋值运算符，确保只能通过 getInstance() 获取实例
>     Singleton(const Singleton&) = delete;
>     void operator=(const Singleton&) = delete;
> 
>     // 示例方法
>     void showMessage() {
>         std::cout << "Hello, I am a singleton instance!" << std::endl;
>     }
> 
> private:
>     // 私有构造函数，防止外部实例化
>     Singleton() {}
> };
> 
> int main() {
>     // 获取单例实例
>     Singleton& instance = Singleton::getInstance();
>     // 调用示例方法
>     instance.showMessage();
> 
>     return 0;
> }
> ```
>
> 在这个例子中，Singleton 类的构造函数是私有的，只能通过静态成员函数 `getInstance()` 获取实例。getInstance() 方法使用了局部静态变量，在第一次调用时创建实例，并在后续调用中返回相同的实例，实现了懒汉式单例模式。同时，删除了拷贝构造函数和赋值运算符，确保单例类不会被拷贝或赋值。

这个个局部静态对象何时初始化呢

> 在 C++ 中，局部静态变量的初始化时机是在程序第一次执行到该变量的声明语句时进行。具体来说，在 `getInstance()` 函数首次被调用时，`static Singleton instance;` 这行代码会被执行，导致局部静态变量 `instance` 被初始化为 Singleton 类的实例。由于局部静态变量的生命周期延长至程序结束，因此 `instance` 变量会在整个程序运行过程中保持存在，直到程序结束时被销毁。

为什么能保证线程安全呢

> 在 C++11 标准中，局部静态变量的初始化是线程安全的。具体来说，C++11 标准保证了当多个线程同时首次进入同一个函数，并且该函数中包含局部静态变量的初始化时，只会有一个线程执行初始化操作，其他线程会等待初始化完成。这种保证可以避免多个线程同时对静态变量进行初始化的竞争条件，从而确保了线程安全性。
>
> 在早期的 C++ 标准中（比如 C++98/03），对于静态变量的初始化并没有明确的线程安全规定，因此需要通过额外的手段（比如使用互斥锁）来保证线程安全。但是在 C++11 标准中，引入了对局部静态变量初始化的线程安全保证，使得开发者不再需要手动添加额外的线程保护措施。



# 3 C++解决循环依赖

- C++中如果一个头文件需要使用另一个头文件的函数，但是那个函数的参数又包含当前头文件的结构体怎么办

```cpp
// A.hpp
#ifndef A_HPP
#define A_HPP

namespace B_ns {
    class B_b;  // 前向声明 B_b
}

void func(B_ns::B_b test);

#endif
```

```cpp
// B.hpp
#ifndef B_HPP
#define B_HPP

namespace B_ns {
    class B {
    public:
        struct B_b {
            int it;
        };

        void func(B_b test);

    private:
        B_b test;
    };
}

#endif
```



