#  Golang

# 数据结构

## channel

> Go 语言中的 **Channel（通道）** 是用于 Goroutine 之间通信和同步的核心数据结构，其底层实现结合了高效的并发控制和内存管理。以下是 Channel 的底层结构和工作原理的详细分析：
>
> ---
>
> ### **1. Channel 的底层结构**
>
> Channel 的底层由 `runtime.hchan` 结构体表示（定义在 `runtime/chan.go` 中），核心字段如下：
>
> ```go
> type hchan struct {
>     qcount   uint           // 当前队列中的元素数量（缓冲区已存数据量）
>     dataqsiz uint           // 缓冲区的大小（容量）
>     buf      unsafe.Pointer // 指向环形缓冲区的指针
>     elemsize uint16         // 元素类型的大小（字节）
>     closed   uint32         // 是否已关闭（0-未关闭，1-已关闭）
>     elemtype *_type         // 元素类型的元信息（用于类型检查）
>     sendx    uint           // 发送索引（指向缓冲区下一个写入位置）
>     recvx    uint           // 接收索引（指向缓冲区下一个读取位置）
>     recvq    waitq          // 等待接收的 Goroutine 队列（双向链表）
>     sendq    waitq          // 等待发送的 Goroutine 队列（双向链表）
>     lock     mutex          // 互斥锁（保护 Channel 的并发操作）
> }
> ```
>
> #### **关键成员说明**
>
> - **`buf`**：指向一个环形缓冲区（只有当 Channel 是带缓冲的时才会分配）。
> - **`sendq` 和 `recvq`**：等待队列，存储因 Channel 满/空而阻塞的 Goroutine（通过 `sudog` 结构体表示）。
> - **`lock`**：互斥锁，保证对 Channel 操作的原子性（例如并发发送和接收）。
>
> ---
>
> ### **2. Channel 的工作原理**
>
> Channel 的行为由其类型（无缓冲或有缓冲）决定，但底层机制是统一的。
>
> #### **2.1 发送数据（`ch <- val`）**
>
> 1. **加锁**：通过 `lock` 互斥锁保护 Channel 的并发操作。
> 2. **直接写入缓冲区（如果可能）**：
>    - 如果缓冲区未满，将数据写入 `buf` 的 `sendx` 位置，更新 `sendx` 和 `qcount`。
> 3. **阻塞等待（如果缓冲区已满）**：
>    - 如果 Channel 是无缓冲的，或者缓冲区已满，当前 Goroutine 会被封装为 `sudog`，加入 `sendq` 队列。
>    - 调用 `gopark` 挂起当前 Goroutine，等待被唤醒。
> 4. **唤醒接收者（如果有等待的接收者）**：
>    - 如果 `recvq` 队列不为空，直接将数据传递给第一个等待的接收者，并唤醒其 Goroutine。
>
> #### **2.2 接收数据（`val := <-ch`）**
>
> 1. **加锁**：同样通过 `lock` 保护操作。
> 2. **直接从缓冲区读取（如果可能）**：
>    - 如果缓冲区非空，从 `recvx` 位置读取数据，更新 `recvx` 和 `qcount`。
> 3. **阻塞等待（如果缓冲区为空）**：
>    - 如果 Channel 是无缓冲的，或者缓冲区为空，当前 Goroutine 被加入 `recvq` 队列。
>    - 调用 `gopark` 挂起，等待被唤醒。
> 4. **唤醒发送者（如果有等待的发送者）**：
>    - 如果 `sendq` 队列不为空，从第一个等待的发送者获取数据（或直接写入缓冲区），并唤醒其 Goroutine。
>
> #### **2.3 关闭 Channel（`close(ch)`）**
>
> 1. **加锁**：确保原子性。
> 2. **标记 Channel 为关闭状态**：设置 `closed = 1`。
> 3. **唤醒所有等待的 Goroutine**：
>    - 将 `sendq` 和 `recvq` 队列中的所有 Goroutine 唤醒，接收者会收到零值，发送者会触发 panic。
>
> ---
>
> ### **3. 无缓冲 Channel vs 带缓冲 Channel**
>
> - **无缓冲 Channel（`make(chan T)`）**：
>   - 发送和接收必须同步完成（直接传递数据，不经过缓冲区）。
>   - 常用于 Goroutine 间的精确同步。
> - **带缓冲 Channel（`make(chan T, size)`）**：
>   - 允许在缓冲区未满时异步发送，或在缓冲区非空时异步接收。
>   - 适用于解耦生产者和消费者，提高吞吐量。
>
> ---
>
> ### **4. 底层实现的优化**
>
> - **环形缓冲区**：通过 `sendx` 和 `recvx` 索引实现循环利用内存，避免频繁内存分配。
> - **等待队列（`sendq` 和 `recvq`）**：通过双向链表管理阻塞的 Goroutine，确保公平唤醒（FIFO）。
> - **零拷贝优化**：当发送者和接收者直接传递数据时，避免通过缓冲区复制数据。
>

### channel发生死锁的情况

1. 无缓冲channel**发送或接收缺少配对操作**

```go
// 示例1：发送阻塞导致死锁
func main() {
    ch := make(chan int)
    ch <- 1  // 发送后无接收者，主 Goroutine 阻塞
}

// 示例2：接收阻塞导致死锁
func main() {
    ch := make(chan int)
    <-ch  // 接收时无发送者，主 Goroutine 阻塞
}
```

