---
layout: post
title: C++ 20 CPO 和 Ranges 新算法
date: "2023-04-18 19:45:00"
tags: [C++,Windows]
categories: [blog]
---
在之前的 Ranges 文章中我称 Ranges 为重要改进，不仅仅是因为 Ranges 本身提供了容器无关的抽象，还因为 Ranges 带来了“新的”算法，解决了一些函数查找的历史问题。

<!-- more -->

这篇文章鸽了好久了，因为理解不到位外加犯懒于是一种都只是半成品草稿，当然，实际上现在发表的这篇也是半成品整理出来的，因为这个话题太复杂了。

### 从函数查找说起

C++ 的函数查找机制主要分为两类：基于作用域从内到外的查找和实参依赖查找（ADL）。从内到外的查找很好理解，包括有限定的查找和简单的无限定查找（当然，也有注入类名这种“特殊”规则），而 ADL 作为无限定查找的一种情况，跳出了单纯的从内到外的查找方式，在发展的过程中变得非常复杂。

由于本文重点不在 ADL，本人也没能力解释清楚 ADL 的所有规则，因此仅介绍基本内容。

ADL 人如其名，是参考了函数的实参类型的查找规则，简单来讲，ADL 会根据实参的类型，查找从该类型的类内开始，到包含该类声明的第一个外部的命名空间为止的所有函数声明/定义，这代表会查找类内定义和声明的函数，基类的函数，以及存在嵌套类的情况，会查找到其外部类（因为此时还未查找到命名空间）里的函数。

无限定查找的一个重点是无限定查找在内部作用域中找到名字后便不再继续查找外部作用域的名字。

无限定查找有 3 种情况优于 ADL，因此会阻止 ADL：

1. 如果该调用在类内，则会查找到类内成员的声明
2. 调用的块作用域内的函数声明（注意，`using` 声明不是函数声明）
3. 通过向外层作用域查找找到非函数，例如对象（变量）和类型，和函数模板

第三条是重点，是接下来的内容的核心：如果能查找到变量名，则不使用 ADL。

### CPO

C++ 20 全新设计了一类被叫做 CPO（customization point object，定制点对象）的 “函数”。

所谓的定制点，是指 C++ 静态多态的接口，即定制点是 C++ 中可以由用户或者标准库本身改写默认实现的地方，这些定制点是标准规定的或者约定俗成的。

例如 C++ 11 中的 `std::begin<T>(T&)`，会自动选择成员函数 `begin()`，还有就是有些库按照约定俗成的 `swap` 函数，会优先使用用户定义的 `swap` 而不是 `std::swap`，虽然 `std::begin` 不存在默认实现，但是第三方库仍然可以选择使用成员函数，或者非成员函数。这种选择函数的地方就叫定制点。

这些函数包括在 ranges 和 iterator 头文件中的：

+ `ranges::begin`
+ `ranges::end`
+ `ranges::cbegin`
+ `ranges::cend`
+ `ranges::rbegin`
+ `ranges::rend`
+ `ranges::crbegin`
+ `ranges::crend`
+ `ranges::size`
+ `ranges::ssize`
+ `ranges::empty`
+ `ranges::data`
+ `ranges::cdata`

以及在 concept 头文件中的

+ `ranges::swap`

以及为 C++ 20 的三路比较服务的 compare 头文件里的

+ `strong_order`
+ `weak_order`
+ `partial_order`
+ `compare_strong_order_fallback`
+ `compare_weak_order_fallback`
+ `compare_partial_order_fallback`

使用这些新算法就可以自动完成选择操作，这实际上是为了简化定制点的实现。

传统上定制点大概有 2 种方式：

1. 手动添加特化（注意，不是重载，重载 STL 是未定义行为）
2. 使用 ADL 两步法

显然给 STL 加特化是繁琐而且不安全的，扩展性也不够好，因为用户很可能误操作删除了特化，并且有泛型的“特化”是重载，所以这种方法几乎不被使用

另一种方式是使用 ADL 两步法：

```cpp

class A {
    void swap(A& other);
}

A a, b;

using std::swap;
swap(a, b);
int i{}, j{};
swap(i, j);

```

由于 `swap(a, b)` 是一个无限定查找，同时 `A` 不是基础类型，并且 `using` 声明不是函数声明（不会阻止 ADL），因此 ADL 生效，找到了 `A::swap` 并使用，而后面的 `swap(i, j)` 中由于 `int` 是基础类型，因此不应用 ADL，并且当前作用域存在 `std::swap`，因此找到了 `std::swap` 并使用。

但问题是，开发者和用户必须要严格遵守这个模式，即先使用 `using` 声明载进行无限定的调用，否则该效果不会被应用：假设把无限定调用改成对 `std::swap` 的有限定调用，会导致没有使用用户的 `swap`；而将 `using` 指令去掉，会导致无法交换 `int` 类型的变量。

因此，ranges 新算法内部自己实现了 ADL 两步法，所以老的 `std::swap` 和 ADL 两步法的无限定调用都可以直接改成 `range::swap`，同时，`range` 新算法本身是函数对象而不是函数，使得用户可以使用 `using` 声明引入这个函数对象，同时使用无限定调用，最后一定找到的是这个函数对象。

### 隐藏友元

在类内的友元函数，如果不在类外部重写声明，是无法通过有限定查找找到的，但可以被无限定查找中的 ADL 找到。

根据以上知识，我们现在可以说，任何定制点优先使用成员函数，如果左操作数不是同类型的对象，例如一些二元运算符的重载，则应实现为隐藏友元。

### Niebloids

Niebloids 是 C++ 20 添加的 Ranges 新算法的一种叫法，Niebloids 采用了多返回值以及 Concepts 约束，更安全也更易用。

同时，Niebloids 保证在无限定调用能同时找到老 STL 算法和 Niebloids 时，一定优先考虑 Niebloids，而不是老的 STL 算法。

例如：

```cpp

using std::vector;
using namespace std::ranges;
// or using std::ranges::find;

vetcor<int> a;
find(a.begin(), a.end(), 2);
// should call std::ranges::find

```

但是现在存在两个问题：

1. `vector` 在 `std` 命名空间里，因此 ADL 可以找到 `std:find`
2. `std::find` 作为老 STL 算法，通常要求 `begin` 和 `end` 迭代器有相同类型，而新算法不要求，因此 `std::find` 比 `std::ranges::find` 更特化

    同时，`std::ranges::find` 存在 Concepts，更约束，但是特化优于约束

因此如果 `std::ranges::find` 是普通函数，则上述代码会使用 `std::find`。

因此解决方法很简单：将 `std::ranges::find` 实现为函数对象。

注意，由于 Ranges 新算法不需要定制，因此这些算法不叫作定制点对象。

### 如何实现 CPO 和 Niebloids

学习如何实现 CPO 和 Niebloids 还是有用的，因为作为库作者很可能需要我们自己提供定制点，但 CPO 和 Niebloids 实际上不能通过 Lambda 实现，原因在 [A note on namespace __cpo](https://quuxplusone.github.io/blog/2021/12/07/namespace-cpo/)。

当然，实现 CPO 和 Niebloids 并不需要多么复杂，如果你不想要知道原因（除了定制标准和发明 CPO 以外没人需要知道原因），照抄下面的模板即可：

```cpp

namespace std::ranges {
  namespace __iter_swap {
    struct __fn {
      auto operator()(~~~) const { ~~~ }
    };
  }
  inline namespace __cpo {
    inline constexpr auto iter_swap = __iter_swap::__fn{};
  }
}

```

