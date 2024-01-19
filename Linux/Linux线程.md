[TOC]



# 1 概念

- 可重入和不可重入：

> 在C++中，"可重入"（reentrant）和"不可重入"（non-reentrant）是描述函数在多线程环境中的行为的术语。
>
> 1. **可重入（Reentrant）**：
>    - 可重入函数是指一个函数可以同时被多个线程调用，而不会导致数据冲突或不一致性。
>    - 这通常通过确保函数内部使用的所有数据都是局部的、或者通过互斥锁来保护共享数据来实现。因此，可重入函数可以安全地在多线程环境中使用。
> 2. **不可重入（Non-reentrant）**：
>    - 不可重入函数是指在被调用时使用了全局数据，而且在多线程环境下可能会导致数据竞争或其他问题。
>    - 如果在多线程环境中使用不可重入函数，可能需要使用互斥锁等手段来确保函数的安全性。
>
> **为什么重入性很重要？** 在多线程应用程序中，如果有多个线程同时调用某个函数，而且该函数使用了共享数据，那么可能会出现竞争条件，导致数据不一致性、程序崩溃等问题。可重入函数的设计能够避免这些问题，提高程序的稳定性和可靠性。
>
> **示例：**
>
> ```cpp
> cppCopy code// 可重入函数的例子
> #include <iostream>
> #include <mutex>
> 
> std::mutex myMutex;
> 
> int reentrantFunction(int x) {
>     std::lock_guard<std::mutex> lock(myMutex);  // 使用互斥锁保护共享资源
>     return x * x;
> }
> 
> // 不可重入函数的例子
> int nonReentrantFunction(int x) {
>     static int sharedData = 0;  // 使用了静态变量，可能导致竞争条件
> 
>     // 在多线程环境下，可能导致 sharedData 数据竞争
>     sharedData += x;
> 
>     return sharedData;
> }
> 
> int main() {
>     // 在实际应用中，请根据具体情况使用可重入或不可重入函数
>     return 0;
> }
> ```

## 1.1 进程与线程的区别

1. 线程是CPU调度的基本单位
2. 进程是资源分配的基本单位

- 在`linux`中，线程在进程内部执行，线程在进程的地址空间内运行
- 线程执行进程代码的一部分

> 进程和线程是操作系统中的基本概念，它们之间有一些关键的区别：
>
> 1. **定义**：
>    - **进程**：进程是一个具有独立功能的程序关于某个数据集合的以此运行活动。它是系统进行资源分配和调度的独立单位，也是基本的执行单元。进程是一个动态的概念，是一个活动的实体。它不只是程序的代码，还包括当前的活动。进程结构特征：由程序、数据和进程控制块三部分组成。
>    - **线程**：线程是进程中的执行运算的最小单位，是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不拥有系统资源，只拥有一点在运行中必不可少的资源（程序计数器，一组寄存器和栈），但它可与同属一个进程的其他线程共享进程所拥有的全部资源。
> 2. **区别**：
>    - **资源分配**：进程是资源分配的基本单位，线程不拥有资源，但可以共享进程资源。
>    - **调度**：线程是CPU调度的基本单位，同一进程中的线程切换，不会引起进程切换；不同进程中的线程切换，会引起进程切换。
>    - **系统开销**：进程的创建和销毁时，系统都要单独为它分配和回收资源，开销远大于线程的创建和销毁；进程的上下文切换需要保存更多的信息，线程（同一进程中）的上下文切换系统开销更小。
>    - **通信方式**：进程拥有各自独立的地址空间，进程间的通信需要依靠`IPC`；线程共享进程资源，线程间可以通过访问共享数据进行通信。
> 3. **关系**：
>    - 一个程序至少有一个进程,一个进程至少有一个线程，一个线程只属于一个进程.
>    - 资源分配给进程，同一进程的所有线程共享该进程的所有资源。
>    - 处理机分给线程，即真正在处理机上运行的是线程。

## 1.2 线程独占和共享的部分

1. 线程共享进程数据，但也拥有自己的一部分数据:

> - 线程ID
> - **一组寄存器**
> - **栈**
> - `errno`
> - 信号屏蔽字
> - 调度优先级

