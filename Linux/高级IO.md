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

- fd_set：读/写/异常文件描述符位图

![image-20240326130722202](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240326130722202.png)