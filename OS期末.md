### 1 进程、线程概念

进程是程序执行的一个实例，它包含了程序的代码、数据以及执行状态。每个进程都有独立的地址空间、代码、数据和堆栈。操作系统通过进程来管理资源和任务。

线程是进程中的一个执行单元，是程序执行流的最小单位。一个进程可以包含多个线程，共享同一个地址空间。

### 2 CPU调度算法，评价指标（周转时间、[平均等待时间](https://blog.csdn.net/DreamWendy/article/details/118077621)等）

调度算法评价指标https://blog.csdn.net/DreamWendy/article/details/118077621

![image-20250104142354280](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250104142354280.png)

>例题：
>
>1. [最高响应比优先算法（HRRF）各类时间计算](https://blog.csdn.net/m0_61709053/article/details/135105147)
>
>   ![image-20250107140846177](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250107140846177.png)
>
>2. ![123](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250105143920914.png)



### 3 同步问题（信号量、互斥锁解决）

1. [哲学家就餐](https://xiaolincoding.com/os/4_process/multithread_sync.html#%E5%93%B2%E5%AD%A6%E5%AE%B6%E5%B0%B1%E9%A4%90%E9%97%AE%E9%A2%98)

2. [读者-写者问题](https://xiaolincoding.com/os/4_process/multithread_sync.html#%E8%AF%BB%E8%80%85-%E5%86%99%E8%80%85%E9%97%AE%E9%A2%98)

3. [公交车售票问题](https://blog.csdn.net/qq_43582207/article/details/130849341)

4. 橘子苹果问题（lab6最后一个实验，三个sem一个lock解决）

   ```
   semaphore empty = 3, apple = 0, orange = 0, mutex = 1;
   void father() {
   	whlie(1) {
   	    P(empty)
           P(mutex);
           putApple();
           V(mutex);
           V(apple);
   	}
   }
   
   
   void son() {
   	while(1) {
   		P(apple);
   		P(mutex);
   		eatApple();
   		V(mutex);
   		V(empty);
   	}
   }
   ```

   

代码示例

![image-20250106142417478](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250106142417478.png)

![image-20250106142447688](C:\Users\ASUS\AppData\Roaming\Typora\typora-user-images\image-20250106142447688.png)



### 4 死锁相关

1. 四个必要条件(互斥、不可抢占、持有并等待、循环等待)

2. [银行家算法](https://blog.csdn.net/DreamWendy/article/details/118077621)，例题 解法：[【1】](https://www.bilibili.com/video/BV1qd4y177eA/?spm_id_from=333.788.videopod.episodes&vd_source=286ca0546d1a508d3fb7c6862b91dafc&p=6)[【2】](https://www.bilibili.com/video/BV1rJ411p7au/?spm_id_from=333.337.search-card.all.click&vd_source=286ca0546d1a508d3fb7c6862b91dafc)

   > 例题：
   >
   > ![image-20250105135405649](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250105135405649.png)
   >
   > - 第二问：**判断是否处于安全状态**：是否可以找到一个安全进程序列<*p*0,*p*3，*p*4，*p*1,*p*2>，它使Finish[i]=true，因而可以断言系统当前是否处于安全状态．
   >
   >   解题步骤：列表，【work(初始值Available)， Allocation、Need、work+Allocation(作为下一行的work), T/F(如果Need<=Available则为True)】
   >
   >   ![image-20250106140904165](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250106140904165.png)
   >
   > - 第三问：
   >
   >   - 判断Request[i]<=Need[i]且Request[i]<Available
   >   - 更新第二问的表，重新算一遍银行家算法

### 5 内存管理

1. 虚拟地址转物理地址

   ![image-20250104152356535](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250104152356535.png)

   **知识点：**

   - 页表存放：页号 + 页框号

   - 页表项数量 = 逻辑页数

   - 逻辑页大小(页) = 物理页大小（页框） = 2^(页内偏移位数)

   - 查询一个逻辑页：

     若**TLB 命中**，直接获取物理地址并访问内存一次

     若**TLB 未命中**（15%概率）：需要访问内存两次：第一次查找页表以获取物理地址、第二次根据物理地址访问内存。

   - 页表项内容：

     ![image-20250105204128085](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250105204128085.png)

   > 例题：
   >
   > - 给定页表，逻辑地址 -> 物理地址
   >
   > ![image-20250104152608778](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250104152608778.png)
   >
   > - 通过逻辑地址找到页目录号、页号（做题逻辑相同）
   >
   >   ![img](https://cdn.xiaolincoding.com//mysql/other/19296e249b2240c29f9c52be70f611d5.png)
   >
   >   - 物理地址=物理页（页框号）号×页大小+页内偏移
   >   - 计算相关：[链接5-3](https://blog.csdn.net/qq_43582207/article/details/130849341)
   >   - 二级页表：https://noobdream.com/Practice/article/5649/
   >   - 求页框物理地址：https://noobdream.com/Practice/article/5680/

   

2. 计算访问时间（[访问TLB、内存、缺页中断](https://blog.csdn.net/DreamWendy/article/details/118077621)[**第十题！！！]**）

3. 页面置换算法

   > 例题：
   >
   > ![image-20250105153909057](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250105153909057.png)

4. 段式存储：链接第15题https://blog.csdn.net/s1ms1mpleple/article/details/139439591

### 6 磁盘相关

1. 了解基本概念（盘片、磁头、磁道、扇区、柱面），计算寻道时间([例题-4-2](https://blog.csdn.net/qq_43582207/article/details/130849341))

2. 几个常见的磁盘调度算法：

   - FCFS、最短寻道时间优先、电梯（SCAN）、循环扫描（C-SCAN）<u>（看很多例题SCAN和C-SCAN都没有移到磁盘末端，当作是LOOK和C-LOOK算法算，但是书上是需要移动到末端的）</u>

   > 例题：
   >
   > 【1】：
   >
   > ![image-20250105155745985](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250105155745985.png)
   >
   > 【2】：
   >
   > ![image-20250105144552387](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250105144552387.png)

3. 磁盘分区(动态分区分配算法)

   - 首次适应：按分区编号遍历
   
   - 最佳适应：按分区大小，从小到大遍历
   
     ![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250107154114726.png)

### 7 文件系统

1. 文件存储方式(磁盘块分配方式)

   1. 连续

   2. 非连续：链表方式、索引方式

      FAT表：FAT（File Allocation Table）表记录了每个块号对应的下一个块号，文件存储在这些块号的链表中。如果某块号对应的 FAT 表值为 `-1`，表示该块号是文件的最后一块。

      > 例题：
      >
      > - 计算FAT表(链表存储方式)
      >
      > ![image-20250104153537445](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250104153537445.png)
      >
      > - https://noobdream.com/Practice/article/5643/
      
      

2. 内存空间管理：空闲表法、空闲链表法、位图法

   



大题讲解：

- https://www.bilibili.com/video/BV1qd4y177eA/?spm_id_from=333.337.search-card.all.click&vd_source=286ca0546d1a508d3fb7c6862b91dafc
- https://blog.csdn.net/qq_43582207/article/details/130849341
- https://blog.csdn.net/s1ms1mpleple/article/details/139439591



# 408 OS大题

1. 三人植树：https://noobdream.com/Practice/article/5767/

```
semaphore Dig = 0, tieqiao = 1, toJiaoshui = 0, canDig = 3;


void jia() {
	wait(canDig);
	wait(tieqiao);
	
	work();
	
	signal(tieqiao);
	signal(Dig);
}


void yi() {
	wait(Dig);
	
	seed();
	
	wait(tieqiao);
	
	tiantu();
	
	signal(tieqiao);
	
	signal(canDig);
	
	signal(toJiaoshui);
	
	
}

void bin() {
	wait(toJiaoshui);
	
	jiaoshui();
}
```

2. 

```
semaphore wind = 0, site = 10, cli_wait = 0;

cobegin

process 顾客i {
	wait(site);
	signal(cli_wait);
	wait(wind);
	signal(site);
  	getserver();
  	
  	

}

process 营业员 {

    while (TRUE) {
    	wait(cli_wait);
		signal(wind);
        server();

    }

}

coend
```

