# Golang

# 数据结构

## chan原理



# 并发

## GMP模型

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250203%E4%B8%8B%E5%8D%8814847493.png" alt="image-20250203下午14847493" style="zoom:50%;" />

- 新建 `G` 时，新`G`会优先加入到 `P` 的本地队列；如果本地队列满了，则会把本地队列中一半的 `G` 移动到全局队列。
- `P` 的本地队列为空时，就从全局队列里去取。



### 1. Goroutine（G）

- Goroutine 是 Go 中的并发执行单元，类似于线程，但比线程更加轻量级。每个 Goroutine 有独立的栈空间，栈的大小会动态扩展，初始栈大小很小（约 2KB）。
- Goroutine 是由 Go 运行时调度的，并不直接对应操作系统线程，而是通过 GMP 模型在多个 OS 线程上高效调度。

### 2. Machine（M）

- M 代表操作系统线程，是 Go 运行时用来实际执行 Goroutine 的实体。
- 每个 M 运行时负责执行 Goroutine 的代码，M 可以是操作系统的内核线程，Go 运行时会根据需要动态地创建或销毁 M。
- M 负责执行具体的计算任务，它与 Goroutine 是一对多的关系，一个 M 可以执行多个 Goroutine，但一个 Goroutine 在某一时刻只能在一个 M 上执行。

### 3. Processor（P）

- P 代表逻辑处理器，用来控制 Goroutine 的调度。P 的数量通常由用户通过 `GOMAXPROCS` 设置，表示可以同时运行 Goroutine 的最大并发数量（即逻辑 CPU 核心数）。
- P 管理一个队列，队列中保存待执行的 Goroutine。当 P 和 M 关联时，P 会将自己队列中的 Goroutine 分配给 M 执行。
- 每个 P 只会绑定一个 M，M 只能在与其关联的 P 上调度 Goroutine。
- P 的数量限制了系统的并发度，即即使有很多 M 和 Goroutine，也只有 P 允许的数量并发执行。

### GMP 模型的工作原理

1. 当程序启动时，Go 运行时会初始化一组 P（逻辑处理器），P 的数量可以通过 `runtime.GOMAXPROCS` 设置，默认值是系统 CPU 核心数。
2. Goroutine 被创建时，运行时会将它放入某个 P 的本地队列中。
3. M 是操作系统的线程，当 M 启动时，它需要与一个 P 关联才能开始执行 Goroutine。P 决定将哪些 Goroutine 分配给 M 运行。
4. 如果 M 发现 P 的队列中没有可运行的 Goroutine，它会尝试从其他 P 的队列中窃取 Goroutine 来执行，这称为工作窃取（**work stealing**）。
5. 当一个 M 处于 I/O 阻塞状态或发生系统调用时，M 会释放 P，让其他 M 可以利用这个 P 继续执行 Goroutine。
6. 如果没有足够的 M 来运行 P 队列中的任务，Go 运行时会动态创建新的 M，反之如果有多余的 M，M 也会被销毁。



## channel



## sync.Pool

- **基本用法**

1. **定义 Pool**：

   ```go
   var bufferPool = &sync.Pool{
       New: func() interface{} {
           return new(bytes.Buffer) // 定义创建新对象的函数
       },
   }
   ```

2. **从 Pool 获取对象**：

   ```go
   buf := bufferPool.Get().(*bytes.Buffer) // 类型断言
   defer bufferPool.Put(buf)               // 用完后放回池中
   buf.Reset()                             // 重置对象状态（避免脏数据）
   ```

3. **放回对象**：

   ```go
   bufferPool.Put(buf) // 对象放回池中，供后续复用
   ```

- **适用场景**

  > - 需要减少内存分配和 GC 压力的场景。
  > - 高并发下频繁创建/销毁对象的场景。

- **功能**

