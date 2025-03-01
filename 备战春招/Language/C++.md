

# 关键字 / 函数

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

   使用inline解决，<u>因为 inline 函数的定义会在每个调用点被直接替换为函数体（即内联）</u>，而不是像普通函数那样进行正常的函数调用。

- 缺点/局限：

  **编译器控制**：虽然使用 inline 给编译器一个提示，但编译器**不一定会遵循**。在某些情况下，编译器可能会忽略 inline 关键字，特别是当函数较复杂时，编译器会避免内联，以防止代码膨胀。

- 与宏函数的区别：

  类型检查、错误处理（inline函数错误会在编译期报错）、作用域

3. **inline变量（C++17）**

​	使用inline修饰的变量允许在多个翻译单元中定义，但只会有一个实例存在。链接器会确保只有<u>一个变量实例被定义</u>。

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



## auto关键字

-  **被auto定义的变量必须被初始化**

-  使用auto关键字声明变量的类型，**不能自动推导出`const`、`volatile`，也不能自动推导出引用类型**

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



# C++11

## 引用

### 左值 & 右值

左值是**可以取地址（即能用 & 取出指针）的对象**，表示对变量或者对象的引用

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

1️⃣初始化 2️⃣安全性 3️⃣访问方式 4️⃣sizeof计算

- **最佳实践**

1. **默认使用引用**：优先用引用传递参数或返回局部对象，提升代码安全性和可读性。
2. **必要时用指针**：动态内存、可选参数、多态操作或兼容 C 代码时使用指针。
3. **避免原始指针**：现代 C++ 中优先用 `智能指针（unique_ptr/shared_ptr）` 管理资源，避免内存泄漏。



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

**静态多态：例如函数重载、运算符重载、函数模板等** 



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

### 哪些函数不能是虚函数

| **函数**                    | **能否是虚函数？** | **原因**                                 |
| --------------------------- | ------------------ | ---------------------------------------- |
| **构造函数**                | ❌ 不能             | 对象构造时，vtable 未建立                |
| **析构函数**                | ✅ 可以             | 避免基类指针删除派生类对象时发生内存泄漏 |
| **静态成员函数**            | ❌ 不能             | 无 this 指针，不存入 vtable              |
| **内联函数**                | ⚠️ 可以，但没意义   | 虚函数通常通过 vtable 调用，无法真正内联 |
| **友元函数**                | ❌ 不能             | 友元函数不属于类成员，无法存入 vtable    |
| **模板函数**                | ⚠️ 可以，但不推荐   | 模板实例化生成不同代码，不适用于多态     |
| **::、.\*、sizeof、typeid** | ❌ 不能             | 这些操作与类的 vtable 无关               |
| **纯虚析构函数**            | ✅ 但必须提供定义   | 编译器需要生成默认析构函数               |

✅ **最佳实践**：

​	•	**析构函数建议设为 virtual**，防止基类指针删除派生类对象时导致内存泄漏。

​	•	**避免 virtual static、virtual friend 和 virtual template，因为它们违反 C++ 语法规则或设计原则。**

​	•	**纯虚析构函数必须提供定义，否则会导致编译错误。**



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

# 🆕🌟C++14/17/20

### C++14

对于C++14，它是对C++11的小幅改进，增加了一些便利特性。比如泛型lambda、变量模板、返回类型推导等。

1. **泛型 Lambda**

   ```cpp
   auto lambda = [](auto x, auto y) { return x + y; };  // 支持自动类型推导
   ```

   - **用途**：编写类型无关的通用 Lambda 表达式

2. **变量模板**

   ```cpp
   template<typename T> constexpr T pi = T(3.1415926);  
   float r = pi<float>;  // 类型明确的常量定义
   ```

   - **用途**：类型相关的常量/元编程

3. **返回类型推导 (`auto` 函数)**

   ```cpp
   auto add(int a, int b) { return a + b; }  // 自动推导返回类型
   ```

   - **用途**：简化模板函数和复杂返回类型的书写