2. 有缓冲channel为空接受 / 为满发送

```go
// 缓冲已满导致发送阻塞
func main() {
    ch := make(chan int, 1)
    ch <- 1
    ch <- 2  // 缓冲已满，无接收者，主 Goroutine 阻塞
}
```

3. **多个 Goroutine 循环等待** ，互相依赖形成环路：多个 Goroutine 通过 Channel 形成环形依赖，彼此等待对方操作。

   ```go
   func main() {
       ch1, ch2 := make(chan int), make(chan int)
       go func() { <-ch1; ch2 <- 1 }()  // 等待 ch1 后发送到 ch2
       go func() { <-ch2; ch1 <- 1 }()  // 等待 ch2 后发送到 ch1
       select {}  // 主 Goroutine 阻塞，子 Goroutine 互相等待
   }
   ```

## slice

> #### 1️⃣ 底层结构
>
> ```go
> type SliceHeader struct {
>  Data uintptr
>  Len  int
>  Cap  int
> }
> ```
>
> #### 2️⃣ 切片的深浅拷贝
>
> - 使用=操作符拷贝切片，这种就是**浅拷贝**
> - 使用[:]下标的方式复制切片，这种也是**浅拷贝**
> - 使用Go语言的内置函数copy()进行切片拷贝，这种就是**深拷贝**，
>
> #### 3️⃣ 空切片、零切片、nil切片
>
> - 切片
>
> 我们把切片内部数组的元素都是零值或者底层数组的内容就全是 nil的切片叫做零切片，使用make创建的、长度、容量都不为0的切片就是零值切片：
>
> ```go
> slice := make([]int,5) // 0 0 0 0 0
> slice := make([]*int,5) // nil nil nil nil nil
> ```
>
> - nil切片
>
> nil切片的长度和容量都为0，并且和nil比较的结果为true，采用直接创建切片的方式、new创建切片的方式都可以创建nil切片：
>
> ```go
> var slice []int
> var slice = *new([]int)
> ```
>
> - 空切片
>
> 空切片的长度和容量也都为0，但是和nil的比较结果为false，因为所有的空切片的数据指针都指向同一个地址 0xc42003bda0；使用字面量、make可以创建空切片：
>
> ```go
> var slice = []int{}
> var slice = make([]int, 0)
> ```
>
> 空切片指向的 zerobase 内存地址是一个神奇的地址，从 Go 语言的源代码中可以看到它的定义：
>
> ```go
> // base address for all 0-byte allocations
> var zerobase uintptr
> 
> // 分配对象内存
> func mallocgc(size uintptr, typ *_type, needzero bool) unsafe.Pointer {
>  ...
>  if size == 0 {
>   return unsafe.Pointer(&zerobase)
>  }
>   ...
> }
> ```
>
> #### 4️⃣ 扩容机制
>
> 切片在扩容时会进行内存对齐，这个和内存分配策略相关。进行内存对齐之后，新 slice 的容量是要 大于等于老 slice 容量的 2倍或者1.25倍，当原 slice 容量小于 1024 的时候，新 slice 容量变成原来的 2 倍；原 slice 容量超过 1024，新 slice 容量变成原来的1.25倍。
>
> - 为什么？以我的理解，在1024一下通过两倍迅速的扩大空间，防止频繁的扩容耗时，而在**大容量时改为 1.25 倍增长** 避免浪费大量内存。
>
> 

# 并发

## GMP模型