> **对象复用**：
>
> - 当程序需要频繁创建和销毁某些对象时（例如缓冲区、临时结构体等），`sync.Pool` 会将这些对象缓存起来，后续需要时直接从池中获取，而不是重新分配内存。
> - 复用对象可显著减少内存分配次数，降低 GC 的工作量。
>
> **自动清理**：
>
> - 池中的对象可能被垃圾回收器（GC）自动清理（Go 1.13 之前是两轮 GC 后清理，之后优化为每次 GC 都可能清理）。
> - 因此，`sync.Pool` 适合存储**临时对象**，不能依赖它保存长期使用的数据。





# 内存

## GC

gc的过程一共分为四个阶段：

1. 栈扫描（开始时STW）
2. 第一次标记（并发）
3. 第二次标记（STW）
4. 清除（并发）

整个进程空间里申请每个对象占据的内存可以视为一个图，初始状态下每个内存对象都是白色标记。

1. 先STW，做一些准备工作，比如 enable write barrier。然后取消STW，将扫描任务作为多个并发的goroutine立即入队给调度器，进而被CPU处理
2. 第一轮先扫描root对象，包括全局指针和 goroutine 栈上的指针，标记为灰色放入队列
3. 第二轮将第一步队列中的对象引用的对象置为灰色加入队列，一个对象引用的所有对象都置灰并加入队列后，这个对象才能置为黑色并从队列之中取出。循环往复，最后队列为空时，整个图剩下的白色内存空间即不可到达的对象，即没有被引用的对象；
4. 第三轮再次STW，将第二轮过程中新增对象申请的内存进行标记（灰色），这里使用了write barrier（写屏障）去记录



## 内存逃逸

在 Go 语言中，**内存逃逸（Escape Analysis）** 是编译器优化的一部分，用于决定变量是在栈（Stack）上分配还是在堆（Heap）上分配。Go 的编译器会在编译阶段进行逃逸分析（Escape Analysis）

**1️⃣ 为什么会发生内存逃逸？**



在 Go 语言中，栈的内存管理效率高，但大小有限。因此，Go 通过逃逸分析决定某些变量是否需要分配到堆上。主要导致变量逃逸的情况有：



1️⃣ **变量的生命周期超出了函数作用域**

​	•	变量如果在函数返回后仍然被使用，就无法存储在栈上，而必须存储在堆上。

```
func escapeExample() *int {
    x := 10 // x 是一个局部变量
    return &x // x 的地址被返回，x 逃逸到堆
}
```

2️⃣ **变量被赋值给接口类型**

​	•	由于接口是一个包含**动态类型信息**的结构，Go 需要在堆上分配实际数据。

```
func escapeToInterface() interface{} {
    s := "hello"
    return s // s 逃逸到堆，因为它被存入了 interface{}
}
```

3️⃣ **变量被闭包捕获**

​	•	Go 的闭包可能会在函数作用域外使用捕获的变量，导致变量逃逸。

```
func closureEscape() func() int {
    x := 42
    return func() int { // x 被闭包引用，因此逃逸到堆
        return x
    }
}
```

4️⃣ **大对象的分配**

​	•	Go 编译器可能会为了优化栈的使用，将较大的变量分配到堆上，即使它们没有逃逸。

```
func largeObject() {
    bigArray := make([]int, 1000000) // 逃逸到堆
    fmt.Println(bigArray[0])
}
```

5️⃣ **变量的地址被传递到堆上的数据结构**

​	•	如果变量的地址被存储在堆中的数据结构中，它也会逃逸。

```
type Data struct {
    ptr *int
}

func escapeStruct() {
    x := 10
    d := &Data{ptr: &x} // x 逃逸到堆，因为 &x 被存入了堆上的 Data 结构
    fmt.Println(d)
}
```

**2️⃣ 如何检测内存逃逸？**



Go 提供了 -gcflags="-m" 选项来查看编译器的逃逸分析：

```
go run -gcflags="-m" main.go
```

示例：

```
func main() {
    x := 10
    fmt.Println(&x) // 变量 x 逃逸
}
```

运行：

```
# 输出（编译器会提示 x 逃逸）
main.go:6:13: &x escapes to heap
```

**3️⃣ 如何减少内存逃逸？**