2. 进程的多个线程共享 同一地址空间,因此`Text Segment`、`Data Segment`都是共享的,如果定义一个函数,在各线程中都可以调用,如果定义一个全局变量,在各线程中都可以访问到,除此之外,各线程还共享以下进程资源和环境:

> - 文件描述符表
> - 每种信号的处理方式(`SIG_IGN`、`SIG_DFL`或者自定义的信号处理函数)
> - 当前工作目录
> - 用户id和组id

# 2 `pthread`线程库（用户级）

- `linux`内核中没有很明确的线程概念，只有轻量级进程，没有直接提供线程的系统调用，只会给轻量级进程的系统调用

## 2.1 `pthread_create`

![image-20231203231834059](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231203231834059.png)

1. 参数一：输出型参数，thread_id
2. 参数二：线程属性，一般为`nullptr`
3. 参数三：函数指针，给出线程执行的函数：返回值和参数均为`void*`
4. 参数四：创建线程成功，传给线程启动函数的参数
5. 返回值：0表示成功，非零表示错误码

- 编译时需要链接静态库

```bash
g++ test.cc -o test -lpthread
```

- 查看线程

```bash
ps -aL
```

同一组线程`PID`相同，`LWP`(light weight process)不同，小的那一个是主线程

## 2.2 线程等待 —— `pthread_join`

![image-20231203234132146](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231203234132146.png)

- 默认阻塞等待

1. 参数1：等待的线程（`pthread_create`的第一个输出型参数）
2. 参数2：输出型参数，获取线程执行函数的返回值（`pthread_create`的第三个参数的返回值）

- 实例：

```cpp
/*该程序上实现了一个通过创建Request对象传入一个线程中，
计算一个`start_`到`end_`的总和，
最后将答案通过Response对象返回的功能*/
#include <iostream>
#include <unistd.h>
using namespace std;

class Request
{
public:
    Request(int start, int end, const string &threadname)
    : start_(start), end_(end), threadname_(threadname)
    {}
public:
    int start_;
    int end_;
    string threadname_;
};

class Response
{
public:
    Response(int result, int exitcode):result_(result),exitcode_(exitcode)
    {}
public:
    int result_;   // 计算结果
    int exitcode_; // 计算结果是否可靠
};

void *sumCount(void *args) // 线程的参数和返回值，不仅仅可以用来进行传递一般参数，也可以传递对象！！
{
    Request *rq = static_cast<Request*>(args); //  Request *rq = (Request*)args
    Response *rsp = new Response(0,0);
    for(int i = rq->start_; i <= rq->end_; i++)
    {
        cout << rq->threadname_ << " is runing, caling..., " << i << endl;
        rsp->result_ += i;
        usleep(100000);
    }
    return rsp;
}

int main()
{
    pthread_t tid;
    Request *rq = new Request(1, 100, "thread 1");
    pthread_create(&tid, nullptr, sumCount, rq);   //rq作为sumCount函数的实参


    void *ret;
    pthread_join(tid, &ret);
    Response *rsp = static_cast<Response *>(ret);
    cout << "rsp->result: " << rsp->result_ << ", exitcode: " << rsp->exitcode_ << endl;
    delete rsp;
    return 0;
}
```



## 2.3 线程退出终止

### 2.3.1 `pthread_exit`

- **多线程没有内存隔离，单个线程崩溃会导致整个应用程序的退出**

- `exit`不能用来终止线程
- 通过`pthread_exit`终止一个线程
- **主线程**调用`pthread_exit`只是退出主线程，并不会导致进程的退出

![image-20231204131827999](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231204131827999.png)

### 2.3.2 `pthread_cancel`

- 如果主线程调用`pthread_cancel(pthread_self())`函数来退出自己， 则主线程对应的轻量级进程状态变更成为`Z`， 其他线程不受影响，这是正确的（正常情况下我们也不会这么做....）

![image-20231204132034479](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231204132034479.png)

一个线程被取消，它的返回值为`PTHREAD_CANCELED`值为-1

```cpp
#define PTHREAD_CANCELED ((void *)-1)
```



## 2.4 线程id

- 返回线程id——`pthread_self`

![image-20231204202719449](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231204202719449.png)

 

## 2.5 分离线程 —— `pthread_detach`