> <img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250203%E4%B8%8B%E5%8D%8814847493.png" alt="image-20250203下午14847493" style="zoom:50%;" />
>
> - 新建 `G` 时，新`G`会优先加入到 `P` 的本地队列；如果本地队列满了，则会把本地队列中一半的 `G` 移动到全局队列。
> - `P` 的本地队列为空时，就从全局队列里去取。
>
> 
>
> ### 1. Goroutine（G）
>
> - Goroutine 是 Go 中的并发执行单元，类似于线程，但比线程更加轻量级。每个 Goroutine 有独立的栈空间，栈的大小会动态扩展，初始栈大小很小（约 2KB）。
> - Goroutine 是由 Go 运行时调度的，并不直接对应操作系统线程，而是通过 GMP 模型在多个 OS 线程上高效调度。
>
> ### 2. Machine（M）
>
> - M 代表操作系统线程，是 Go 运行时用来实际执行 Goroutine 的实体。
> - 每个 M 运行时负责执行 Goroutine 的代码，M 可以是操作系统的内核线程，Go 运行时会根据需要动态地创建或销毁 M。
> - M 负责执行具体的计算任务，它与 Goroutine 是一对多的关系，一个 M 可以执行多个 Goroutine，但一个 Goroutine 在某一时刻只能在一个 M 上执行。
> - 初始化数量：Go程序启动时，**初始M的数量通常等于`GOMAXPROCS`的值**（即逻辑处理器的数量，默认等于CPU核心数）。
>
> ### 3. Processor（P）
>
> - P 代表逻辑处理器，用来控制 Goroutine 的调度。P 的数量通常由用户通过 `GOMAXPROCS` 设置，表示可以同时运行 Goroutine 的最大并发数量（即逻辑 CPU 核心数）。
> - P 管理一个队列，队列中保存待执行的 Goroutine。当 P 和 M 关联时，P 会将自己队列中的 Goroutine 分配给 M 执行。
> - 每个 P 只会绑定一个 M，M 只能在与其关联的 P 上调度 Goroutine。
> - P 的数量限制了系统的并发度，即即使有很多 M 和 Goroutine，也只有 P 允许的数量并发执行。
>
> ### GMP 模型的工作原理
>
> 1. 当程序启动时，Go 运行时会初始化一组 P（逻辑处理器），P 的数量可以通过 `runtime.GOMAXPROCS` 设置，默认值是系统 CPU 核心数。
> 2. Goroutine 被创建时，运行时会将它放入某个 P 的本地队列中。
> 3. M 是操作系统的线程，当 M 启动时，它需要与一个 P 关联才能开始执行 Goroutine。P 决定将哪些 Goroutine 分配给 M 运行。
> 4. 如果 M 发现 P 的队列中没有可运行的 Goroutine，它会尝试从其他 P 的队列中窃取 Goroutine 来执行，这称为工作窃取（**work stealing**）。
> 5. 当一个 M 处于 I/O 阻塞状态或发生系统调用时，M 会释放 P，让其他 M 可以利用这个 P 继续执行 Goroutine。
> 6. 如果没有足够的 M 来运行 P 队列中的任务，Go 运行时会动态创建新的 M，反之如果有多余的 M，M 也会被销毁。
>
> 
>
> ###  work stealing 
>
> 当本线程⽆可运行的G时，尝试从其他线程绑定的P偷取G，⽽不是销毁线程。先全局队列，后本地队列
>
> ### hand off
>
> 当本线程因为G进行系统调用阻塞时，线程释放绑定的P，把P转移给其他空闲的线程执⾏。
>
> 
>
> ### go func()在GMP上的流程
>
> - 创建G ——》绑定到执行当前操作的M上的本地队列中，如果满了则放入全局队列 ——〉M调度并执行G ——》时间片消耗完之后加入本地队列
>
> - 当G发生阻塞，此时唤醒一个休眠的M或者新建来接管当前的P，之后阻塞的G与之前的M捆绑，当G唤醒之后被放入全局队列，旧的M加入休眠的M队列或销毁
>
> ![image-20250204下午20930874](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250204%E4%B8%8B%E5%8D%8820930874.png)
>
> #### 本地队列没有G了，那么会先去全局队列里拿还是先去其他P的队列里拿
>
> 1️⃣ **先尝试从全局运行队列（Global Queue）获取 G**
>
> ​	•	P **首先** 会去 **全局队列** 取 G，因为全局队列存放的是新建的或因系统调用被放回的 G。
>
> ​	•	取的数量通常是 **最多 32 个** 或全局队列里的一半（取较小值）。
>
> 
>
> 2️⃣ **如果全局队列为空，则尝试从其他 P 偷取（Work Stealing）**
>
> ​	•	P **随机** 选择 **其他 P 的本地队列**，尝试**偷取**（Work Stealing）一半的 G。
>
> ​	•	这种机制可以防止某些 P 任务过多，而另一些 P 处于空闲状态，提高调度效率。
>
> 
>
> 3️⃣ **如果仍然没有可运行的 G，那么 P 会进入休眠状态**
>
> ​	•	P 会通过 park 进入**休眠**状态，等待新的 G 可执行。
>
> ​	•	如果所有 P 都没有 G，那么 M（操作系统线程） 也可能进入休眠，最终 runtime 可能会进入 idle 状态。
>
> 
>
> 



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



## goroutine之间的通信方式