为了优化性能，尽量减少不必要的逃逸，可以使用以下方法：



🔹 **尽量使用值类型而不是指针类型**

```
func noEscape() int {
    x := 10
    return x // x 没有逃逸
}
```

🔹 **避免不必要的接口转换**

```
func avoidInterface() {
    s := "hello"
    fmt.Println(s) // s 没有逃逸
}
```

🔹 **避免闭包持有变量的引用**

```
func noClosureEscape() {
    x := 10
    func(y int) { fmt.Println(y) }(x) // 传递值，而不是捕获变量
}
```

🔹 **使用 sync.Pool 进行对象复用**

```
import "sync"

var pool = sync.Pool{
    New: func() interface{} { return new(int) },
}

func usePool() {
    obj := pool.Get().(*int) // 从 pool 获取对象，减少逃逸
    *obj = 42
    pool.Put(obj) // 归还到 pool
}
```

**4️⃣ 逃逸的影响**



✅ **优点**

​	•	逃逸到堆上可以避免栈溢出（栈的大小是有限的）。



❌ **缺点**

​	•	堆分配的内存需要垃圾回收（GC），比栈上的内存管理成本更高。

​	•	堆上的数据访问速度比栈慢。

**5️⃣ 总结**



📌 **内存逃逸**是 Go 语言编译器在决定变量存储位置时的重要优化手段。当变量超出作用域、被闭包捕获、存入接口、存入堆上的数据结构或太大时，它会被分配到堆上。

📌 使用 -gcflags="-m" 进行逃逸分析可以帮助优化代码，减少 GC 开销，提高程序性能。



## 指针

### uintptr

uintptr 是 Go 语言中的一种**无符号整数类型**，它的大小与指针的大小相同（通常为 32 位或 64 位），用于存储指针的数值表示。

```go
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	var x int = 100
	ptr := unsafe.Pointer(&x)     // *int → unsafe.Pointer
	addr := uintptr(ptr)          // unsafe.Pointer → uintptr
	newPtr := unsafe.Pointer(addr) // uintptr → unsafe.Pointer
	fmt.Println(*(*int)(newPtr))  // 100
}
```

**⚠️ 注意：** uintptr 只是一个整数，GC **不会** 认为它是指针，因此在 uintptr 存在期间，GC 可能回收 ptr 指向的对象，导致悬空指针。



### unsafe.Pointer

- **unsafe.Pointer 不能参与指针运算**

```go
p := unsafe.Pointer(uintptr(unsafe.Pointer(&x)) + 1) // ✅ 正确
p = unsafe.Pointer(&x + 1) // ❌ 错误，Go 不支持指针运算
```

- **unsafe.Pointer 的作用**

unsafe.Pointer 主要用于 **绕过 Go 语言的类型安全限制**，用于**低级内存操作**，如：

​	•	**任意类型的指针转换**（*T1 ↔ *T2）

​	•	**指针与 uintptr 相互转换**，实现**指针运算**

​	•	**访问非导出字段**

​	•	**与 C 代码进行交互**



## 滥用导致的GC破坏

**1️⃣ Go GC（垃圾回收）如何处理指针**

Go 的垃圾回收（GC）是**基于可达性分析**的：

​	•	GC 遍历所有 **根对象**（如全局变量、栈上的指针等）。

​	•	通过指针引用找到**仍然可达的对象**，将其标记为存活。

​	•	不可达的对象会被回收。

如果 GC 发现某个内存地址**没有任何指针引用**，它可能会认为该对象已经不需要了，并进行回收。

**2️⃣ unsafe.Pointer 的特殊性**

unsafe.Pointer 是 Go 语言中的**通用指针类型**，可以与任何指针类型（*T）互相转换：

```
var x int
ptr := unsafe.Pointer(&x) // *int → unsafe.Pointer
```

它允许绕过 Go 的类型系统进行指针操作，因此在不小心使用时，可能导致 GC 误判对象是否仍然存活。

 unsafe.Pointer 破坏 GC 的两种情况**



