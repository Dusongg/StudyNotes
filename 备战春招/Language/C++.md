

# 关键字

## move实现

std::move **并不会真正移动对象**，它只是**将对象转换为右值引用**，以便可以在后续代码中**触发移动语义**。



```cpp
// C++ 标准库中的 move 实现（位于 <utility> 头文件）
template <typename T>
constexpr typename std::remove_reference<T>::type&& move(T&& t) noexcept {
    return static_cast<typename std::remove_reference<T>::type&&>(t);
}
```

- std::remove_reference<T>::type 是 C++ 标准库中的一个**类型萃取（Type Trait）**，它的作用是**去除类型 T 上的引用**，无论是左值引用 T& 还是右值引用 T&&，都返回**原始类型（无引用版本）**。<u>（通过模版偏特化实现）</u>

  ```cpp
  namespace std {
      template <typename T>
      struct remove_reference {
          typedef T type;  // 默认情况下，返回 T 本身
      };
  
      // 偏特化：处理 T& 类型（去除左值引用）
      template <typename T>
      struct remove_reference<T&> {
          typedef T type;  // 去掉左值引用
      };
  
      // 偏特化：处理 T&& 类型（去除右值引用）
      template <typename T>
      struct remove_reference<T&&> {
          typedef T type;  // 去掉右值引用
      };
  }
  ```

  

## inline

1. **减少函数调用开销**：作用于函数定义，类似于宏替换，将函数体直接插入到调用处来优化小函数的执行效率，

2. **避免多重定义问题**：

   当多个源文件包含同一个函数的定义时，链接器报错，

   使用inline解决，因为 inline 函数的定义会在每个调用点被直接替换为函数体（即内联），而不是像普通函数那样进行正常的函数调用。

- 缺点/局限：

  **编译器控制**：虽然使用 inline 给编译器一个提示，但编译器**不一定会遵循**。在某些情况下，编译器可能会忽略 inline 关键字，特别是当函数较复杂时，编译器会避免内联，以防止代码膨胀。

- 与宏函数的区别：

  类型检查、错误处理（inline函数错误会在编译期报错）、作用域

##  static全局/普通全局/ static局部/普通局部

1. 静态全局变量和普通全局变量的关系：
   - **作用域**：静态全局变量的作用域限于定义它的文件，即在文件内部可见，在文件外部不可见。普通全局变量的作用域是整个程序。
   - **生命周期**：静态全局变量的生命周期与程序的生命周期相同，而普通全局变量也是如此。
   - **存储位置**：静态全局变量存储在静态存储区，普通全局变量也存储在静态存储区，但是它们的链接属性不同。静态全局变量的链接属性是内部链接（internal linkage），只能在定义它的文件内部访问，而普通全局变量的链接属性是外部链接（external linkage），可以在其他文件中使用 `extern` 关键字来访问。
2. 静态局部变量和普通局部变量的关系：
   - **作用域**：静态局部变量和普通局部变量的作用域相同，都限于定义它们的代码块内部。
   - **生命周期**：**<u>静态局部变量的生命周期与程序的生命周期相同</u>**，而普通局部变量的生命周期与其所在的代码块的执行周期相同，即离开代码块就会被销毁。
   - **存储位置**：普通局部变量存储在栈上，静态局部变量存储在静态存储区





##  `dynamic_cast` 和 `static_cast`的区别

![image-20240331212119133](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240331212119133.png)

> 1. **静态类型转换(static_cast)**：
>    - `static_cast`在编译时进行，用于通常的转换操作，如基本数据类型之间的转换，非const指针转换为const指针，以及父类指针转换为子类指针（不安全）。
>    - `static_cast`不进行<u>运行时类型检查</u>，因此在执行转换时可能会导致不安全的类型转换。
> 2. **动态类型转换(dynamic_cast)**：
>    - `dynamic_cast`在运行时进行，用于类层次结构中的向下转换（将父类指针或引用转换为子类指针或引用）。
>    - `dynamic_cast`会检查转换是否有效，如果转换失败（即指针或引用不指向子类对象），则返回nullptr（对于指针）或抛出std::bad_cast异常（对于引用）。
>    - `dynamic_cast`只能用于含有虚函数的类，因为它需要在运行时检查对象的实际类型。
>
> 因此，`static_cast`和`dynamic_cast`的选择取决于转换的类型和安全性需求。如果可以在编译时确定转换是安全的，并且不需要运行时检查，可以使用`static_cast`。如果需要在运行时检查转换的有效性，并且转换涉及到类的继承关系，应该使用`dynamic_cast`。