### C++17

C++17引入了更重要的特性，如结构化绑定、if constexpr、内联变量、文件系统库等

1. **结构化绑定**

   ```cpp
   std::tuple<int, string> data{42, "test"};
   auto [id, name] = data;  // 直接解包到变量
   ```

   - **用途**：简化多返回值处理（替代 `std::tie`）

2. **`if constexpr` 编译期分支**

   ```cpp
   template<typename T> void process(T val) {
       if constexpr (std::is_integral_v<T>) { /* 整数处理 */ }
       else { /* 其他类型处理 */ }
   }
   ```

   - **用途**：模板元编程中的条件编译

3. **内联变量**

   ```cpp
   // 头文件中定义
   inline constexpr double gravity = 9.8;  
   ```

   - **用途**：解决头文件多次包含问题（替代 `const static`）

4. **`std::optional` 可选值**

   ```cpp
   std::optional<int> find(int key) { 
       if (found) return value;  
       else return std::nullopt;
   }
   ```

   - **用途**：明确表达可能存在或不存在的值

5. **`std::filesystem` 文件系统库**

   ``` cpp
   namespace fs = std::filesystem;
   fs::create_directory("data");
   for (auto& entry : fs::directory_iterator(".")) { /* 遍历文件 */ }
   ```

   - **用途**：跨平台文件操作

### C++20

C++20带来了更大的变化，比如概念(concepts)、协程(coroutines)、范围库(ranges)、模块(modules)等，这些虽然有些还在逐渐被编译器支持，但已经被广泛讨论和应用。

1. **概念 (Concepts)**

   ``` cpp
   template<typename T> concept Number = requires(T a) { a + a; };
   template<Number T> T add(T a, T b) { return a + b; }  // 约束模板类型
   ```

   - **用途**：增强模板类型约束，提升编译错误可读性

2. **协程 (Coroutines)**

   ``` cpp
   generator<int> sequence() {
       for (int i=0; ; ++i) co_yield i;  // 生成无限序列
   }
   ```

   - **用途**：异步编程、惰性求值（需编译器支持）

3. **范围库 (Ranges)**

   ``` cpp
   auto even = views::filter([](int x){ return x%2 ==0; });
   for (int x : vec | even | views::take(3)) { /* 取前3个偶数 */ }
   ```

   - **用途**：声明式集合操作（替代传统循环）

4. **模块 (Modules)**

   ``` cpp
   // math.ixx
   export module math;
   export int add(int a, int b) { return a + b; }
   
   // main.cpp
   import math;  // 替代头文件包含
   ```

   - **用途**：加快编译速度，解决头文件包含顺序问题

5. **三路比较运算符 (`<=>`)**

   ``` cpp
   struct Point { 
       int x, y;
       auto operator<=>(const Point&) const = default; // 自动生成比较操作
   };
   ```

   - **用途**：简化自定义类型的比较运算符重载

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
| **安全性**           |                               |                     |                            |



> [!WARNING]
>
> std::vector 和 std::array 在赋值时**默认都是深拷贝**，也就是说它们会复制所有的元素，而不是仅仅复制指针或引用。



## array和list的区别

- 存储空间
- 存储方式——》缓存友好性
- 应用场景：list用于插入删除操作多， array随机访问



## 讨论C++的map和unordered_map的区别，谈一谈心得

底层实现， 效率， 稳定性， 有序无序