- 情况下，新创建的线程是`joinable`的，线程退出后，需要对其进行`pthread_join`操作，否则无法释放
  资源，从而造成系统泄漏。
- 如果不关心线程的返回值，join是一种负担，这个时候，我们可以告诉系统，当线程退出时，自动释放线
  程资源。

- 主线程将其他线程分离，或者线程自己选择分离

![image-20231205204702737](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231205204702737.png)

# 3 C++并发支持库

> 参考：
>
> 1. https://paul.pub/cpp-concurrency/
> 2. https://en.cppreference.com/w/cpp/thread

## 3.1 线程相关

![image-20240118202712108](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240118202712108.png)

## 3.2 锁 : `mutex`/`lock`

### 3.2.1 mutex

![image-20240119213118361](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240119213118361.png)



### 3.2.2 lock & 封装lock的RAII类

- lock: 同时申请锁，其中一个申请不到则阻塞，避免deadlock

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240118225006871.png)

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240118215838439.png)

- 使用RAII的类：`lock_guard`, `uniqur_lock`, `shared_lock`, `scoped_lock`

![image-20240118220404402](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240118220404402.png)

- **在C++17之后删除了类似于`unique_lock<mutex>`的模板类型，改为直接的`unique_lock`类型**

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240119223241861.png)

```cpp
#include <chrono>
#include <functional>
#include <iostream>
#include <mutex>
#include <string>
#include <thread>
#include <vector>
using namespace std::chrono_literals;
 
struct Employee
{
    std::vector<std::string> lunch_partners;
    std::string id;
    std::mutex m;
    Employee(std::string id) : id(id) {}
    std::string partners() const
    {
        std::string ret = "Employee " + id + " has lunch partners: ";
        for (int count{}; const auto& partner : lunch_partners)
            ret += (count++ ? ", " : "") + partner;
        return ret;
    }
};
 
void send_mail(Employee&, Employee&)
{
    // Simulate a time-consuming messaging operation
    std::this_thread::sleep_for(1s);
}
 
void assign_lunch_partner(Employee& e1, Employee& e2)
{
    static std::mutex io_mutex;
    {
        std::lock_guard<std::mutex> lk(io_mutex);
        std::cout << e1.id << " and " << e2.id << " are waiting for locks" << std::endl;
    }
 
    {
        // Use std::scoped_lock to acquire two locks without worrying about
        // other calls to assign_lunch_partner deadlocking us
        // and it also provides a convenient RAII-style mechanism
 
        std::scoped_lock lock(e1.m, e2.m);
 
        // Equivalent code 1 (using std::lock and std::lock_guard)
        // std::lock(e1.m, e2.m);
        // std::lock_guard<std::mutex> lk1(e1.m, std::adopt_lock);
        // std::lock_guard<std::mutex> lk2(e2.m, std::adopt_lock);
 
        // Equivalent code 2 (if unique_locks are needed, e.g. for condition variables)
        // std::unique_lock<std::mutex> lk1(e1.m, std::defer_lock);
        // std::unique_lock<std::mutex> lk2(e2.m, std::defer_lock);
        // std::lock(lk1, lk2);
        {
            std::lock_guard<std::mutex> lk(io_mutex);
            std::cout << e1.id << " and " << e2.id << " got locks" << std::endl;
        }
        e1.lunch_partners.push_back(e2.id);
        e2.lunch_partners.push_back(e1.id);
    }
 
    send_mail(e1, e2);
    send_mail(e2, e1);
}
 
int main()
{
    Employee alice("Alice"), bob("Bob"), christina("Christina"), dave("Dave");
 
    // Assign in parallel threads because mailing users about lunch assignments
    // takes a long time
    std::vector<std::thread> threads;
    threads.emplace_back(assign_lunch_partner, std::ref(alice), std::ref(bob));
    threads.emplace_back(assign_lunch_partner, std::ref(christina), std::ref(bob));
    threads.emplace_back(assign_lunch_partner, std::ref(christina), std::ref(alice));
    threads.emplace_back(assign_lunch_partner, std::ref(dave), std::ref(bob));
 
    for (auto& thread : threads)
        thread.join();
    std::cout << alice.partners() << '\n'  << bob.partners() << '\n'
              << christina.partners() << '\n' << dave.partners() << '\n';
}
```