```cpp
int main() {
    Base* b1 = new Base;
    if (Derived* d = dynamic_cast<Derived*>(b1); d != nullptr)
    {
        std::cout << "成功从 b1 向下转换到 d\n";
        d->name(); // 可以安全调用
    }
 
    Base* b2 = new Derived;
    if (Derived* d = dynamic_cast<Derived*>(b2); d != nullptr)
    {
        std::cout << "成功从 b2 向下转换到 d\n";
        d->name(); // 可以安全调用
    }
 
    delete b1;
    delete b2;
}
```

### 

## auto关键字

-  **被auto定义的变量必须被初始化**

-  使用auto关键字声明变量的类型，不能自动推导出`const`、`volatile`，也不能自动推导出引用类型

```cpp
int main() {
	int a = 10;
	const int& b = a;
	decltype(b) c = 10;   //c的类型为const int&
	auto d = b;           //d的类型为int
}
```



- **推导规则**

1️⃣ **编译时推导**

auto 类型推导发生在**编译期**，不会影响运行时性能。

编译器会根据变量的初始化值推导类型，因此 auto 变量在**定义时必须初始化**。



2️⃣ **推导忽略 const 和 &**（但 auto& 会保持引用）

```
const int x = 100;
auto y = x;   // y 是 int（去掉了 const）
auto& z = x;  // z 是 const int&（引用保持 const）
```

3️⃣ **函数返回值推导 (auto f()->类型)**

```
auto add(int a, int b) -> int {
    return a + b; 
}
```

- **auto 的缺点**

1️⃣ **可读性降低**

​	•	auto 可能导致代码可读性变差，特别是在复杂类型时：

```
auto it = myMap.begin();  // 它是 `std::map<int, std::string>::iterator`？
```

需要知道 myMap 的类型才能确定 it 的类型。



2️⃣ **推导失败**

​	•	auto 变量**必须初始化**，否则会报错：

```
auto x;  // ❌ 错误：必须初始化
```

3️⃣ **数组和 std::initializer_list 的问题**

```
auto arr = {1, 2, 3}; // 推导为 `std::initializer_list<int>` 而不是 `int[3]`
```

4️⃣ **无法用于函数参数**

```
void foo(auto x) { } // ❌ C++14 之前不支持
```

**适用场景**

✅ 适合推导**冗长的类型**（如迭代器、Lambda 表达式）

✅ 适合泛型编程（配合 decltype、template）

🚫 **不适合简单类型**（如 int、double）





## new/malloc/delete/free区别

###  `new`表达式调用的详细步骤

> 1. 调用一个名为`operator new`（或者`operator new[]`）的标准库函数，该函数分配一块足够大的、原始的、未命名的内存空间以便存储特定类型的对象（或者对象数组）。
>
> 2. 编译器运行相应的构造函数以构造这些对象，并为其传入初始值。
> 3. 对象被分配了空间并构造完成，返回一个指向该对象的指针

### `delete`表达式调用的详细步骤

> 1. 对sp所指的对象或者arr所指的数组中的元素执行对应的析构函数
> 2. 编译器调用名为`operator delete`（或者`operator delete[]`）的标准库函数释放内存空间

- 应用程序可以在全局作用域中定义`operator new`函数和`operator delete`函数，也可以将他们定义为成员函数，如果被分配（释放）对象是类类型，则编译器**首先在类以及基类的作用域中查找**
- 并且我们可以通过作用域限定符令`new`或`delete`表达式直接执行全局作用域中的版本

```cpp
//删除类中的new和delete，并使用全局的new和delete
class Test {
public:
	Test() = default;
	Test(int x) : ele(x) {}
	void* operator new(size_t size) = delete;
	void operator delete(void* p) = delete;
private:
	int ele;
};
int main() { 
	Test* t = new Test();    //error:无法调用已删除的函数
	Test* t = ::new Test();   //使用全局作用域的new

	delete t;     //error
	::delete t;    //使用全局作用域的delete
}
```

### [`operator new`接口和`operator delete`接口](https://en.cppreference.com/w/cpp/memory/new/operator_new)