> 在 Go 语言中，**Goroutine 之间的通信**主要通过以下方式实现，每种方式适用于不同的场景：
>
> ---
>
> ### 1. **Channel（通道）**
> - **基本概念**：  
>   Channel 是 Go 语言中**最核心的通信机制**，遵循“通过通信共享内存，而非通过共享内存通信”的设计哲学。
> - **使用方式**：
>   ```go
>   // 无缓冲通道（同步通信）
>   ch := make(chan int)
>   go func() { ch <- 42 }() // 发送数据
>   val := <-ch              // 接收数据
>       
>   // 带缓冲通道（异步通信）
>   bufferedCh := make(chan int, 3)
>   bufferedCh <- 1 // 不阻塞，直到缓冲区满
>   ```
> - **适用场景**：
>   - 同步或异步传递数据。
>   - 控制 Goroutine 的执行顺序（如等待任务完成）。
> - **注意事项**：
>   - 无缓冲 Channel 需要发送和接收方同时就绪，否则会阻塞（可能死锁）。
>   - 带缓冲 Channel 需注意缓冲区大小和容量。
>
> ---
>
> ### 2. **共享内存 + 互斥锁（sync.Mutex）**
> - **基本概念**：  
>   通过共享变量结合锁（`sync.Mutex` 或 `sync.RWMutex`）实现数据同步。
> - **使用方式**：
>   ```go
>   var counter int
>   var mu sync.Mutex
>       
>   go func() {
>       mu.Lock()
>       counter++
>       mu.Unlock()
>   }()
>   ```
> - **适用场景**：
>   - 高频的局部状态更新（如计数器）。
>   - 需要直接操作共享数据结构的场景。
> - **注意事项**：
>   - 需手动管理锁，避免死锁或竞态条件。
>   - 过度使用会降低并发性能。
>
> ---
>
> ### 3. **WaitGroup（等待组）**
> - **基本概念**：  
>   `sync.WaitGroup` 用于等待一组 Goroutine 完成任务。
> - **使用方式**：
>   ```go
>   var wg sync.WaitGroup
>   wg.Add(2) // 等待 2 个任务
>       
>   go func() {
>       defer wg.Done()
>       // 任务1
>   }()
>       
>   go func() {
>       defer wg.Done()
>       // 任务2
>   }()
>       
>   wg.Wait() // 阻塞直到所有任务完成
>   ```
> - **适用场景**：
>   - 需要等待多个 Goroutine 全部结束。
>   - 替代简单的 Channel 同步逻辑。
>
> ---
>
> ### 4. **条件变量（sync.Cond）**
> - **基本概念**：  
>   `sync.Cond` 用于在特定条件下唤醒等待的 Goroutine。
> - **使用方式**：
>   ```go
>   var mu sync.Mutex
>   cond := sync.NewCond(&mu)
>   var ready bool
>       
>   // 等待条件满足
>   go func() {
>       cond.L.Lock()
>       for !ready {
>           cond.Wait() // 释放锁并阻塞，唤醒后重新加锁
>       }
>       cond.L.Unlock()
>   }()
>       
>   // 触发条件
>   cond.L.Lock()
>   ready = true
>   cond.Signal() // 唤醒一个等待的 Goroutine
>   cond.L.Unlock()
>   ```
> - **适用场景**：
>   - 复杂的条件等待（如资源池、任务队列）。
>
> ---
>
> ### 5. **原子操作（sync/atomic）**
> - **基本概念**：  
>   通过 `sync/atomic` 包直接操作内存中的基本类型（如 `int32`, `int64`），无需锁。
> - **使用方式**：
>   ```go
>   var counter int64
>   atomic.AddInt64(&counter, 1) // 原子递增
>   val := atomic.LoadInt64(&counter)
>   ```
> - **适用场景**：
>   - 高性能的简单计数器或状态标记。
>   - 替代锁的轻量级操作。
>
> ---
>
> ### 6. **Context（上下文）**
> - **基本概念**：  
>   `context.Context` 用于传递取消信号、超时或截止时间，控制 Goroutine 的生命周期。
> - **使用方式**：
>   ```go
>   ctx, cancel := context.WithTimeout(context.Background(), 2*time.Second)
>   defer cancel()
>       
>   go func(ctx context.Context) {
>       select {
>       case <-ctx.Done():
>           fmt.Println("任务超时或被取消")
>       case <-time.After(3*time.Second):
>           fmt.Println("任务完成")
>       }
>   }(ctx)
>   ```
> - **适用场景**：
>   
>   - 控制 Goroutine 的取消、超时或级联退出（如 HTTP 请求链）。
>
> ---
>
> ### 对比总结
> | 通信方式        | 特点                      | 适用场景                      |
> | --------------- | ------------------------- | ----------------------------- |
> | **Channel**     | 安全、解耦，支持同步/异步 | 数据传输、任务协调            |
> | **共享内存+锁** | 灵活但需手动管理          | 高频状态更新、复杂数据结构    |
> | **WaitGroup**   | 简单等待多个任务完成      | 批量任务同步                  |
> | **条件变量**    | 复杂条件触发              | 资源池、任务队列              |
> | **原子操作**    | 高性能无锁操作            | 计数器、标志位                |
> | **Context**     | 生命周期控制和信号传递    | 超时、取消、跨 Goroutine 控制 |
>
> ---
>
> ### 最佳实践
> 1. **优先使用 Channel**：简化同步逻辑，避免竞态条件。
> 2. **共享内存时加锁**：确保数据安全，但避免锁嵌套。
> 3. **明确通信目的**：
>    - 传数据 → Channel。
>    - 同步状态 → WaitGroup/Cond。
>    - 控制生命周期 → Context。



## goroutine上下文切换

### 切换时机

**(1) 主动让出（协作式调度）**

- 调用 `runtime.Gosched()`：主动让出 CPU，触发调度。
- 遇到**阻塞操作**（如通道操作、I/O 等待、`time.Sleep`）。
- 执行**系统调用**（Go 运行时可能将阻塞的系统调用转为异步模式）。

**(2) 抢占式调度（Preemption）**

- **基于信号的抢占**：Go 1.14+ 中，运行时通过发送 `SIGURG` 信号抢占长时间运行的 Goroutine。
- **时间片耗尽**：默认 Goroutine 最多执行 10ms，超时后会被抢占（需启用抢占机制）。

### 上下文切换内容

**存放在哪里**：保存在栈和调度器的「G 对象」中

