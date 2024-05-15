# 1 安装

1. 解压`tar zxvf nginx-1.21.6.tar.gz`

2. 安装依赖
   1. `yum install -y gcc`
   2. `yum install -y pcre pcre-devel`
   3. `yum install -y zlib zlib-devel`

3. ```bash
   ./configure --prefix=/usr/local/nginx
   make
   make install
   ```

4. 启动

   ```bash
   cd /usr/local/nginx/sbin
   ./ngnix
   ./nginx -s stop
   ./ngnix -s quit
   ./ngnix -s reload
   ```

   ![image-20240515185253951](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240515185253951.png)

5. 关闭防火墙：`systemctl stop firewalld.service`

   禁止防火墙开机启动：`systemctl disable firewalld.serviece`

   ![image-20240515185234709](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240515185234709.png)

   重启防火墙：`firewall-cmd --reload`

6. 将`ngnix`安装成系统服务

   1. 创建服务脚本

      `vim /usr/lib/systemd/system/nginx.service`

   2. 写入内容

      ```
      [Unit]
      Description=nginx - web server
      After=network.target remote-fs.target nss-lookup.target
      [Unit]
      Description=nginx - web server
      After=network.target remote-fs.target nss-lookup.target
      
      [Service]
      Type=forking
      PIDFile=/usr/local/nginx/logs/nginx.pid
      ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
      ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
      ExecReload=/usr/local/nginx/sbin/nginx -s reload
      ExecStop=/usr/local/nginx/sbin/nginx -s stop
      ExecQuit=/usr/local/nginx/sbin/nginx -s quit
      PrivateTmp=true
      
      [Install]
      WantedBy=multi-user.target
      ```

   3. 重新加载系统服务：`systemctl daemon-reload`

   4. 启动服务：`systemctl start nginx.service`

   5. 开机启动：`systemctl enable nginx.service`

## 使用Docker安装

```bash
docker run -d --name nginx -p 80:80 [-e 环境变量] [-v 宿主文件:挂载文件路径] nginx
```

 

# 2 nginx基本原理

![image-20240515193618986](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240515193618986.png)

- master主进程`pid:3983`（在`nginx.pid`中可以配置），master通过fork出子进程：worker进程`pid:3985`

![image-20240515193918849](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240515193918849.png)

# 3 初时nginx.conf

- **[sendfile](https://xiaolincoding.com/os/8_network_system/zero_copy.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E8%A6%81%E6%9C%89-dma-%E6%8A%80%E6%9C%AF)**
- `location`:匹配uri

```nginx
worker_processes  1; #允许进程数量，建议设置为cpu核心数或者auto自动检测，注意Windows服务器上虽然可以启动多个processes，但是实际只会用其中一个

events {
    #单个进程最大连接数（最大连接数=连接数*进程数）
    #根据硬件调整，和前面工作进程配合起来用，尽量大，但是别把cpu跑到100%就行。
    worker_connections  1024;
}


http {
    #文件扩展名与文件类型映射表(是conf目录下的一个文件)
    include       mime.types;
    #默认文件类型，如果mime.types预先定义的类型没匹配上，默认使用二进制流的方式传输
    default_type  application/octet-stream;

    #sendfile指令指定nginx是否调用sendfile 函数（zero copy 方式）来输出文件，对于普通应用，必须设为on。如果用来进行下载等应用磁盘IO重负载应用，可设置为off，以平衡磁盘与网络IO处理速度。
    sendfile        on;
    
     #长连接超时时间，单位是秒
    keepalive_timeout  65;

 #虚拟主机的配置 
    server {
    #监听端口
        listen       80;
        #域名，可以有多个，用空格隔开
        server_name  localhost;

	#配置根目录以及默认页面
        location / {    
            root   html;  #/usr/local/nginx/html
            index  index.html index.htm;
        }

	#出错页面配置
        error_page   500 502 503 504  /50x.html;
        #/50x.html文件所在位置
        location = /50x.html {
            root   html;
        }
    }

}

```

#  4 虚拟主机与域名解析

## 4.1 配置域名

![image-20240515233859071](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240515233859071.png)