标准库定义了`operator new`函数和`operator delete`函数重载的8个重载版本。 

1. 如果将operator new 或 operator delete 定义成类的成员函数时，他们是**==隐式静态==**的，因为`operator new`发生在对象构造之前，也就是说此时只能由类去调用，同样`operator delete`发生在对象析构之后
2. 调用`operator new`或`operator new[]`时，第一个参数类型必须是`size_t`，且**不能有默认实参**，它表名存储该对象所需的字节数或存储数组中所有元素所需要的总空间
3. 不允许重载`void* operator new(size_t, void*);`





# C++11

## 引用

### 左值 & 右值

左值是**可以取地址（即能用 & 取出指针）的对象**，通常是**变量**、**引用**或**返回左值引用的表达式**。

右值可以进一步细分为：

- **纯右值（PRvalue，Pure Rvalue）**：

​	•	**字面量**：10, 3.14, "hello"

​	•	**表达式结果**：a + b, func()

​	•	**临时对象**：std::string("abc")

- **将亡值（Xvalue，Expiring Value）**：

​	•	**即将被销毁的对象**，如 std::move(obj)



#### 移动语义

什么是：它通过将资源的所有权从一个对象“转移”到另一个对象，而不是复制资源的内容，从而避免了不必要的资源复制，提高程序的效率

**转让所有权**，如移动构造、移动赋值



```cpp
class MyClass {
private:
    int* data;
public:
    MyClass(int val) : data(new int(val)) {}  // 普通构造函数
    ~MyClass() { delete data; }

    // 移动构造函数
    MyClass(MyClass&& other) noexcept : data(other.data) {
        other.data = nullptr;  // 使原对象的指针为空，防止析构时重复删除资源
    }

    // 移动赋值运算符
    MyClass& operator=(MyClass&& other) noexcept {
        if (this != &other) {
            delete data;  // 先释放当前对象的资源
            data = other.data;  // 交换资源
            other.data = nullptr;  // 防止析构时重复释放
        }
        return *this;
    }
};
```



- 移动构造不抛出异常

如果重新分配过程中使用了移动构造函数，且在移动了部分而不是全部元素后抛出了一个异常，就会产生问题：旧空间中的移动源元素已经被改变了，而新空间中未构造的元素可能尚不存在。在此情况下，`vector`将不能满足自身保持不变的要求

![image-20231106215633750](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231106215633750.png)



#### 完美转发 & 引用折叠

- 引用折叠

为了防止引用的嵌套：

T& &  → T&

T& && → T&

T&& & → T&

T&& && → T&&



- 完美转发

  完美转发允许一个函数**将参数“原封不动”地传递给另一个函数**，避免**不必要的拷贝**或**错误的类型推导**。

```cpp
void print(int& x) { std::cout << "Lvalue: " << x << std::endl; }
void print(int&& x) { std::cout << "Rvalue: " << x << std::endl; }

template <typename T>
void wrapper(T&& arg) {
    print(std::forward<T>(arg));  // 完美转发
}

int main() {
    int a = 10;
    wrapper(a);      // Lvalue 传入，print(int&) 调用
    wrapper(20);     // Rvalue 传入，print(int&&) 调用
}
```



#### 万能引用/ const引用

- 万能引用：T&& 在模板参数推导中，可以匹配左值和右值。**必须结合模板** 才能发挥作用
- const引用：const T& 可以绑定**左值**和**右值**。绑定右值时，const T& 可以延长右值的生命周期。**不能修改** 绑定的对象



### 引用 vs 指针

1️⃣初始化 2️⃣安全性 3️⃣访问方式





## lambda相关

创建一个函数闭包，包括<u>捕获列表、参数列表、返回值、函数体</u>(参数列表和返回值可以省略)，编译器会自动生成一个匿名类（这个类重载了`operator()`）

- 捕获列表

- lambda类型：uuid

- 悬垂引用, 通过值传递解决

  ```cpp
  #include <iostream>
  #include <functional>
  
  std::function<void()> getLambda() {
      int x = 42;
      return [&]() {  // 以引用捕获 x
          std::cout << x << std::endl;  // ❌ x 在 getLambda() 结束时被销毁
      };
  }
  
  int main() {
      auto lambda = getLambda();  // 返回的 Lambda 仍然持有对 x 的引用
      lambda();  // ❌ 悬垂引用，未定义行为
  }
  ```