### 3.2.3 Call once

![image-20240118233738777](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240118233738777.png)

## 3.3 条件变量



![image-20240119213920600](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240119213920600.png)

### 3.3.1`condition_variable`的方法

![image-20240119222028467](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240119222028467.png)

- `wait`原理与用法

`wait`阻塞当前进程， 直到`notify_all`或`notify_one()`，之后循环判断直到满足谓词`bool(stop_wating()) == true`，

详细例子可以看下面的代码

![image-20240119232925584](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240119232925584.png)

- `native_handle`：返回本机句柄，用法如下：

```cpp
/*以下伪代码用于展示native_handle的用法*/
std::mutex m;
std::condition_variable cv;   

// cv.wait(lk, []{ return ready; });
    while (!ready) {
        pthread_cond_wait(cv.native_handle(), m.native_handle());
    }
```

- 条件变量与锁使用，实现同步互斥

```cpp

#include <condition_variable>
#include <iostream>
#include <mutex>
#include <string>
#include <thread>

std::mutex m;
std::condition_variable cv;
std::string data_;
bool ready = false;
bool processed = false;

void worker_thread()
{
    // Wait until main() sends data
    std::unique_lock lk(m);
    cv.wait(lk, [] { return ready; });

    // after the wait, we own the lock.
    std::cout << "Worker thread is processing data\n";
    data_ += " after processing";

    // Send data back to main()
    processed = true;
    std::cout << "Worker thread signals data processing completed\n";

    // Manual unlocking is done before notifying, to avoid waking up
    // the waiting thread only to block again (see notify_one for details)
    lk.unlock();
    cv.notify_one();
}

int main()
{
    std::thread worker(worker_thread);

    data_ = "Example data";
    // send data to the worker thread
    {
        std::lock_guard lk(m);
        ready = true;
        std::cout << "main() signals data ready for processing\n";
    }
    cv.notify_one();

    // wait for the worker
    {
        std::unique_lock lk(m);
        cv.wait(lk, [] { return processed; });
    }
    std::cout << "Back in main(), data = " << data_ << '\n';

    worker.join();
}
```



## 3.1 `std::thread::detach`

主线程已经结束之后，被分离的线程无法在正常执行

```cpp
//windows下，结果test1创建，而test2没有创建
#include <iostream>
#include <thread>
#include <chrono>    
using namespace std;
using namespace std::chrono_literals;
void f() {
    this_thread::sleep_for(10s);
    system("mkdir C:\\Users\\ASUS\\Desktop\\test2");   
}
int main() {
    thread t{ f };
    system("mkdir C:\\Users\\ASUS\\Desktop\\test1");
    t.detach();
    return 0;
}
```



# 4 线程库的底层细节

- 用户级线程库`pthread`为动态库，使用时加载到内存，映射到主线程的共享区中

- `tid`地址为一个用户级`tcb`的起始地址

  ![image-20231204204313130](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231204204313130.png)

- 线程库底层通过调用`clone`系统调用函数创建轻量级进程

![image-20231204204437356](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231204204437356.png)

## 4.1 独立栈结构

除了主线程，所有其他线程的独立栈都在共享区，具体来讲是在`pthread`库中，`tid`指向的用户`tcb`中

```cpp
/*通过以下程序打印结果可知，tid和位于线程栈的局部变量均位于共享区中*/
void* func(void* arg) {
    int i = 10;
    int j = 1;
    while(i--) {
        pthread_t tid = pthread_self();
        //tid和位于线程栈的局部变量均位于共享区中
        printf("thread_number : %d, tid: %x, stack_var: %x\n", *((int*)arg), &tid, &j);
        sleep(1);
    }
}
int main() {
    vector<pthread_t> tids;
    for (int i = 0; i < 3; i++) {
        pthread_t tid;
        int* number = new int(i);
        pthread_create(&tid, nullptr, func, number);
        tids.push_back(tid);
    }
    for (int i = 0; i < 3; i++) {
        pthread_join(tids[i], nullptr);
    }
}

```

![image-20231205201344516](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231205201344516.png)

![image-20231120191422225](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231120191422225.png)

## 4.2 线程的局部存储