> **1️⃣ std::map 和 std::unordered_map 的区别**
>
> **底层实现**
>
> ​	•	**std::map**：底层通常使用**红黑树**（一种平衡二叉搜索树）来存储元素。这意味着map中的元素是有序的，按照键值进行排序。
>
> ​	•	**std::unordered_map**：底层使用**哈希表**来存储元素，元素是无序的，哈希表根据哈希函数来决定元素的存储位置。
>
> **键的顺序**
>
> ​	•	**std::map**：会自动按键的顺序（根据键的比较操作符）对元素进行排序。默认是升序排列，可以自定义排序方式。
>
> ​	•	**std::unordered_map**：元素的顺序是基于哈希函数计算的，因此它不保证任何顺序。插入顺序和遍历顺序可能不同。
>
> **查找效率**
>
> ​	•	**std::map**：由于底层是平衡二叉树，查找、插入、删除操作的时间复杂度是**O(log n)**。
>
> ​	•	**std::unordered_map**：由于使用哈希表，查找、插入和删除操作的平均时间复杂度是**O(1)**。但是，在哈希冲突严重的情况下，最坏情况下复杂度为**O(n)**。
>
> **内存使用**
>
> ​	•	**std::map**：由于底层是树结构，每个元素除了存储数据外，还需要额外的指针来表示树的结构，因此内存开销相对较大。
>
> ​	•	**std::unordered_map**：由于是哈希表实现，哈希表内部需要额外的内存来存储桶（bucket）和解决冲突，因此也会有一定的内存开销，但一般来说，unordered_map的内存开销要比map小。
>
> **排序**
>
> ​	•	**std::map**：自动排序，支持有序遍历。
>
> ​	•	**std::unordered_map**：元素无序，不能按顺序遍历。
>
> **操作稳定性**
>
> ​	•	**std::map**：插入操作是**稳定的**，因为插入时会维护有序性。
>
> ​	•	**std::unordered_map**：插入操作可能会重新调整哈希表的结构，因此性能不稳定，尤其在负载因子较高时。
>
> **2️⃣ 适用场景的选择**
>
> ​	•	**std::map**：
>
> ​	•	需要有序存储和按顺序遍历的场景，例如需要范围查询、排序或者查找大于或小于某个键的元素时。
>
> ​	•	当对元素的顺序要求严格时，使用map较为合适。
>
> ​	•	**std::unordered_map**：
>
> ​	•	需要快速查找、插入、删除的场景，特别是当不需要保持元素顺序时。比如频繁的查找操作，或者要处理大量的键值对并且对排序无要求时，unordered_map会有更高的性能表现。
>
> ​	•	当哈希冲突较少时，unordered_map可以提供接近常数时间复杂度的操作。
>
> 
>
> **3️⃣ 心得总结**
>
> ​	•	**性能上的考虑**：在很多情况下，unordered_map提供了更好的性能，尤其是当数据量大且不需要有序存储时。它的查找和插入操作时间复杂度是**O(1)**，而map是**O(log n)**。不过需要注意，unordered_map的性能很大程度上依赖于哈希函数的质量，哈希冲突过多时，unordered_map的性能会下降。
>
> ​	•	**内存与空间的权衡**：std::map由于需要维持树形结构，内存开销相对较高，而unordered_map的内存开销较小，但是由于哈希表实现的原因，也会占用一定的额外内存。
>
> ​	•	**有序与无序**：当需要元素的顺序时，使用map，它保证键值是有序的。反之，如果顺序不重要，只关心数据的快速查找、插入和删除，那么unordered_map是更合适的选择。
>
> ​	•	**哈希函数的选择**：unordered_map依赖于哈希函数的质量，选择一个合适的哈希函数是至关重要的，否则可能导致性能大幅下降。
>
> 
>
> **总结：**
>
> ​	•	**std::map**：适用于需要有序数据的场景，查找、插入、删除的时间复杂度为**O(log n)**。
>
> ​	•	**std::unordered_map**：适用于对查找、插入性能要求高且对元素顺序没有要求的场景，操作的时间复杂度为**O(1)**，但依赖于哈希函数的质量。
>
> 
>
> 根据具体的需求和场景，选择合适的容器可以大大提升程序的效率和可维护性。



# 内存相关

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



## `malloc`是线程安全的吗？