> **(1) Goroutine 栈**
>
> - **存储位置**：每个 Goroutine 有自己的 **私有栈**，初始大小为 2KB，存放在 **堆内存**（由 Go 运行时管理）。
> - **动态扩展**：
>   - 栈空间不足时，运行时会分配更大的栈，并将旧栈内容复制到新栈。
>   - 栈缩容在 GC 时触发，检测到使用率过低则缩减。
> - **内容组成**：
>   - **局部变量**：函数调用链中的局部变量。
>   - **函数调用帧**：包括返回地址、参数、寄存器保存区。
>   - **defer 链**：延迟执行的函数列表。
>
> **(2) G 对象（Goroutine 元数据）**
>
> - **存储位置**：Go 运行时维护的全局数据结构（调度器队列）。
> - **内容组成**：
>   - **寄存器状态**：程序计数器（`PC`）、栈指针（`SP`）、通用寄存器（通过 `gobuf` 结构保存）。
>   - **调度信息**：Goroutine 状态（运行、阻塞等）、绑定的 M（线程）和 P（逻辑处理器）。
>   - **Channel 等待状态**：若因通道操作阻塞，记录等待的 Channel 和关联数据。
>   - **GC 元数据**：垃圾回收器追踪的栈和堆引用。

| **切换内容**          | **说明**                                                     |
| :-------------------- | :----------------------------------------------------------- |
| **寄存器状态**        | 包括程序计数器（`PC`）、栈指针（`SP`）、通用寄存器（如 `RAX`, `RBX` 等）。 |
| **栈指针（SP）**      | 指向当前 Goroutine 的私有栈，切换时需要更新为新 Goroutine 的栈地址。 |
| **调度元数据**        | 如 Goroutine 的状态（运行、阻塞等）、绑定的线程（M）、关联的处理器（P）等。 |
| **延迟函数（Defer）** | 当前 Goroutine 尚未执行的 `defer` 函数链。                   |
| **Channel 等待状态**  | /。                                                          |
| **GC 相关元数据**     | 垃圾回收器需要追踪的 Goroutine 栈和堆引用。                  |

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



## 内存泄漏

> 在 Go 语言中，虽然垃圾回收（GC）能自动管理内存，但若代码编写不当，仍可能导致 **内存泄漏**（即分配的内存无法被回收）。以下是常见的场景及示例：
>
> ---
>
> ### 一、Goroutine 泄漏
> **原因**：启动的 Goroutine 因阻塞而无法退出，导致其引用的内存无法释放。  
> **典型场景**：
> 1. **未关闭的 Channel 导致阻塞**：
>    ```go
>    func leak() {
>        ch := make(chan int)
>        go func() {
>            val := <-ch  // 等待数据，但 ch 永远不会被写入
>            fmt.Println(val)
>        }()
>    }
>    ```
>    **解决**：确保 Channel 有明确的关闭逻辑或超时机制。
>
> 2. **无限循环未退出**：
>    ```go
>    func leak() {
>        for {
>            go func() {
>                // 无限运行的 Goroutine
>                time.Sleep(1 * time.Second)
>            }()
>        }
>    }
>    ```
>    **解决**：使用 `context.Context` 或 `done` Channel 控制 Goroutine 生命周期。
>
> ---
>
> ### 二、未释放资源
> **原因**：打开的资源（文件、网络连接等）未关闭，导致相关内存无法释放。  
> **示例**：
> ```go
> func openFiles() {
>     for {
>         file, _ := os.Open("data.txt")
>         // 使用 file 但未调用 file.Close()
>     }
> }
> ```
> **解决**：使用 `defer` 或显式调用 `Close()` 释放资源。
>
> ---
>
> ### 三、全局变量或缓存无限增长
> **原因**：全局缓存（如 `map`）未设置淘汰策略，持续添加数据导致内存占用上升。  
> **示例**：
> ```go
> var cache = make(map[string][]byte)
> 
> func addToCache(key string, data []byte) {
>     cache[key] = data  // 缓存不断增长，无清理逻辑
> }
> ```
> **解决**：使用带容量限制的缓存（如 LRU 缓存库），或定期清理。
>
> ---
>
> ### 四、循环引用与根对象
> **原因**：虽然 Go 的 GC 能处理循环引用，但如果对象被全局变量或长期存活的对象引用，仍会导致泄漏。  
> **示例**：
> ```go
> var global *Object
> 
> type Object struct {
>     child *Object
> }
> 
> func createCycle() {
>     a := &Object{}
>     b := &Object{}
>     a.child = b
>     b.child = a
>     global = a  // 全局变量引用，导致 a 和 b 无法释放
> }
> ```
> **解决**：避免长期存活的对象引用无用数据。
>
> ---
>
> ### 五、未释放的 CGO 内存
> **原因**：通过 CGO 分配的内存（如 C 的 `malloc`）未手动释放。  
> **示例**：
> ```go
> /*
> #include <stdlib.h>
> */
> import "C"
> 
> func leak() {
>     ptr := C.malloc(1024)
>     // 未调用 C.free(ptr)
> }
> ```
> **解决**：使用 `defer C.free(ptr)` 确保释放。
>
> ---
>
> ### 六、时间包（time）的误用
> **原因**：未停止的 `time.Ticker` 或 `time.Timer` 会持续占用内存。  
> **示例**：
> ```go
> func leak() {
>     ticker := time.NewTicker(1 * time.Second)
>     defer ticker.Stop()  // 若忘记调用 Stop()，ticker 会持续触发
>     for range ticker.C {
>         // 业务逻辑
>     }
> }
> ```
> **解决**：始终调用 `ticker.Stop()` 或 `timer.Stop()`。
>
> ---
>
> ### 七、指针导致的意外引用
> **原因**：切片或结构体中的指针字段可能意外引用大对象。  
> **示例**：
> ```go
> type BigStruct struct {
>     data [1 << 20]byte  // 1MB 大对象
> }
> 
> func storePointer() {
>     var slice []*BigStruct
>     for {
>         big := &BigStruct{}
>         slice = append(slice, big)  // 切片持有大对象的指针
>     }
> }
> ```
> **解决**：及时清空无用的指针引用。
>
> ---
>
> ### 如何检测内存泄漏？
> 1. **使用 `pprof` 工具**：
>    ```go
>    import _ "net/http/pprof"
>    // 启动 HTTP 服务后，访问 /debug/pprof/heap 分析堆内存
>    ```
>
> 2. **监控 Goroutine 数量**：
>    ```go
>    go func() {
>        for {
>            log.Println("Goroutines:", runtime.NumGoroutine())
>            time.Sleep(1 * time.Second)
>        }
>    }()
>    ```
>
> 3. **使用 `runtime.ReadMemStats`**：
>    ```go
>    var m runtime.MemStats
>    runtime.ReadMemStats(&m)
>    fmt.Println("HeapAlloc:", m.HeapAlloc)
>    ```
>
> ---
>
> ### 总结
> Go 中的内存泄漏通常由以下原因导致：
> - **Goroutine 泄漏**（最常见）
> - **未释放资源**（文件、网络连接）
> - **全局缓存无限增长**
> - **CGO 内存未释放**
> - **误用时间包或第三方库**
>
> **关键预防措施**：
> - 使用 `defer` 确保资源释放。
> - 通过 `context` 或 `done` Channel 控制 Goroutine 生命周期。
> - 对全局缓存设置容量或过期策略。
> - 定期用 `pprof` 分析内存和 Goroutine。



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