如果线程想要独自占有全局变量，需要在主线程代码的变量前带**`__pthread`编译选项**，称为线程的局部存储

```cpp
__pthread int global_var = 10;
```

- `__pthread`只能定义内置类型

## 4.3 `LWP`

- "用户级线程 + 内核的`LWP` = Linux线程"

> 问：`LWP`与线程id的区别？
>
> 答：在Linux中，线程ID（Thread ID）和轻量级进程ID（Lightweight Process ID，LWP ID）是两个不同的概念，但它们之间存在密切的关系。
>
> 1. **线程ID**：这是由线程库（如`pthread`库）为每个线程分配的唯一标识符。这个ID只在当前**进程内部唯一**，不能保证在系统级别（即跨进程）唯一。你可以通过`pthread_self()`函数获取到这个ID。
> 2. **LWP ID**：这是由操作系统内核为每个轻量级进程（在Linux中，每个线程都被视为一个轻量级进程）分配的唯一标识符。这个ID在**整个系统中是唯一的**，即使在不同的进程中也是如此。你可以通过系统调用`syscall(SYS_gettid)`来获取这个ID。
>
> 总的来说，**线程ID是在用户空间中的概念，而`LWP ID`则是在内核空间中的概念**。在大多数情况下，你可以将`LWP ID`视为线程在内核中的真实ID。

# 5 线程互斥

> 问：什么是线程互斥？
>
> 答：线程互斥是指当有若干个线程访问同一块资源时，规定同一时间只有一个线程可以得到访问权，其它线程需要等占用资源者释放该资源才可以申请访问。这是一种特殊的线程同步。简单来说，线程互斥就是为了防止多个线程同时访问和修改同一块数据，从而引发数据不一致的问题

## 5.1 锁是原子操作

-  锁本身就是共享资源，但是申请和释放锁都是**原子性操作**

内存中的`mutex`初始值值为1表示此时没有线程持有锁，CPU中的`al`寄存器值为1表示当前线程持有锁，0表示不持有锁

![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231207123945078.png)[^1]

[^1]:`xchgb`表示**交换**`al`寄存器与`mutex`的值，当`mutex==1`时，交换使得`mutex==0,return 0`; 当`mutex==0`时挂起等待

## 5.2 互斥锁`mutex`库函数

### 5.2 .1 `mutex`创建和销毁

![image-20231207111434374](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231207111434374.png)

- 第一种，将锁定义成局部变量

```cpp
int main() {
    pthread_mutex_t lock;
    pthread_mutex_init(&lock, nullptr);
    /*
        
    */
   pthread_mutex_destroy(&lock);  
}
```

- 第二种，将锁定义成全局变量：**不需要手动初始化锁和释放锁**

```cpp
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

### 5.2.2 对临界资源加锁

![image-20231207112130106](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231207112130106.png)

## 5.3 饥饿问题

- 一个线程持续拥有锁（死锁），导致其他线程饥饿

```cpp
/*以下程序描述：
	多个线程执行抢票程序，并对tickets的操作代码块（临界区）上锁
*/

#include <iostream>
#include <unistd.h>
#include <vector>
using namespace std;

int tickets = 1000;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