在标准C中，`malloc`函数本身不是线程安全的，因为它在多线程环境下无法保证分配内存的正确性。如果在多线程程序中使用`malloc`，可能会导致竞态条件或内存错误。

为了在多线程环境下安全地使用`malloc`，可以考虑以下方法：

1. 使用线程库提供的线程安全的内存分配函数，如`pthread_alloc`。
2. 在使用`malloc`之前使用互斥锁来保护内存分配的操作，确保同一时间只有一个线程在分配内存。

另外，C11标准引入了`<threads.h>`头文件，提供了一组线程安全的内存分配函数，如`aligned_alloc`和`calloc`。

### malloc/calloc/realloc区别

1️⃣ malloc(size_t size)

2️⃣ calloc(size_t num, size_t size)：**初始化为 0**，线程安全

3️⃣ realloc(void *ptr, size_t new_size)： ptr 指向的已分配内存的大小，并可能搬移数据。

> ### **`realloc`**
>
> - **功能**：**调整**已分配内存块的大小（扩大或缩小）。
>
> - **参数**：`void* realloc(void* ptr, size_t new_size);`
>
> - **特点**：
>
>   - 若`new_size > 原大小`：扩展内存，**新增部分未初始化**。
>   - 若`new_size < 原大小`：截断内存，剩余部分可能被释放。
>   - 若`ptr`为`NULL`：等价于`malloc(new_size)`。
>   - **可能移动内存位置**，原指针失效，需使用返回值。
>
> - **示例**：
>
>   ```c
>   int* arr = (int*)malloc(10 * sizeof(int));
>   arr = (int*)realloc(arr, 20 * sizeof(int)); // 扩展为20个int的空间
>   ```

## 🆕🌟strcpy / memcpy / memmove

### strcpy与memcpy的区别

- **strcpy**：用于复制**以 null 结尾的 C 字符串**（即 char 数组）。它会复制源字符串的每个字符，直到遇到字符串的终止符 '\0'，并将其复制到目标数组中。

- **memcpy**：用于复制**任意类型的内存数据**。它按字节复制数据，不关心数据内容的类型，也不处理字符串的结束符 '\0'。它需要指定复制的字节数。

```c
//strcpy:
char *strcpy(char *dest, const char *src);

//memcpy:
void *memcpy(void *dest, const void *src, size_t n);
```

### memcpy与memmove的区别

- memmove 能正确处理源和目标内存重叠的情况，而 memcpy 假设源和目标内存区域不重叠。

```c
void *memmove(void *dest, const void *src, size_t n) {
    unsigned char *d = dest;
    const unsigned char *s = src;
    
    if (d < s || d >= s + n) {
        // 没有重叠，正常按字节复制
        while (n--) {
            *d++ = *s++;
        }
    } else {
        // 源区域在目标区域之后，需要从后向前复制
        d += n;
        s += n;
        while (n--) {
            *(--d) = *(--s);
        }
    }
    
    return dest;
}
```

- memcpy 更高效，因为它不需要判断内存是否重叠，也没有处理内存重叠的额外逻辑。

> - **安全性**：`memmove` 是 `memcpy` 的安全替代，始终优先用于不确定内存是否重叠的场景。
> - **性能**：`memcpy` 在无内存重叠时可能更高效，但现代编译器的优化可能缩小两者的差距。
> - **底层实现**：`memmove` 通过方向选择（从前向后或从后向前复制）确保数据完整性。



## 结构体内存对齐规则与原因

1. 每个成员的起始地址必须是其自身类型大小与编译器对齐值（可通过 `#pragma pack(n)` 指定）中的较小者的整数倍。
2. 结构体的总大小必须是其最大成员对齐值的整数倍，不足时末尾填充。
3. 嵌套结构体的对齐值取其自身最大成员对齐值，而非父结构体的对齐值。

- **为什么要结构体对齐**