- C++14支持范型Lambda

  ```cpp
  auto add = [](auto a, auto b) { return a + b; };
  std::cout << add(3, 4) << std::endl;    // 输出 7
  std::cout << add(3.5, 2.5) << std::endl; // 输出 6.0
  ```

  





## emplace

涉及：模版可变参数、万能引用、完美转发、定位new构造

![-19dbe4eb6d3f7cc5](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/-19dbe4eb6d3f7cc5.png)

### `emplace_back`与`push_bach`区别



## 智能指针

```cpp
shared_ptr<int> p(new int(1024));

shared_ptr<int> p2 = new int(1024); //errror:不存在从int*到shared_ptr<int>的适当构造函数
```

### `shared_ptr`简单实现

```cpp
#include <iostream>
#include <atomic>

template <typename T>
class SimpleSharedPtr {
private:
    struct ControlBlock {  // 控制块
        T* ptr;
        std::atomic<size_t> ref_count;

        ControlBlock(T* p) : ptr(p), ref_count(1) {}
        ~ControlBlock() { delete ptr; }
    };

    ControlBlock* control;  // 指向控制块

public:
    // ✅ 默认构造
    explicit SimpleSharedPtr(T* p = nullptr) {
        if (p) {
            control = new ControlBlock(p);
        } else {
            control = nullptr;
        }
    }

    // ✅ 拷贝构造（增加引用计数）
    SimpleSharedPtr(const SimpleSharedPtr& other) {
        control = other.control;
        if (control) {
            ++(control->ref_count);
        }
    }

    // ✅ 移动构造（转移控制权）
    SimpleSharedPtr(SimpleSharedPtr&& other) noexcept {
        control = other.control;
        other.control = nullptr;  // 让 other 失效
    }

    // ✅ 赋值运算符（避免自赋值）
    SimpleSharedPtr& operator=(const SimpleSharedPtr& other) {
        if (this == &other) return *this;

        release();  // 释放旧资源

        control = other.control;
        if (control) {
            ++(control->ref_count);
        }
        return *this;
    }

    // ✅ 移动赋值运算符
    SimpleSharedPtr& operator=(SimpleSharedPtr&& other) noexcept {
        if (this == &other) return *this;

        release();  // 释放旧资源

        control = other.control;
        other.control = nullptr;  // 让 other 失效
        return *this;
    }

    // ✅ 析构函数
    ~SimpleSharedPtr() {
        release();
    }

    // ✅ 获取原始指针
    T* get() const { return control ? control->ptr : nullptr; }

    // ✅ 重载 * 和 -> 访问对象
    T& operator*() const { return *(control->ptr); }
    T* operator->() const { return control->ptr; }

    // ✅ 返回引用计数
    size_t use_count() const { return control ? control->ref_count.load() : 0; }

private:
    void release() {
        if (control && --(control->ref_count) == 0) {
            delete control;  // 释放控制块
            control = nullptr;
        }
    }
};
```



# 面向对象

## 虚继承原理

- 菱形继承带来的问题：数据冗余与二义性问题

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/53fb409c9bbc44198d0d7d273930cd8a.png)

虚基类将自己的基类子对象存储在了派生类内存的一块共享位置，并通过自己的虚基表指针找到虚基表中的偏移量来访问该地址，以达到菱形继承造成的数据冗余和二义性问题



## 用C实现继承

```cpp
//C语言模拟C++的继承与多态

typedef void (*FUN)();   //定义一个函数指针来实现对成员函数的继承

struct _A    //父类
{
  FUN _fun;  //由于C语言中结构体不能包含函数，故只能用函数指针在外面实现
  int _a;
};

struct _B     //子类
{
  _A _a_;   //在子类中定义一个基类的对象即可实现对父类的继承

  int _b;
};

void _fA()    //父类的同名函数
{
  printf("_A:_fun()\n");
}

void _fB()    //子类的同名函数
{
  printf("_B:_fun()\n");
}

void Test()
{
  //C语言模拟继承与多态的测试

  _A _a;  //定义一个父类对象_a

  _B _b;  //定义一个子类对象_b

  _a._fun = _fA;    //父类的对象调用父类的同名函数

  _b._a_._fun = _fB;  //子类的对象调用子类的同名函数



  _A* p2 = &_a;  //定义一个父类指针指向父类的对象

  p2->_fun();   //调用父类的同名函数

  p2 = (_A*)&_b; //让父类指针指向子类的对象,由于类型不匹配所以要进行强转

  p2->_fun();   //调用子类的同名函数

}
```



