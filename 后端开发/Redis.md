# 1 初始工作

- 安装：`sudo apt install redis`

- 配置文件

  ```
  bind 0.0.0.0 ::1    #将本地环回改为任意ip
  
  protected-mode no   #开启后其他主机才能访问
  ```

  配置完后重启服务器：`service redis-sever restart`

  查看运行状态：`service redis-server status`

  ![](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240421202357236.png)

- 链接

  ```bash
  redis-cli
  ```

- 退出：`quit` 或 `ctrl + d`