# 设计模式

## 创建型模式

### 工厂模式

定义一个创建对象的接口，由子类决定实例化哪一个类

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250215%E4%B8%8B%E5%8D%8883412837.png" alt="image-20250215下午83412837" style="zoom:50%;" />

```go
package factory

type Product interface {
    Operation() string
}

type ProductA struct {}
func (p *ProductA) Operation() string {
    return "ProductA"
}

type ProductB struct {}
func (p *ProductA) Operation() string {
    return "ProductB"
}

type Factory interface {
    CreateProduct() Product
}

type ConcreteFactoryA struct {}
func (c *ConcreteFactoryA) CreateProduct() Product {
    return &ProductA{}
}

type ConcreteFactoryB struct {}
func (c *ConcreteFactoryA) CreateProduct() Product {
    return &ProductB{}
}

func main() {
  var factoryA Factory
  factoryA = new(ConcreteFactoryA)
  
  var product Product
  product = facotryA.CreateProduct()
  product.Operation()
}
```



### 单例模式

饿汉：

```go
package singleton

type singleton struct {}

var instance = &singleton{}

func GetInstance() *singleton {
    return instance
}
```



懒汉：

```go
//双检查锁
package singleton

import "sync"

type singleton struct {}

var (
    instance *singleton
    mu       sync.Mutex
)

// 传统双重检查锁实现
func GetInstance() *singleton {
    if instance == nil { // 第一次检查
        mu.Lock()
        defer mu.Unlock()
        
        if instance == nil { // 第二次检查
            instance = &singleton{}
        }
    }
    return instance
}


//sync.Once
package singleton

import "sync"

type singleton struct {}

var once sync.Once
var instance *singleton


func GetInstance() *singleton {
    once.Do(func() {
        instance = &singleton{}
    })
    return instance
}
```



## 结构型模式

### 代理模式

```go
package proxy

import "fmt"

type RealSubject interface {
    Request()
}

type RealSubjectImpl struct {}

func (r *RealSubjectImpl) Request() {
    fmt.Println("RealSubject: Handling request")
}

type Proxy struct {
    realSubject RealSubject
}

func (p *Proxy) Request() {
    fmt.Println("Proxy: Logging request")
    if p.realSubject == nil {
        p.realSubject = &RealSubjectImpl{}
    }
    p.realSubject.Request()
}
```

### 装饰器模式

通过将功能动态地附加到对象上，来扩展对象的功能，而不改变对象本身的结构

```go
package main

import "fmt"

// Component 接口，定义了基本行为
type Coffee interface {
    Cost() int
}

// SimpleCoffee 具体组件，实现了 Coffee 接口
type SimpleCoffee struct {}

func (c *SimpleCoffee) Cost() int {
    return 5 // 基础咖啡的价格
}

// Decorator 装饰器，包含对 Component 的引用
type CoffeeDecorator struct {
    coffee Coffee
}

func (d *CoffeeDecorator) Cost() int {
    return d.coffee.Cost() // 继承自 Coffee 的 Cost 方法
}

// MilkDecorator 具体装饰器，为 Coffee 增加牛奶
type MilkDecorator struct {
    CoffeeDecorator
}

func (m *MilkDecorator) Cost() int {
    return m.coffee.Cost() + 2 // 添加牛奶的费用
}

// SugarDecorator 具体装饰器，为 Coffee 增加糖
type SugarDecorator struct {
    CoffeeDecorator
}

func (s *SugarDecorator) Cost() int {
    return s.coffee.Cost() + 1 // 添加糖的费用
}

func main() {
    // 创建一个简单的咖啡
    coffee := &SimpleCoffee{}
    fmt.Println("Simple Coffee Cost:", coffee.Cost())

    // 使用 MilkDecorator 和 SugarDecorator 装饰咖啡
    milkCoffee := &MilkDecorator{CoffeeDecorator{coffee}}
    fmt.Println("Milk Coffee Cost:", milkCoffee.Cost())

    sugarMilkCoffee := &SugarDecorator{CoffeeDecorator{milkCoffee}}
    fmt.Println("Milk and Sugar Coffee Cost:", sugarMilkCoffee.Cost())
}
```