**🔹 1. 通过 uintptr 进行指针运算，导致对象悬空**



Go 的 GC 只追踪**指针类型**（如 *int），但 uintptr 是一个普通整数，不会被 GC 视为指针。因此，如果把 unsafe.Pointer 转换为 uintptr，GC 可能错误地认为原对象已经不可达，并回收它。



**❌ 错误示例**

```
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	data := []int{1, 2, 3}
	ptr := uintptr(unsafe.Pointer(&data[0])) // unsafe.Pointer → uintptr
	// 此时，Go GC 可能回收 `data`，因为 `ptr` 只是个整数，GC 不知道它是指针
	fmt.Println(*(*int)(unsafe.Pointer(ptr))) // 悬空指针，可能导致崩溃！
}
```

**💡 解决方案**

不要存储 uintptr，而是保持 unsafe.Pointer，让 GC 能追踪：

```
ptr := unsafe.Pointer(&data[0]) // 这样 GC 仍然知道 data 的存在
```

**🔹 2. 通过 unsafe.Pointer 修改 Go 对象，使 GC 失效**



GC 依赖类型信息来正确回收内存。如果使用 unsafe.Pointer 误修改了 Go 运行时的内部数据结构，可能会导致 GC 失效。



**❌ 错误示例**

```
package main

import (
	"fmt"
	"unsafe"
)

func main() {
	str := "hello"
	strHeader := (*[2]uintptr)(unsafe.Pointer(&str)) // 修改字符串结构体
	strHeader[1] = 1000                              // 非法修改长度字段
	fmt.Println(str)                                 // 崩溃或未定义行为
}
```

​	•	Go **字符串是不可变的**，但这里我们直接修改了底层结构，破坏了 GC 规则，可能导致崩溃。



**💡 解决方案**

不要用 unsafe.Pointer 直接修改 Go 运行时管理的对象，尤其是字符串、切片、映射等复杂结构。

- **unsafe.Pointer 安全使用原则**

| **安全性** | **原则**                                         |
| ---------- | ------------------------------------------------ |
| ✅ **允许** | 用于转换不同指针类型，例如 *T ↔ unsafe.Pointer   |
| ✅ **允许** | 计算结构体字段偏移，如 unsafe.Offsetof           |
| ❌ **避免** | 使用 uintptr 进行指针运算并存储（GC 无法追踪）   |
| ❌ **避免** | 修改 Go 运行时的内部数据结构（如 string、slice） |
| ❌ **避免** | 直接访问和操作 Go 的内存管理区域                 |



* **总结**



1️⃣ Go 的 GC 通过**指针可达性**管理内存，unsafe.Pointer 可能绕过类型系统，影响 GC 判断。

2️⃣ **将 unsafe.Pointer 转换为 uintptr** 可能导致 GC 误回收对象，造成悬空指针。

3️⃣ 使用 unsafe.Pointer **修改 Go 运行时的内部数据结构**（如 string、slice）可能破坏 GC 和运行时，导致未定义行为。

4️⃣ **安全使用 unsafe.Pointer：** 只用于类型转换，避免 uintptr 存储和指针运算。





## new 与make的区别

- make 仅用来分配及**初始化**类型为 **slice、map、chan** 的数据

- new 可分配任意类型的数据，根据传入的类型申请一块内存，返回指向这块内存的指针，即类型 *Type。**不初始化**





## 内存泄漏





## C++空对象/类大小，为什么？ —— 对比go的`struct{}`

**空类占1个字节**（类和对象大小通常相等）

原因：

- 保证每个对象有唯一的地址，
- 保证内存布局合理，比如对象数组

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

设计理念不同：Go 设计 struct{} 的初衷是**高效、简洁**，如果结构体不包含任何字段，它**不应该浪费任何内存**。这与 C++ 的设计不同，C++ 强制**对象必须有唯一的地址**，而 Go 不要求这一点。

- 空结构体用途：（节省空间）
  - map-》set
  - 在channel中传递信号，不传输值





# 其他

## Go/C++区别


