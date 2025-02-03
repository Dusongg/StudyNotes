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