### 适配器模式

将一个类的接口转换成客户端所期望的另一个接口，从而使原本由于接口不匹配而无法一起工作的类能够协同工作。

```go
package main

import "fmt"

// 目标接口：新的系统需要的接口
type Sensor interface {
    ReadData() float64
}

// 适配者：旧的系统接口，无法直接使用
type OldSystem struct {}

func (o *OldSystem) GetTemperature() float64 {
    return 25.5 // 模拟返回温度值
}

// 适配器：将 OldSystem 的接口适配成 Sensor 接口
type SensorAdapter struct {
    oldSystem *OldSystem
}

func (a *SensorAdapter) ReadData() float64 {
    // 调用旧系统的 GetTemperature 方法
    return a.oldSystem.GetTemperature()
}

func main() {
    // 创建旧系统实例
    oldSystem := &OldSystem{}
    
    // 使用适配器将旧系统适配成新的接口
    sensor := &SensorAdapter{oldSystem: oldSystem}

    // 使用新的接口调用
    fmt.Printf("Temperature: %.2f\n", sensor.ReadData())
}
```



## 行为型模式

### 策略模式

定义一系列算法，将每一个算法封装起来，并使它们可以相互替换。

```go
package strategy

import "fmt"

type Strategy interface {
    Execute(a, b int) int
}

type AddStrategy struct {}
func (s *AddStrategy) Execute(a, b int) int {
    return a + b
}

type SubtractStrategy struct {}
func (s *SubtractStrategy) Execute(a, b int) int {
    return a - b
}

type Context struct {
    strategy Strategy
}

func (c *Context) SetStrategy(strategy Strategy) {
    c.strategy = strategy
}

func (c *Context) ExecuteStrategy(a, b int) int {
    return c.strategy.Execute(a, b)
}
```



### 观察者模式

允许对象在其状态发生变化时通知依赖它的所有对象

```go
package observer

import "fmt"

type Observer interface {
    Update(message string)
}

type ConcreteObserver struct {
    name string
}

func (o *ConcreteObserver) Update(message string) {
    fmt.Printf("%s received message: %s\n", o.name, message)
}

type Subject interface {
    Attach(observer Observer)
    Detach(observer Observer)
    Notify(message string)
}

type ConcreteSubject struct {
    observers []Observer
}

func (s *ConcreteSubject) Attach(observer Observer) {
    s.observers = append(s.observers, observer)
}

func (s *ConcreteSubject) Detach(observer Observer) {
    for i, o := range s.observers {
        if o == observer {
            s.observers = append(s.observers[:i], s.observers[i+1:]...)
            break
        }
    }
}

func (s *ConcreteSubject) Notify(message string) {
    for _, observer := range s.observers {
        observer.Update(message)
    }
}
```



# GIN

```go
package main

import "github.com/gin-gonic/gin"

func main() {
	r := gin.Default()
	r.GET("/ping", func(c *gin.Context) {
		c.JSON(200, gin.H{
			"message": "pong",
		})
	})
	r.Run() // 监听并在 0.0.0.0:8080 上启动服务
}
```



## bind