//多线程执行该函数
void *getTicket(void *args)
{
    string name = "thread-" + to_string(*(int*)args);
    while (true)
    {
        // 线程对于锁的竞争能力可能会不同
        pthread_mutex_lock(&lock); // 申请锁成功，才能往后执行，不成功，阻塞等待。
        if(tickets > 0)
        {
            cout << "who: " << name << ", get a ticket: " << tickets << endl;
            tickets--;   //非原子操作
            pthread_mutex_unlock(&lock);
        }
        else{
            pthread_mutex_unlock(&lock);
            break;
        }
        //防止一个线程持续拥有锁（死锁），导致其他线程饥饿
        //如果没有sleep，该线程解锁之后会立马申请锁，导致一个线程持续占有锁
        usleep(13); 
    }
    cout << name << "... quit" << endl;
    return nullptr;
}
int main()
{
    vector<pthread_t> tids;
    for (int i = 1; i <= 5; i++){
        int* no = new int(i);   //作为线程序号
        pthread_t tid;     
        pthread_create(&tid, nullptr, getTicket, no);    //创建线程并执行
        tids.push_back(tid); 
    }

    for (auto thread : tids){
        pthread_join(thread, nullptr);
    }

    return 0;
}
```

##  5.4 死锁

### 5.4.1 产生死锁的四个必要条件 —— 《操作系统精髓与设计原理》P169

1. 互斥
2. 持有并等待
3. 不可抢占
4. 循环等待

> - 死锁预防
> - 死锁避免 
>   - 进程启动拒绝
>   - 资源分配拒绝 —— 银行家算法（安全/不安全状态） —— P171
> - 死锁检测
> - **哲学家就餐问题**

# 6 线程同步

- 保证数据安全的情况下，让线程访问资源具有一定的顺序性

> 线程同步是指在多线程编程中，为了确保多个线程之间正确、有序地执行，采取的一种机制或手段。在多线程环境中，多个线程可能同时访问共享的资源，如共享内存区域、文件、网络连接等，如果不加以控制，可能导致数据的不一致性、竞态条件等问题。线程同步的目的就是通过合适的同步机制来确保线程之间按照一定的顺序协调工作，从而避免潜在的问题

## 6.1 条件变量

- 通过条件变量实现线程同步

- 申请锁失败，到条件变量“排队”（条件变量必须依赖于锁的使用）

### 6.1.1 初始化与销毁（类比`mutex`）

![image-20231209132621006](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20231209132621006.png)

- 参数二`attr`：线程属性，一般设置为`nullptr`

### 6.1.2 等待条件满足 —— `pthread_cond_wait`

- `pthread_cond_wait` **让线程等待的时候，会自动释放锁**，并让线程阻塞
- 访问临界资源的**判断**必须放在加锁之后的代码中 ？？？为什么

### 6.1.3 唤醒等待

- 唤醒一个线程 —— `pthread_cond_signal`

- 唤醒所有线程 —— `pthread_cond_broadcast`

```cpp
#include <iostream>
#include <pthread.h>
#include <unistd.h>

using namespace std;

int cnt = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

void* Count(void* args) {
    pthread_detach(pthread_self());   //detach之后的线程不需要join
    int number = (uint64_t)args;
    cout << "thread: " << number << "create success" << endl;

    while(true) {
        pthread_mutex_lock(&mutex);
        pthread_cond_wait(&cond, &mutex);   //释放锁并阻塞等待

        cout << "thread: " << number << ", cnt: " << cnt++ << endl;
        pthread_mutex_unlock(&mutex);
    }
}

int main() {
    for (uint64_t i = 0; i < 5; i++) {
        pthread_t tid;
        pthread_create(&tid, nullptr, Count, (void*)i);
        usleep(1000);   //让线程按顺序创建
    }

    while(true) {
        sleep(1);
        pthread_cond_signal(&cond);    //唤醒在cond的等待队列中的一个线程，默认是第一个
    }
    return 0;
}
```

## 6.2 生产者/消费者问题

### 6.2.1 并行与并发的区别

> 1. **并 行（Parallelism）：**
>    - **定义：** 并行是指在**<u>同一时刻</u>**执行多个任务，这些任务可以是同一程序的不同部分，也可以是不同程序的任务。并行处理的目标是通过同时执行多个操作来提高整体系统的性能。
>    - **特点：** 并行性强调同时发生，通常指多个处理单元在同一时刻执行不同的任务。这可以通过在多个处理器上同时执行多个线程或进程来实现。
> 2. **并发（Concurrency）：**
>    - **定义：** 并发是指在<u>**一段时间**</u>内，多个任务都在执行，但不一定同时执行。并发强调任务之间的独立性，系统可以在执行任务之间快速切换，以便看起来像是同时执行的。
>    - **特点：** 并发性强调任务的独立性和交替执行，可以在单一处理器上通过时间片轮转或事件驱动的方式实现。

- 生产者消费者模型的高效体现在非临界区上的并发执行

  

```cpp
//生产者消费者问题
//阻塞队列
//生产者线程通过push向队列里生产数据
//消费者线程通过pop从队列里取出数据
#include <queue>
#include <pthread.h>