> 1. **硬件要求**
>    - **未对齐访问的代价**：某些处理器（如ARM、x86）对未对齐的内存访问会触发异常或需要多次内存操作。例如，读取一个4字节的 `int` 若起始于地址0x3，需两次内存读取并拼接数据。
>    - **原子性操作限制**：硬件可能仅保证对齐数据的原子性访问。
> 2. **性能优化**
>    - **缓存行利用率**：内存对齐可确保数据跨越更少的缓存行（通常64字节），减少缓存未命中。例如，一个未对齐的 `double` 可能横跨两个缓存行，导致两次缓存加载。
>    - **SIMD指令支持**：SSE/AVX等向量指令要求数据按特定对齐方式（如16/32字节）存储。
> 3. **跨平台兼容性**
>    - 不同体系结构（如x86、SPARC）的对齐要求可能不同，显式对齐可避免移植问题。例如，SPARC处理器对未对齐访问直接抛出总线错误。

## 字节序：大端 or 小端

为什么要有：不同硬件平台的架构不同，网络和主机的字节序可能也不同

1）**大端序：** 高字节存储在内存的低地址处，低字节存储在高地址处。例如，对于16进制数0x12345678，大端序在内存中的存储方式是：12 34 56 78。

2）**小端序：** 低字节存储在内存的低地址处，高字节存储在高地址处。对于同样的16进制数0x12345678，小端序在内存中的存储方式是：78 56 34 12。

```c++
bool isLittleEndian() {
    uint16_t num = 1;
    return *(reinterpret_cast<char*>(&num)) == 1;
}
```

- **字节序转换**

`htons()` 和 `htonl()` 用于将主机字节序转换为网络字节序（一般是大端），`ntohs()` 和 `ntohl()` 用于将网络字节序转换为主机字节序：

```c
#include <arpa/inet.h>
#include <cstdint>
#include <iostream>

int main() {
    uint32_t num = 0x12345678;
    uint32_t n_num = htonl(num); // Host to Network Long
    uint32_t h_num = ntohl(n_num); // Network to Host Long

    std::cout << "Original: " << std::hex << num << std::endl;
    std::cout << "Network Byte Order: " << std::hex << n_num << std::endl;
    std::cout << "Converted Back: " << std::hex << h_num << std::endl;

    return 0;
}

```



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



# 其他

## extern的作用

- **声明变量或函数**：extern 关键字告诉编译器，这个变量或函数在其他地方定义，并且可以在当前文件中使用。它让编译器知道该符号（变量或函数）将在链接阶段找到。

- **不会分配内存**：extern 不会为变量分配内存，它只是提供了一个对符号的声明。实际的内存分配和初始化将在定义中进行。







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





## 友元类/友元函数

访问类中的私有成员/私有函数

⚠️注意点：

- 友元关系不被继承
- 不具有**交换性**与**传递性**



## C++与其他语言的区别

### 与C的区别

1️⃣new/delete 2️⃣引用 3️⃣面向对象 4️⃣异常处理 5️⃣STL 6️⃣命名空间

### 🆕🌟与Go的区别

- 标准库容器

- 第三方库，包管理

  - C++：❌ 无官方工具（需用 vcpkg / Conan）依赖 CMake 和第三方工具
  - Golang：✅ Go Modules 官方支持

- 并发模型

> **C++线程模型1:1， 而Golang线程模型是M：N，他们两种各有什么优缺点**
>
> - 调度延迟
> - 切换开销
> - 资源开销
>
> **适用场景**
>
> - **C++ 1:1模型**
>   ✅ 实时系统（自动驾驶控制）
>   ✅ 高性能计算（OpenMP并行计算）
>   ✅ 低延迟网络（DPDK高速报文处理）
> - **Golang M:N 模型**
>   ✅ 微服务架构（Envoy代理）
>   ✅ Web服务器（处理10k+并发连接）
>   ✅ 流处理系统（Kafka消费者组）