绑定传入参数，对参数进行类型校验（[可自定义验证器](https://docs.fengfengzhidao.com/#/docs/Gin%E6%A1%86%E6%9E%B6%E6%96%87%E6%A1%A3/4.bind%E7%BB%91%E5%AE%9A%E5%99%A8)）

- `ShouldBind`：`ShouldBindJSON`、`ShouldBindQuery`、`ShouldBindUri`...

###  验证器

```go
type UserInfo struct {
  Username string `json:"username" binding:"required" msg:"用户名不能为空"`
  Password string `json:"password" binding:"min=3,max=6" msg:"密码长度不能小于3大于6"`
  Email    string `json:"email" binding:"email" msg:"邮箱地址格式不正确"`
}

```

- 常用校验器

```go
// 不能为空，并且不能没有这个字段
required： 必填字段，如：binding:"required"  

// 针对字符串的长度
min 最小长度，如：binding:"min=5"
max 最大长度，如：binding:"max=10"
len 长度，如：binding:"len=6"

// 针对数字的大小
eq 等于，如：binding:"eq=3"
ne 不等于，如：binding:"ne=12"
gt 大于，如：binding:"gt=10"
gte 大于等于，如：binding:"gte=10"
lt 小于，如：binding:"lt=10"
lte 小于等于，如：binding:"lte=10"

// 针对同级字段的
eqfield 等于其他字段的值，如：PassWord string `binding:"eqfield=Password"`
nefield 不等于其他字段的值


- 忽略字段，如：binding:"-"

```







## 文件上传/下载

- 上传

```go
func main() {
	router := gin.Default()
	// 为 multipart forms 设置较低的内存限制 (默认是 32 MiB)
	router.MaxMultipartMemory = 8 << 20  // 8 MiB
	router.POST("/upload", func(c *gin.Context) {
		// 单文件
		file, _ := c.FormFile("file")
		log.Println(file.Filename)

		dst := "./" + file.Filename
		// 上传文件至指定的完整文件路径
		c.SaveUploadedFile(file, dst)

		c.String(http.StatusOK, fmt.Sprintf("'%s' uploaded!", file.Filename))
	})
	router.Run(":8080")
}
```

如何使用 `curl`：

```sh
curl -X POST http://localhost:8080/upload \
  -F "file=@/Users/appleboy/test.zip" \
  -H "Content-Type: multipart/form-data"
```





## 中间件和路由   

中间件：可用于**权限验证** or  **耗时统计**                       

```go      
func TimeMiddleware(c *gin.Context) {
  startTime := time.Now()
  c.Next()
  since := time.Since(startTime)
  // 获取当前请求所对应的函数
  f := c.HandlerName()
  fmt.Printf("函数 %s 耗时 %d\n", f, since)
}
```



https://docs.fengfengzhidao.com/#/docs/Gin%E6%A1%86%E6%9E%B6%E6%96%87%E6%A1%A3/6.%E4%B8%AD%E9%97%B4%E4%BB%B6%E5%92%8C%E8%B7%AF%E7%94%B1

```go
package main

import (
  "fmt"
  "github.com/gin-gonic/gin"
)

func middle(c *gin.Context) {
  fmt.Println("middle ...in")
}

func main() {
  router := gin.Default()

  r := router.Group("/api").Use(middle)  // 可以链式，也可以直接r.Use(middle)
  r.GET("/index", func(c *gin.Context) {
    c.String(200, "index")
  })
  r.GET("/home", func(c *gin.Context) {
    c.String(200, "home")
  })

  router.Run(":8080")
}

```

- gin.Default

gin.Default()默认使用了Logger和Recovery中间件，其中：

Logger中间件将日志写入gin.DefaultWriter，即使配置了GIN_MODE=release。 Recovery中间件会recover任何panic。如果有panic的话，会写入500响应码。 如果不想使用上面两个默认的中间件，可以使用gin.New()新建一个没有任何默认中间件的路由。

使用gin.New，如果不指定日志，那么在控制台中就不会有日志显示

```go
func Default() *Engine {
  debugPrintWARNINGDefault()
  engine := New()
  engine.Use(Logger(), Recovery())
  return engine
}
```

## 日志



# 其他  

## Go/C++区别

- 标准库容器

- 第三方库，包管理

  - C++：❌ 无官方工具（需用 vcpkg / Conan）依赖 CMake 和第三方工具
  - Golang：✅ Go Modules 官方支持

- 并发模型

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






## Go中变量的初始化顺序（全局/`init()`/）

1. 递归包内的变量/init()
2. 该包的全局变量
3. 该包的init()



```go
// mypkg/mypkg.go
package mypkg

import "fmt"

var X = initializeX()

func initializeX() int {
	fmt.Println("Initializing X in mypkg")
	return 42
}

func init() {
	fmt.Println("Executing init() in mypkg")
}
/////////////////////////

package main

import (
	"fmt"
	_ "mypkg" // 只触发 mypkg 的初始化，不使用其标识符
)

func init() {
	fmt.Println("Executing init() in main")
}

func main() {
	fmt.Println("Executing main()")
}
```

输出：

```bash
Initializing X in mypkg
Executing init() in mypkg
Executing init() in main
Executing main()
```





## 两个协程交叉打印，从1-100

- Go

```go
func main() {
	var wg sync.WaitGroup

	// 创建一个 channel，用于控制两个协程的顺序
	ch1 := make(chan struct{})
	ch2 := make(chan struct{})

	// 第一个协程，打印奇数
	wg.Add(1)
	go func() {
		defer wg.Done()
		for i := 1; i <= 100; i += 2 {
			fmt.Println(i)
			ch1 <- struct{}{}
			<-ch2

		}
	}()

	// 第二个协程，打印偶数
	wg.Add(1)
	go func() {
		defer wg.Done()
		for i := 2; i <= 100; i += 2 {
			<-ch1
			fmt.Println(i)
			ch2 <- struct{}{}
		}
	}()

	// 启动打印过程，给第一个协程一个信号

	wg.Wait()
}
```

- C++

```C++
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;
std::condition_variable cv_odd, cv_even;
bool is_odd_turn = true;  // 同步控制標誌

void print_odd() {
    for (int i = 1; i <= 100; i += 2) {
        std::unique_lock<std::mutex> lock(mtx);
        cv_odd.wait(lock, []{ return is_odd_turn; }); // 等待奇數輪次
        std::cout << "Odd: " << i << std::endl;
        is_odd_turn = false;         // 切換到偶數輪次
        cv_even.notify_one();        // 喚醒偶數線程
    }
}

void print_even() {
    for (int i = 2; i <= 100; i += 2) {
        std::unique_lock<std::mutex> lock(mtx);
        cv_even.wait(lock, []{ return !is_odd_turn; }); // 等待偶數輪次
        std::cout << "Even: " << i << std::endl;
        is_odd_turn = true;          // 切換回奇數輪次
        cv_odd.notify_one();         // 喚醒奇數線程
    }
}

int main() {
    std::thread t1(print_odd);
    std::thread t2(print_even);
    
    t1.join();
    t2.join();
    
    return 0;
}
```