template<typename T> 
class block_queue {
    static const int default_capacity = 10;
public:
    block_queue(capacity = default_capacity) {
        pthread_mutex_init(&_mutex, nullptr);
        pthread_cond_init(&_c_cond, nullptr);
        pthread_cond_init(&_p_cond, nullptr);
    }

    T& pop() {
        pthread_mutex_lock(&_mutex);
        //防止伪唤醒，用while，而不用if，当被唤醒时再判断一次
        while (q.empty()) {    //判断队列是否为空也是在访问临界资源
            pthread_cond_wait(&_c_cond, &_mutex);   //等待并释放锁
        }
        T& ret = q.front();
        q.pop();

        pthread_cond_signal(&_p_cond);    
        pthread_mutex_unlock(&_mutex);
    }
    
    void push(const T& data) {
        pthread_mutex_lock(&_mutex);
        while (q.size() == capacity) {     //队列满了，加入等待队列
            pthread_cond_wait(&_p_cond, &_mutex);
        }
        q.push(data);

        pthread_cond_signal(&_c_cond);    //唤醒消费者的等待队列
        pthread_mutex_unlock(&_mutex);   //唤醒与解锁这两行代码能否交换位置？ 可以， 为什么？
    }
    ~block_queue() {
        pthread_mutex_destroy(&_mutex);
        ptrehad_cond_destroy(&_c_cond);
        ptrehad_cond_destroy(&_p_cond);

    }

private:
    int capacity;
    queue<T> q;
    pthread_mutex_t _mutex;
    pthread_cond_t _c_cond;   //消费者的条件变量
    pthread_cond_t _p_cond;   //生产者的条件变量
};
```



# 7 POSIX信号量

- 本质是一个计数器，描述临界资源中资源数量的多少，保证其`PV`操作时原子的 

```cpp
P() //-
//访问资源
V() //+
```

## 7.1 基本操作

### 7.1.1初始化信号量

![image-20240116103417518](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240116103417518.png)

- 参数
  -  `sem`：输出型参数
  - `pshared`: 0表示线程间共享，非0表示进程间共享
  - `value`：信号量的初始值

### 7.1.2 销毁

```cpp 
int sem_destroy(sem_t *sem);
```

### 7.1.3 PV操作

```cpp
//P() -> --
int set_wait(sem_t *sem);
//V() -> ++
int set_post(sem_t *sem);

```

## 7.2 基于信号量的环形生产者-消费者问题

```cpp
/* SapceSem表示剩余空间，初始值为n
   DateSem表示剩余生产
//消费者：
{
    P(DateSem);
    //消费
    V(SpaceSem)
}
*/
#include <semaphore.h>
#include <vector>
using namespace std;

template<typename T>
class circular_queue {
public:
    circular_queue(int capacity = 5) : capacity_(capacity), pos_p(0), pos_c(0) {
        sem_init(&remain, 0, capacity_);   //剩余空间的信号量初始值为总容量
        sem_init(&ouput, 0, 0);
        pthread_mutex_init(&mutex_p, nullptr);
        pthread_mutex_init(&mutex_c, nullptr);
    }
    void push(T d) {
        P(&remain);     //对剩余空间的信号量--，当remain为0时，表示没有剩余空间，阻塞在当前位置，等待消费

        lock(&mutex_p);     //考虑多个线程生产数据，保证原子性
        v[pos_p] = d;
        pos_p = (pos_p + 1) % capacity_;
        unlock(&mutex_p); 

        V(&ouput);      //对生产数据的信号量++
    }   
    T& pop() {
        P(&ouput);      //对生产量的信号量--，当ouput为0时，表示没有剩余产品，阻塞在当前位置，等待生产

        lock(&mutex_c);
        T& ret = v[pos_c];
        pos_c = (pos_c + 1) % capacity_;
        unlock(&mutex_c);

        V(&remain);
        return ret;
    }
    ~circular_queue() {
        sem_destroy(&remain);
        sem_destroy(&ouput);
        pthread_mutex_destroy(&mutex_c);
        pthread_mutex_destroy(&mutex_p);
    }
private:
    void P(sem_t* sem) { sem_wait(sem); }
    void V(sem_t* sem)  {sem_post(sem); }
    void lock(pthread_mutex_t* mutex) { pthread_mutex_lock(mutex); }
    void unlock(pthread_mutex_t* mutex) { pthread_mutex_unlock(mutex); }

private:
    int pos_p,pos_c;    //生产消费的位置
    vector<T> cir_queue;
    sem_t remain, ouput;    //信号量，remian:队列剩余空间， output:生产的数据量
    pthread_mutex_t mutex_p, mutex_c;   
    int capacity_;
}; 
```



# 8 线程安全的单例模式

 ## 8.1 饿汉模式实现单例

```cpp
template <typename T>
class Singleton {
    static T data;
public:
    static T* GetInstance() {
        return &data;
    }
};
```

## 8.2 懒汉模式实现单例

```cpp
template <typename T>
class Singleton {
    static T* data;
public:
    static T* GetInstance {
  		if (data == nullptr) { 		//线程不安全 
            data = new T();    
        }      
        return data;
    }
};
```

## 8.3 单例模式的线程池

```cpp
#pragma once

