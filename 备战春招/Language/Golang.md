# Golang

# 数据结构

## channel

Go 语言中的 **Channel（通道）** 是用于 Goroutine 之间通信和同步的核心数据结构，其底层实现结合了高效的并发控制和内存管理。以下是 Channel 的底层结构和工作原理的详细分析：

---

### **1. Channel 的底层结构**
Channel 的底层由 `runtime.hchan` 结构体表示（定义在 `runtime/chan.go` 中），核心字段如下：

```go
type hchan struct {
    qcount   uint           // 当前队列中的元素数量（缓冲区已存数据量）
    dataqsiz uint           // 缓冲区的大小（容量）
    buf      unsafe.Pointer // 指向环形缓冲区的指针
    elemsize uint16         // 元素类型的大小（字节）
    closed   uint32         // 是否已关闭（0-未关闭，1-已关闭）
    elemtype *_type         // 元素类型的元信息（用于类型检查）
    sendx    uint           // 发送索引（指向缓冲区下一个写入位置）
    recvx    uint           // 接收索引（指向缓冲区下一个读取位置）
    recvq    waitq          // 等待接收的 Goroutine 队列（双向链表）
    sendq    waitq          // 等待发送的 Goroutine 队列（双向链表）
    lock     mutex          // 互斥锁（保护 Channel 的并发操作）
}
```

#### **关键成员说明**
- **`buf`**：指向一个环形缓冲区（只有当 Channel 是带缓冲的时才会分配）。
- **`sendq` 和 `recvq`**：等待队列，存储因 Channel 满/空而阻塞的 Goroutine（通过 `sudog` 结构体表示）。
- **`lock`**：互斥锁，保证对 Channel 操作的原子性（例如并发发送和接收）。

---

### **2. Channel 的工作原理**
Channel 的行为由其类型（无缓冲或有缓冲）决定，但底层机制是统一的。

#### **2.1 发送数据（`ch <- val`）**
1. **加锁**：通过 `lock` 互斥锁保护 Channel 的并发操作。
2. **直接写入缓冲区（如果可能）**：
   - 如果缓冲区未满，将数据写入 `buf` 的 `sendx` 位置，更新 `sendx` 和 `qcount`。
3. **阻塞等待（如果缓冲区已满）**：
   - 如果 Channel 是无缓冲的，或者缓冲区已满，当前 Goroutine 会被封装为 `sudog`，加入 `sendq` 队列。
   - 调用 `gopark` 挂起当前 Goroutine，等待被唤醒。
4. **唤醒接收者（如果有等待的接收者）**：
   - 如果 `recvq` 队列不为空，直接将数据传递给第一个等待的接收者，并唤醒其 Goroutine。

#### **2.2 接收数据（`val := <-ch`）**
1. **加锁**：同样通过 `lock` 保护操作。
2. **直接从缓冲区读取（如果可能）**：
   - 如果缓冲区非空，从 `recvx` 位置读取数据，更新 `recvx` 和 `qcount`。
3. **阻塞等待（如果缓冲区为空）**：
   - 如果 Channel 是无缓冲的，或者缓冲区为空，当前 Goroutine 被加入 `recvq` 队列。
   - 调用 `gopark` 挂起，等待被唤醒。
4. **唤醒发送者（如果有等待的发送者）**：
   - 如果 `sendq` 队列不为空，从第一个等待的发送者获取数据（或直接写入缓冲区），并唤醒其 Goroutine。

#### **2.3 关闭 Channel（`close(ch)`）**
1. **加锁**：确保原子性。
2. **标记 Channel 为关闭状态**：设置 `closed = 1`。
3. **唤醒所有等待的 Goroutine**：
   - 将 `sendq` 和 `recvq` 队列中的所有 Goroutine 唤醒，接收者会收到零值，发送者会触发 panic。

---

### **3. 无缓冲 Channel vs 带缓冲 Channel**
- **无缓冲 Channel（`make(chan T)`）**：
  - 发送和接收必须同步完成（直接传递数据，不经过缓冲区）。
  - 常用于 Goroutine 间的精确同步。
- **带缓冲 Channel（`make(chan T, size)`）**：
  - 允许在缓冲区未满时异步发送，或在缓冲区非空时异步接收。
  - 适用于解耦生产者和消费者，提高吞吐量。

---

### **4. 底层实现的优化**
- **环形缓冲区**：通过 `sendx` 和 `recvx` 索引实现循环利用内存，避免频繁内存分配。
- **等待队列（`sendq` 和 `recvq`）**：通过双向链表管理阻塞的 Goroutine，确保公平唤醒（FIFO）。
- **零拷贝优化**：当发送者和接收者直接传递数据时，避免通过缓冲区复制数据。



## slice

#### 1️⃣ 底层结构

```go
type SliceHeader struct {
 Data uintptr
 Len  int
 Cap  int
}
```

#### 2️⃣ 切片的深浅拷贝

- 使用=操作符拷贝切片，这种就是浅拷贝
- 使用[:]下标的方式复制切片，这种也是浅拷贝
- 使用Go语言的内置函数copy()进行切片拷贝，这种就是深拷贝，

#### 3️⃣ 空切片、零切片、nil切片

- 切片

我们把切片内部数组的元素都是零值或者底层数组的内容就全是 nil的切片叫做零切片，使用make创建的、长度、容量都不为0的切片就是零值切片：

```go
slice := make([]int,5) // 0 0 0 0 0
slice := make([]*int,5) // nil nil nil nil nil
```

- nil切片

nil切片的长度和容量都为0，并且和nil比较的结果为true，采用直接创建切片的方式、new创建切片的方式都可以创建nil切片：

```go
var slice []int
var slice = *new([]int)
```