- 类型系统

  - C++支持隐式类型转换
  - Golang禁止隐式类型转换

- 实现多态的方式

  - C++ 的多态主要通过 **虚函数（virtual function）** 和 **运行时多态（Runtime Polymorphism）** 实现。此外，还可以使用 **模板（Templates）** 实现 **编译时多态（Compile-time Polymorphism）**。

  - Go **不支持类继承**，但支持**接口（interface）（鸭子类型）**来实现多态。

    ```go
    package main
    
    import "fmt"
    
    // 定义接口
    type Animal interface {
        MakeSound()
    }
    
    // Dog 结构体实现 Animal 接口
    type Dog struct{}
    func (d Dog) MakeSound() {
        fmt.Println("Dog barks")
    }
    
    // Cat 结构体实现 Animal 接口
    type Cat struct{}
    func (c Cat) MakeSound() {
        fmt.Println("Cat meows")
    }
    
    func main() {
        var animal Animal
    
        animal = Dog{}
        animal.MakeSound() // ✅ Dog barks
    
        animal = Cat{}
        animal.MakeSound() // ✅ Cat meows
    }
    ```

    

- 应用场景：

  - C++：交易系统、游戏引擎、嵌入式
  - Golang：云原生、微服务、分布式





## 单例模式

### 懒汉模式

- 使用call_once

```cpp
#include <iostream>
#include <mutex>

class Singleton {
public:
    static Singleton& getInstance() {
        std::call_once(initFlag, []() {
            instance = new Singleton();
        });
        return *instance;
    }

    // 禁止拷贝和赋值
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    void show() { std::cout << "Singleton Instance" << std::endl; }

private:
    Singleton() { std::cout << "Singleton Constructor" << std::endl; }
    ~Singleton() { std::cout << "Singleton Destructor" << std::endl; }

    static Singleton* instance;
    static std::once_flag initFlag;
};

// 初始化静态成员
Singleton* Singleton::instance = nullptr;
std::once_flag Singleton::initFlag;

int main() {
    Singleton& s1 = Singleton::getInstance();
    s1.show();

    Singleton& s2 = Singleton::getInstance();
    s2.show();

    std::cout << "s1 and s2 are the same instance: " << (&s1 == &s2) << std::endl;
    return 0;
}
```

- 使用静态局部变量实现线程安全

```cpp
#include <iostream>

class Singleton {
public:
    static Singleton& getInstance() {
        static Singleton instance; // C++11 线程安全
        return instance;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    void show() { std::cout << "Singleton Instance" << std::endl; }

private:
    Singleton() { std::cout << "Singleton Constructor" << std::endl; }
    ~Singleton() { std::cout << "Singleton Destructor" << std::endl; }
};

int main() {
    Singleton& s1 = Singleton::getInstance();
    s1.show();

    Singleton& s2 = Singleton::getInstance();
    s2.show();

    std::cout << "s1 and s2 are the same instance: " << (&s1 == &s2) << std::endl;
    return 0;
}
```





### 饿汉模式

```cpp
#include <iostream>

class Singleton {
public:
    static Singleton& getInstance() {
        return instance;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    void show() { std::cout << "Singleton Instance" << std::endl; }

private:
    Singleton() { std::cout << "Singleton Constructor" << std::endl; }
    ~Singleton() { std::cout << "Singleton Destructor" << std::endl; }

    static Singleton instance;
};

// 静态实例提前创建（程序启动时）
Singleton Singleton::instance;

int main() {
    Singleton& s1 = Singleton::getInstance();
    s1.show();

    Singleton& s2 = Singleton::getInstance();
    s2.show();

    std::cout << "s1 and s2 are the same instance: " << (&s1 == &s2) << std::endl;
    return 0;
}
```





#### Double check