#include <iostream>
#include <vector>
#include <string>
#include <queue>
#include <pthread.h>
#include <unistd.h>

struct ThreadInfo
{
    pthread_t tid;
    std::string name;
};

static const int defalutnum = 5;

template <class T>
class ThreadPool
{
public:
    void Lock()
    {
        pthread_mutex_lock(&mutex_);
    }
    void Unlock()
    {
        pthread_mutex_unlock(&mutex_);
    }
    void Wakeup()
    {
        pthread_cond_signal(&cond_);
    }
    void ThreadSleep()
    {
        pthread_cond_wait(&cond_, &mutex_);
    }
    bool IsQueueEmpty()
    {
        return tasks_.empty();
    }
    std::string GetThreadName(pthread_t tid)
    {
        for (const auto &ti : threads_)
        {
            if (ti.tid == tid)
                return ti.name;
        }
        return "None";
    }

public:
    static void *HandlerTask(void *args)
    {
        ThreadPool<T> *tp = static_cast<ThreadPool<T> *>(args);
        std::string name = tp->GetThreadName(pthread_self());
        while (true)
        {
            tp->Lock();

            while (tp->IsQueueEmpty())
            {
                tp->ThreadSleep();
            }
            T t = tp->Pop();
            tp->Unlock();

            t();
            std::cout << name << " run, " << "result: " << t.GetResult() << std::endl;
        }
    }
    void Start()
    {
        int num = threads_.size();
        for (int i = 0; i < num; i++)
        {
            threads_[i].name = "thread-" + std::to_string(i + 1);
            pthread_create(&(threads_[i].tid), nullptr, HandlerTask, this);
        }
    }
    T Pop()
    {
        T t = tasks_.front();
        tasks_.pop();
        return t;
    }
    void Push(const T &t)    // 
    {
        Lock();
        tasks_.push(t);
        Wakeup();
        Unlock();
    }
    static ThreadPool<T> *GetInstance()     //获得线程池对象
    {
        if (nullptr == tp_) //创建对象完之后，避免加锁解锁使效率降低
        {
            pthread_mutex_lock(&lock_);   //保证线程安全
            if (nullptr == tp_)
            {
                std::cout << "log: singleton create done first!" << std::endl;
                tp_ = new ThreadPool<T>();
            }
            pthread_mutex_unlock(&lock_); 
        }

        return tp_;
    }

private:
    ThreadPool(int num = defalutnum) : threads_(num)
    {
        pthread_mutex_init(&mutex_, nullptr);
        pthread_cond_init(&cond_, nullptr);
    }
    ~ThreadPool()
    {
        pthread_mutex_destroy(&mutex_);
        pthread_cond_destroy(&cond_);
    }
    ThreadPool(const ThreadPool<T> &) = delete;
    const ThreadPool<T> &operator=(const ThreadPool<T> &) = delete; 
private:
    std::vector<ThreadInfo> threads_;
    std::queue<T> tasks_;

    pthread_mutex_t mutex_;
    pthread_cond_t cond_;

    static ThreadPool<T> *tp_;
    static pthread_mutex_t lock_;
};

template <class T>
ThreadPool<T> *ThreadPool<T>::tp_ = nullptr;

template <class T>
pthread_mutex_t ThreadPool<T>::lock_ = PTHREAD_MUTEX_INITIALIZER;    //给创建线程池对象上锁
```

