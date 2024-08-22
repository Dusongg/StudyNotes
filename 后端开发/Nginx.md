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

# 3 初识nginx.conf

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
            index  index.html index.html;
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

```nginx
server {
    #监听端口
        listen       80;
        #域名，可以有多个，用空格隔开
        server_name  localhost;

	#配置根目录以及默认页面
        location / {    
            root   /www/www;  #/usr/local/nginx/html;
            index  index.html index.html;
        }

	#出错页面配置
        error_page   500 502 503 504  /50x.html;
        #/50x.html文件所在位置
        location = /50x.html {
            root   html;
        }
    }

server {
    #监听端口
        listen       80;
        #域名，可以有多个，用空格隔开
        server_name  dusong.com;

	#配置根目录以及默认页面
        location / {    
            root   /www/gif;  #/usr/local/nginx/html;
            index  index.html index.html;
        }

	#出错页面配置
        error_page   500 502 503 504  /50x.html;
        #/50x.html文件所在位置
        location = /50x.html {
            root   html;
        }
    }

```

## 4.2 ServerName匹配规则

- `server_name`配置：
  - `*.dusong.com`
  - `dusong.*`
  - 正则：`~^[0-9]+\.dusong.\com$`

# 5 基本使用

## 5.1proxy_pass

```nginx
server {
    #监听端口
        listen       80;
        #域名，可以有多个，用空格隔开
        server_name  localhost;

	#配置根目录以及默认页面
        location / {    
        	proxy_pass qq.com;
            #root   /www/gif;  #/usr/local/nginx/html;
            #index  index.html index.html;
        }

	#出错页面配置
        error_page   500 502 503 504  /50x.html;
        #/50x.html文件所在位置
        location = /50x.html {
            root   html;
        }
    }
```

![image-20240516160627201](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240516160627201.png)

## 5.2 负载均衡

```nginx
upstream httpsd {
    server 192.168.17.132:80;   #默认80端口，可以不写
    server 192.168.17.133:80;
}

server {
    #监听端口
        listen       80;
        #域名，可以有多个，用空格隔开
        server_name  localhost;

	#配置根目录以及默认页面
        location / {    
        	proxy_pass http://httpsd;
            #root   /www/gif;  #/usr/local/nginx/html;
            #index  index.html index.html;
        }

	#出错页面配置
        error_page   500 502 503 504  /50x.html;
        #/50x.html文件所在位置
        location = /50x.html {
            root   html;
        }
    }
```

## 5.3 负载均衡策略

### 5.3.1 轮询

默认情况下使用轮询方式，逐一转发，这种方式适用于无状态请求。

### 5.3.2 权重

1. `weight`：权重 

```nginx
upstream httpds {
    server 192.168.17.132 weight=8;
    server 192.168.17.133 weight=4;
    server 192.168.17.134 weight=2;
}
```

2. `down`  和  `backup`：

```nginx
upstream httpds {
    server 192.168.17.132 weight=8 down;  #停用
    server 192.168.17.133 weight=4 backup;	#备用机
    server 192.168.17.134 weight=2;
}
```

### 5.3.3 ip_hash、least_conn、url_hash、fair



### 5.3.4 cookie/session/token

 

## 5.4 Nginx动静分离配置

```nginx
location ~*/(js|img|css) {   #匹配当前本机nginx的js/img/css静态文件 
    root html;
    index index.html index.html;
}
```

## 5.5 urlrewrite

![image-20240516201703214](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240516201703214.png)

```nginx
server {
    
    location / {
        rewrite ^/([0-9]+).html$		/index.jsp?pageNum=$1    break;
    }
    
}
```

### 5.5.1 应用服务器防火墙配置

![image-20240516203452903](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240516203452903.png)

### 5.5.2 nginx作为网关服务器

- 实现动静分离、urlrewrite、proxy_pass、负载均衡

> 什么是网关，nginx可以作为网关嘛？
>
> 
>
> 网关（Gateway）是一个网络设备或服务，用于连接不同网络并处理网络之间的通信。网关可以实现网络协议转换、数据包过滤、流量控制等功能，常用于在不同网络拓扑和协议之间进行数据交换。
>
> Nginx 是一个高性能的 HTTP 和反向代理服务器，也是一个邮件代理服务器（IMAP/POP3）。它常用于：
>
> 1. **反向代理**：Nginx 可以作为反向代理服务器，将客户端请求转发给后端服务器，并返回后端服务器的响应给客户端。这样可以实现负载均衡、缓存和安全等功能。
> 2. **负载均衡**：Nginx 可以将请求分发到多个后端服务器上，以实现负载均衡，分散服务器压力，提高可用性。
> 3. **静态文件服务**：Nginx 可以高效地处理静态文件请求，如 HTML、CSS、JavaScript、图片等。
> 4. **HTTP/HTTPS 服务器**：Nginx 支持 HTTP 和 HTTPS 协议，可以配置 SSL/TLS 证书实现安全通信。
>
> 因此，Nginx 可以作为网关的一种实现方式，特别是在 HTTP 和反向代理场景中。它可以处理客户端请求，转发给不同的后端服务，并进行负载均衡和缓存，从而提高系统的性能和可靠性。Nginx 还可以作为 API 网关，处理 API 请求，并进行路由、认证和限流等操作。

## 5.6 防盗链配置

![image-20240516210451976](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240516210451976.png)

### 5.6.1 使用curl测试

![image-20240516210329234](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240516210329234.png)

### 5.6.1 配合rewrite返回报错图片

```nginx
valid_referers none 192.168.44.101;
if($invalid_referer) {
    rewrite ^/ /img/x.png break     #返回指定目录的一个图片
}
```



## 5.7 高可用配置

- keepalived通过判断keepalived进程是否在运行推断服务器是否宕机，进而切换到备用机

![image-20240523163310565](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523163310565.png)

### 5.7.1 Keepalived

1. 安装：`yum install -y keepalived`

2. 修改配置：`vim /etc/keepalived/keepalived.conf`

   1. 主机配置

      ![image-20240523163022243](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523163022243.png)

   2. 从机配置

      ![image-20240523162845555](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523162845555.png)

3. 启动：`systemctl start keepalived`



# 6 线上实战

1. 购买域名

2. 购买vps

3. 安装LNMP（在oneinstack.com上配置安装）

   ![image-20240523172014667](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523172014667.png)

4. 解析域名到主机

   ![image-20240523171936847](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523171936847.png)

5. 在线申请证书

   ![image-20240523172233603](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523172233603.png)

6. 将证书配置到服务器上

   1. ![image-20240523172551393](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523172551393.png)

   2. ![image-20240523172809225](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523172809225.png)

   3. 配置到nginx

      ![image-20240523173115905](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523173115905.png)

7. 安装Discuz

![image-20240523175236917](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240523175236917.png)



# 7 Ningx高级
