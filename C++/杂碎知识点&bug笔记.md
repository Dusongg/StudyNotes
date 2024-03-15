# 1. 不允许数组拷贝和赋值——P102

``` cpp
int a[] = {1, 2, 3};
int a2[] = a;	//error:不允许使用一个数组初始化另一个数组
a2 = a; 		//error:不能把一个数组直接赋值给另一个数组
```

- 一些编译器支持数组的赋值，但这是**编译器拓展（compiler extension）**，一般来说，最好避免使用非标准特性

# 2. 导致类的拷贝控制成员被定义为删除函数的原因——P450、476、533、751



# 3 在参数列表后放置*引用限定符*(reference qualifier)

> 引子：
>
> ```cpp
> string s1 = "hello";
> string s2 = "world";
> s1 + s2 = "wow!!!";
> ```
>
> 此处我们对两个`string`的链接结果——**一个右值**，进行了赋值

在旧标准中，我们没有办法阻止这种使用方式。为了维持向后兼容性，新标准库（C++11）类**任然允许向右值赋值**。

但是，我们可能希望自己的类中阻止这种用法。在此情况下，我们希望**强制左侧运算对象（即，`this`指向的对象）是一个左值**

我们指出`this`的左值/右值属性的方式与定义`const`成员函数相同，即，在**参数列表后放置一个引用限定符**

```cpp
class Foo1 {
public:
	Foo1& operator=(const Foo1&)& { return *this; }
};
class Foo2 {
public:
	Foo2& operator=(const Foo2&)&& { return *this; }
};
int main() {
	Foo1 i, j;
	move(i) = j;   //error
	i = j;   //ok

	Foo2 m, n;
	move(m) = n; //ok
	m = n; //error
	
}

```

# 4 对象函数指针

```cpp
class Test {
public:
	int x = 10;
	int func() {
		return x;
	}
	static int s_func() {
		return 2;
	}
};
int main() {
	Test t;
	//using fp = int(Test::*)();
	int(Test:: * fp)();
	fp = Test::func;      //函数可以取地址也可以不用取地址,函数名会隐式转换为函数指针
	int(*s_fp)();
	s_fp = Test::s_func;

	cout << (t.*fp)() << endl;
	cout << (*s_fp)() << endl;
}
```

# 5 `iostream`用法 + `ctime`

```cpp
class Solution {
public:
    int dayOfYear(string date) {
        tm dt;
        istringstream(date) >> get_time(&dt, "%Y-%m-%d");
        return dt.tm_yday + 1;
    }
};

作者：灵茶山艾府
链接：https://leetcode.cn/problems/day-of-the-year/solutions/2585579/ge-chong-yu-yan-de-ku-han-shu-xie-fa-pyt-6gdt/
来源：力扣（LeetCode）
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。
```



# 6 成员初始化列表与构造函数体内初始化的区别

> 1. **效率：**
>    - **成员初始化列表：** 通常来说，成员初始化列表更加高效。它允许在对象构造时直接初始化成员，而不是先调用默认构造函数然后再赋值。这对于一些类类型或非内置类型的成员来说可以避免不必要的默认构造和拷贝构造。**（直接构造）**
>    - **构造函数体内初始化：** 在构造函数体内进行初始化可能需要额外的步骤，例如调用默认构造函数再赋值，这可能会引入一些性能开销。**（先默认构造，再赋值）**
> 2. **可用性：**
>    - **成员初始化列表：** 不是所有的成员都可以在构造函数体内初始化，尤其是对于**const**和**引用类型**的成员，它们通常只能通过成员初始化列表进行初始化。
>    - **构造函数体内初始化：** 更加灵活，可以用于几乎所有类型的成员初始化。
> 3. **初始化顺序：**
>    - **成员初始化列表：** 在成员初始化列表中，成员的初始化顺序是由它们在**类中的声明顺序决定**的，而不是由成员初始化列表中的顺序决定。
>    - **构造函数体内初始化：** 成员初始化顺序取决于在构造函数体内的实际初始化语句的顺序。