- 空切片

空切片的长度和容量也都为0，但是和nil的比较结果为false，因为所有的空切片的数据指针都指向同一个地址 0xc42003bda0；使用字面量、make可以创建空切片：

```go
var slice = []int{}
var slice = make([]int, 0)
```

空切片指向的 zerobase 内存地址是一个神奇的地址，从 Go 语言的源代码中可以看到它的定义：

```go
// base address for all 0-byte allocations
var zerobase uintptr

// 分配对象内存
func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
 ...
 if size == 0 {
  return unsafe.Pointer(&zerobase)
 }
  ...
}
```

#### 4️⃣ 扩容机制

切片在扩容时会进行内存对齐，这个和内存分配策略相关。进行内存对齐之后，新 slice 的容量是要 大于等于老 slice 容量的 2倍或者1.25倍，当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；原 slice 容量超过 1024，新 slice 容量变成原来的1.25倍。

- 为什么？以我的理解，在1024一下通过两倍迅速的扩大空间，防止频繁的扩容耗时，而在**大容量时改为 1.25 倍增长** 避免浪费大量内存。



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
- 初始化数量：Go程序启动时，**初始M的数量通常等于`GOMAXPROCS`的值**（即逻辑处理器的数量，默认等于CPU核心数）。

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



###  work stealing 

当本线程⽆可运行的G时，尝试从其他线程绑定的P偷取G，⽽不是销毁线程。先全局队列，后本地队列

### hand off

当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执⾏。



### go func()在GMP上的流程

- 创建G ——》绑定到执行当前操作的M上的本地队列中，如果满了则放入全局队列 ——〉M调度并执行G ——》时间片消耗完之后加入本地队列

- 当G发生阻塞，此时唤醒一个休眠的M或者新建来接管当前的P，之后阻塞的G与之前的M捆绑，当G唤醒之后被放入全局队列，旧的M加入休眠的M队列或销毁

![image-20250204下午20930874](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250204%E4%B8%8B%E5%8D%8820930874.png)

#### 本地队列没有G了，那么会先去全局队列里拿还是先去其他P的队列里拿

1️⃣ **先尝试从全局运行队列（Global Queue）获取 G**

​	•	P **首先** 会去 **全局队列** 取 G，因为全局队列存放的是新建的或因系统调用被放回的 G。

​	•	取的数量通常是 **最多 32 个** 或全局队列里的一半（取较小值）。



2️⃣ **如果全局队列为空，则尝试从其他 P 偷取（Work Stealing）**

​	•	P **随机** 选择 **其他 P 的本地队列**，尝试**偷取**（Work Stealing）一半的 G。

​	•	这种机制可以防止某些 P 任务过多，而另一些 P 处于空闲状态，提高调度效率。



3️⃣ **如果仍然没有可运行的 G，那么 P 会进入休眠状态**

​	•	P 会通过 park 进入**休眠**状态，等待新的 G 可执行。

​	•	如果所有 P 都没有 G，那么 M（操作系统线程） 也可能进入休眠，最终 runtime 可能会进入 idle 状态。







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

- **💡目标**：**在保证对象不丢失的情况下提高GC效率，减少STW时间**



### 三色标记法无STW可能产生的问题

cond1:**黑色对象指向白色对象** ——》强三色不变式解决 ——〉插入写屏障

cond2:**同时原本的指向白色对象的灰色对象的引用丢失** ——》弱三色不变式解决 ——〉删除写屏障

在没有STW的情况下，如果同时满足上述条件，则会出现对象丢失的情况

💡**解决办法**：

- 强三色不变式

不允许黑色指向白色

- 弱三色不变式

黑色可以指向白色，仅当白色上游有灰色引用

### 屏障机制

1. **插入屏障（对象被引用时触发）**

在A对象引用B时，B被标记为灰色 —— 满足强三色不变式

- 不在栈上使用（保证性能）——在走完一趟三色标记之后，将栈上的对象置白，通过STW重写走三色标记

2. **删除屏障（对象被删除时触发）**

被删除的对象如果自身为 灰或白 ，则被标记为灰

- 回收精度低，被删除的对象可能没有被其他的对象引用，但是当前这一轮最终会被标记为黑色

3. 混合写屏障

![image-20250204下午61843476](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250204%E4%B8%8B%E5%8D%8861843476.png)

### 还有哪些地方需要STW

**初始阶段（Mark Termination）**：

- GC 开始时需要短暂 STW，用于扫描根对象（如全局变量、当前 Goroutine 的栈等），确保一致性。

**终止阶段（Mark Termination）**：

- 在标记完成后，仍需极短时间的 STW（通常小于 100μs）确认所有标记工作完成，并切换 GC 阶段（如开启清扫阶段）。

## 内存逃逸

在 Go 语言中，**内存逃逸（Escape Analysis）** 是编译器优化的一部分。Go 的编译器会在编译阶段进行逃逸分析（Escape Analysis）决定变量是在栈（Stack）上分配还是在堆（Heap）上分配

**1️⃣ 为什么会发生内存逃逸？**

在 Go 语言中，栈的**内存管理效率高**，但大小有限，并且作用域也有限。因此，Go 通过逃逸分析决定某些变量是否需要分配到堆上。

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

​	•Go 编译器可能会为了优化栈的使用，将较大的变量分配到堆上，即使它们没有逃逸。

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

​	•管理开销：堆分配的内存需要垃圾回收（GC），比栈上的内存管理成本更高。

​	•访问开销：堆上的数据访问速度比栈慢。



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

- 标准库容器
- 开发效率
- 第三方库，包管理
- 并发

C++：灵活但复杂，适合底层和性能关键场景。

Go：简洁高效，适合现代网络服务和快速迭代。
