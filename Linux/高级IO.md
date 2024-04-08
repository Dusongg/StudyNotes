# 1 非阻塞IO

- 使用`fcntl`函数，设置文件在底层的flag标志位

![image-20240326105638140](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240326105638140.png)

- cmd参数：

![image-20240326105701349](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240326105701349.png)

- 实现非阻塞

```cpp
void setNoBlock(int fd) {
    int f1 = fcntl(fd, F_GETFL);
    /*if (f1 < 0)   ...*/
    fcntl(fd, F_SETFL, f1 | O_NONBLOCK)
}
```

- 非阻塞读**返回值小于0**， 错误码被设置`errno == EWOULDBLOCK`
  - `#define EWOULDBLOCK EAGAIN`
  - `#define EAGAIN 11   /*Try again*/`



#  2 select

![image-20240326114829036](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240326114829036.png)

- nfds：等待的文件描述符，`nfds = maxfd + 1`
- timeout：设置等待时间

![image-20240326115130479](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240326115130479.png)

- fd_set：读/写/异常文件描述符位图，输入输出型参数

![image-20240326130722202](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240326130722202.png)



## 缺点

1. 等待的fd有上限 ： sizeof(fd_set)  * 8 = 1024
2. 数据拷贝频率高
3. 内核中也要遍历fd

# 3 poll



# 4 epoll



![image-20240330231640551](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240330231640551.png)

![image-20240330225942531](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240330225942531.png)

- epoll_creat()
- epoll_ctl()

![image-20240408180735321](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240408180735321.png)

- epoll_wait()

![image-20240408180756883](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240408180756883.png)

## 4.1 ET / LT

- ET通知一次需要上层将数据全部取走，则需要循环读取，直到读取出错，所以需要将fd设置成**<u>非阻塞</u>**的

- ET的通知小路更高，且IO效率更高，因为此时接收缓冲区的空余空间更大，tcp接收窗口更大，对方能一次性发送的数据也就更多
- 