```cpp
#include <iostream>
#include <mutex>

class Singleton {
public:
    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mutex_);
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }

    void doSomething() {
        std::cout << "Singleton doing something" << std::endl;
    }

private:
    Singleton() {} // Private constructor to prevent instantiation
    static Singleton* instance;
    static std::mutex mutex_;
};

Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mutex_;
```







## 解释型语言和编译型语言的区别

> **1️⃣ 解释型语言（Interpreted Language）**
>
> ​	•	**执行方式：** 解释型语言通过解释器逐行解释源代码，并在运行时即时执行。也就是说，程序的每一行代码在执行时都会被解释器逐行读取并转换为机器码。
>
> ​	•	**执行过程：** 在运行时，解释器会逐行分析源代码，并执行相应的操作。每次执行时，解释器都需要重新解析源代码。
>
> ​	•	**常见语言：** Python、JavaScript、Ruby、PHP、Perl等。
>
> ​	•	**优缺点：**
>
> ​	•	**优点：**
>
> ​	•	**跨平台性强：** 只要目标系统上安装了解释器，就可以执行程序，不需要重新编译。
>
> ​	•	**调试方便：** 由于逐行执行，调试和错误定位相对容易。
>
> ​	•	**缺点：**
>
> ​	•	**执行效率较低：** 由于需要逐行解释并执行，每次执行都需要进行解释，导致执行速度较慢。
>
> ​	•	**需要解释器：** 需要一个适当的解释器来运行程序，依赖环境。
>
> 
>
> **2️⃣ 编译型语言（Compiled Language）**
>
> ​	•	**执行方式：** 编译型语言在程序运行之前通过编译器将源代码转换为目标机器码或中间代码，生成可执行文件。然后，操作系统直接执行这些可执行文件。
>
> ​	•	**执行过程：** 编译的过程是在程序运行前进行的，编译器将整个程序编译成目标代码（机器代码或中间代码），执行时不再需要编译器，只直接运行编译后的可执行文件。
>
> ​	•	**常见语言：** C、C++、Rust、Go、Swift、Fortran等。
>
> ​	•	**优缺点：**
>
> ​	•	**优点：**
>
> ​	•	**执行效率高：** 由于已经被编译成机器码，程序运行时不需要额外的翻译过程，执行速度快。
>
> ​	•	**无需依赖编译器：** 编译后的可执行文件不再需要源代码或编译器，直接运行即可。
>
> ​	•	**缺点：**
>
> ​	•	**跨平台性差：** 生成的可执行文件依赖于平台（如操作系统和硬件架构），需要针对不同平台重新编译。
>
> ​	•	**调试较困难：** 由于没有逐行执行源代码，调试时通常需要额外的调试工具，错误定位较为复杂。
>



## RAII

RAII（Resource Acquisition Is Initialization，资源获取即初始化），RAII机制的核心思想是通过构造函数获取资源，并通过析构函数释放资源，从而确保资源的正确管理和自动释放，避免资源泄漏或忘记释放资源。

- **内存管理**：使用std::vector、std::unique_ptr等智能指针来管理动态分配的内存。

- **文件操作**：通过std::ifstream、std::ofstream等文件流对象，文件会在对象销毁时自动关闭。

```cpp
lass FileManager {
private:
    std::fstream file;

public:
    // 构造函数：打开文件
    FileManager(const std::string& filename) {
        file.open(filename, std::ios::in | std::ios::out);
        if (!file) {
            throw std::ios_base::failure("Failed to open file");
        }
        std::cout << "File opened\n";
    }

    // 析构函数：关闭文件
    ~FileManager() {
        if (file.is_open()) {
            file.close();
            std::cout << "File closed\n";
        }
    }

    void writeData(const std::string& data) {
        file << data;
    }

    std::string readData() {
        std::string data;
        file >> data;
        return data;
    }
};
```

- **互斥锁管理**：通过std::lock_guard、std::unique_lock等对象来管理锁，当对象超出作用域时自动释放锁，避免死锁。