# 7 union的构造函数默认删除

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

# 8 懒汉模式静态局部变量初始化为线程安全的

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



# 9 C++解决循环依赖

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



# 10 函数模板的声明和定义需要在同一文件中

问：为什么下述代码报错了？

```cpp
//test_template.h
template<typename T>
T add(T a, T b);

//test_template.cpp
#include "test_template.h"
template<typename T>
T add(T a, T b) {
	return a + b;
}

//test.cpp
#include "test_template.h"
int main() {
	cout << add<int>(1, 2) << endl;
}

```

答：

> 在C++中，模板的定义和声明通常需要放在同一个文件中，因为模板是编译时生成代码的一种机制。在你的代码中，模板函数 `add` 的声明和定义分别在 `test_template.h` 和 `test_template.cpp` 中，这会**<u>导致链接错误。</u>**
>
> 要解决这个问题，可以将模板函数的声明和定义统一放在同一个文件中。你可以将 `test_template.h` 和 `test_template.cpp` 合并为一个文件，或者将模板函数的定义直接放在 `test.cpp` 中，如下所示：
>
> ```cpp
> cppCopy code// test_template.h
> template<typename T>
> T add(T a, T b) {
>     return a + b;
> }
> 
> // test.cpp
> #include "test_template.h"
> int main() {
>     cout << add<int>(1, 2) << endl;
> }
> ```

# 11 不完全类型

在C++中，不完全类型（Incomplete Type）是指在某一点上还不完全定义的类型。这意味着编译器不知道这种类型的大小或者其他关键细节。不完全类型通常出现在以下几种情况下：

1. **类的前向声明（Forward Declaration）**：在类的定义之前声明这个类，但不提供具体的定义。这样就可以在声明之后使用这个类的指针或引用，而不需要知道类的具体实现细节。

   ```cpp
   cppCopy codeclass MyClass; // 前向声明
   
   void function(MyClass* ptr); // 可以使用MyClass的指针，但不能使用MyClass的对象
   ```

2. **数组的大小未知**：在声明数组时，如果指定了数组的大小但是没有提供具体的类型，那么这个类型就是不完全类型。

   ```cpp
   cppCopy code
   void function(int array[]); // 不完全类型，因为数组的大小未知
   ```

3. **递归类型**：当一个类型包含自己的指针或引用时，就是不完全类型。在编译器能够确定类型的大小之前，这种类型被认为是不完全类型。

   ```cpp
   cppCopy codestruct Node {
       int data;
       Node* next; // Node类型的指针，Node是不完全类型
   };
   ```

在使用不完全类型时，需要注意避免引发编译错误。通常，可以使用指针或引用来避免直接使用不完全类型。

```cpp
class Delivery_boy { 
public: 
    void Pick_up_meals(std::string where, std::string which) {} 
    void Food_delivery() {} 
private: 
    static Delivery_boy p1;     //静态成员可以是不完全类型  
};
```

上述代码为什么是正确的?

> 上述代码是正确的，因为在类内部声明静态成员时，该静态成员可以是不完全类型。在这种情况下，**<u>只有声明，没有定义，所以编译器不需要知道该类型的具体细节</u>**，只需要知道它是一个静态成员即可。这种情况下，静态成员的定义需要在类外部单独提供。



## 11.1 智能指针要求指向的内容是complete type

在C++中，`std::shared_ptr`要求指向的类型必须是完整的（complete type）。这意味着在使用`std::shared_ptr`时，必须提供完整的类型定义，编译器必须知道该类型的大小和结构，以便正确地管理内存。

如果尝试将不完整类型传递给`std::shared_ptr`，编译器会发出错误，因为`std::shared_ptr`需要完整的类型信息来进行内存管理。例如，在尝试这样做时会导致编译错误：

```cpp
cppCopy codestruct Node {
    int data;
    std::shared_ptr<Node> next; // 编译错误：不能使用不完整类型
};
```

如果需要使用`std::shared_ptr`管理不完整类型的对象，通常需要使用指针或者其他手段来间接管理这些对象的生命周期。