## 什么是多态

多态可分为**动态多态**和**静态多态：**

**动态多态：程序能通过引用或指针的动态类型获取类型特定行为的能力**

> 当我们使用基类的引用或指针调用基类中定义的一个函数是，我们并不知道该函数真正作用的对象是什么类型，因为它可能是一个基类的对象也可能是一个派生类的对象。如果该函数时虚函数，则直接运行时才会决定到底执行哪个版本，判断的依据是引用或指针所绑定的对象的真实类型；      (C++ Primer   P537）

静态多态：例如函数重载、运算符重载、函数模板等 



## 重载/重写/隐藏(重定义)

**重载**：两函数处于**同一作用域**；函数名相同；参数不同

**覆盖(重写)**：两函数分别位于基类和派生类作用域；**函数名、参数、返回值都相同**(协变除外)；两函数均为虚函数

**隐藏(重定义)**：两函数分别位于基类和派生类作用域；**函数名相同**；



## 多态原理



![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/f286db550ba44b1eb7813efecdf4204d.png)

### 虚函数表存放在内存的哪个区域

C++中**虚函数表位于只读数据段（.rodata）；而虚函数则位于代码段（.text）。**

### 虚函数表和虚函数表指针是在哪个阶段初始化的

**1️⃣ 虚函数表（vtable）**

**编译期生成，运行期不变**，用于存储该类的 **虚函数地址**，**继承时**，子类的 **vtable** 会 **覆盖** 父类的虚函数地址（若重写）。

**2️⃣ 虚函数表指针（vptr）**

**运行期，由构造函数初始化**





## 虚函数实现原理





## 类/对象内存布局

- **非静态/静态成员函数**：代码区

- 虚函数代码：代码区
- 虚函数表（vtable）：只读数据段（编译时确定）
- 虚函数表指针（vptr）：对象内存中。

## class与struct的区别

1. 默认继承方式和默认访问权限
2. 使用习惯区别



## 为什么析构函数写为虚函数，构造函数不能为虚函数

**1️⃣ 为什么析构函数要是虚函数？**

析构函数通常被声明为虚函数，以确保在 **基类指针或引用删除派生类对象时，派生类的析构函数能被正确调用**，从而避免 **内存泄漏或未释放资源**。



**2️⃣ 为什么构造函数不能是虚函数？**

**对象的构造顺序不支持虚函数调用**，在 C++ 中，**对象的构造是从基类到派生类** 逐步进行的。在构造基类时，**虚函数表（vtable）还未初始化**，虚函数的调用机制依赖于 vtable，而 vtable 只有在 **基类构造函数执行完** 之后，才能初始化为派生类的 vtable。

# C++14/17/20





# STL

## array/int[]/vector 

```
int a[] = {1, 2, 3};
int b[] = a; // ❌ 错误，数组不能整体赋值
```

```
std::array<int, 3> a = {1, 2, 3};
std::array<int, 3> b = a;  // ✅ 允许整体赋值
```



| **特性**             | std::vector<int>              | int[]**（C 数组）** | std::array<int, N>         |
| -------------------- | ----------------------------- | ------------------- | -------------------------- |
| **大小**             | 动态，可变                    | 固定，编译期确定    | 固定，编译期确定           |
| **存储位置**         | 堆（new）                     | 栈（或手动 new）    | 栈（类似 C 数组）          |
| **功能**             | 丰富（STL 成员函数）、范围for | 仅基本操作          | 支持 size()、赋值、范围for |
| **能否整体赋值**     | ✅ 支持                        | ❌ 不支持            | ✅ 支持                     |
| **能否 push_back()** | ✅ 支持                        | ❌ 不支持            | ❌ 不支持                   |

> [!WARNING]
>
> std::vector 和 std::array 在赋值时**默认都是深拷贝**，也就是说它们会复制所有的元素，而不是仅仅复制指针或引用。





# 其他

## `malloc`是线程安全的吗？

在标准C中，`malloc`函数本身不是线程安全的，因为它在多线程环境下无法保证分配内存的正确性。如果在多线程程序中使用`malloc`，可能会导致竞态条件或内存错误。

为了在多线程环境下安全地使用`malloc`，可以考虑以下方法：

1. 使用线程库提供的线程安全的内存分配函数，如`pthread_alloc`。
2. 在使用`malloc`之前使用互斥锁来保护内存分配的操作，确保同一时间只有一个线程在分配内存。

另外，C11标准引入了`<threads.h>`头文件，提供了一组线程安全的内存分配函数，如`aligned_alloc`和`calloc`。

### malloc/calloc/realloc区别

1️⃣ malloc(size_t size)

2️⃣ calloc(size_t num, size_t size)：**初始化为 0**，线程安全

3️⃣ realloc(void *ptr, size_t new_size)：整 ptr 指向的已分配内存的大小，并可能搬移数据。



## 源文件生成可执行文件的步骤

> C++文件的编译过程可以分为四个主要阶段：预处理（Preprocessing）、编译（Compilation）、汇编（Assembly）和链接（Linking）。以下是每个阶段的具体工作：
>
> 1. **预处理（Preprocessing）**：在这个阶段，预处理器会处理源文件，执行以`#`开头的预处理指令。预处理指令可以包括`#include`（包含头文件）、`#define`（定义宏）、`#ifdef`、`#ifndef`、`#endif`等。预处理器的主要作用是<u>**展开宏、包含头文件内容，并删除注释**</u>等。处理后的文件通常是以`.i`或`.ii`为扩展名的中间文件。
>
>    ```bash
>    g++ -E main.cpp -o main.i
>    ```
>
> 2. **编译（Compilation）**：在这个阶段，编译器将预处理后的文件翻译成汇编语言。编译器会进行<u>**语法分析、语义分析、优化等**</u>操作，生成相应的汇编代码。生成的汇编代码通常以`.s`为扩展名。
>
>    ```bash
>    g++ -S main.i -o main.s
>    ```
>
> 3. **汇编（Assembly）**：汇编器将汇编代码<u>**翻译成机器可执行的指令码**</u>。汇编器生成的文件通常以`.o`为扩展名，也称为目标文件。
>
>    ```bash
>    g++ -c main.s -o main.o
>                      
>    objdump -d main.o  #机械码文件通过objdump查看
>    ```
>
>    
>
> 4. **链接（Linking）**：链接器将目标文件与所需的库文件链接在一起，生成最终的可执行文件。链接器的工作包括符号解析、重定位等。最终生成的可执行文件通常没有特定的扩展名（在Linux中通常没有扩展名，在Windows中可能是`.exe`）。
>
>    ```bash
>    g++ main.o -o main
>    ```



## 内存对齐规则与原因





## 友元类/友元函数

访问类中的私有成员/私有函数

⚠️注意点：

- 友元关系不被继承
- 不具有**交换性**与**传递性**



## C++与其他语言的区别

### 与C的区别

1️⃣new/delete 2️⃣引用 3️⃣面向对象 4️⃣异常处理 5️⃣STL 6️⃣命名空间

### 与Go的区别



## 空对象大小，为什么？ —— 对比go的`struct{}`

**空类占1个字节**（类和对象大小通常相等）

原因：

- 保证每个对象有独立的地址，
- 保证内存布局合理，比如对象数组
- 1个字节不存放任何东西

> [!NOTE]
>
> **特殊情况：继承可能导致空类占 0 字节**
>
> - 空基类优化——〉<u>C++ 允许空基类在子类中不额外占据空间，避免浪费内存。</u>
>
>   ```cpp
>   #include <iostream>
>         
>   class Empty {};
>   class Derived : public Empty {};
>         
>   int main() {
>       std::cout << "Size of Derived: " << sizeof(Derived) << " bytes" << std::endl;   //Size of Derived: 1 bytes
>       return 0;
>   }
>   ```
>
>   



### 为什么go的空结构体大小为0

Go 设计 struct{} 的初衷是**高效、简洁**，如果结构体不包含任何字段，它**不应该浪费任何内存**。这与 C++ 的设计不同，C++ 强制**对象必须有唯一的地址**，而 Go 不要求这一点。

- 空结构体用途：（节省空间）
  - map-》set
  - 在channel中传递信号，不传输值
