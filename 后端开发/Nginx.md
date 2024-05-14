# 1 安装

1. 解压`tar zxvf nginx-1.21.6.tar.gz`
2. 安装依赖
   1. `yum install -y gcc`
   2. `yum install -y pcre pcre-devel`
   3. `yum install -y zlib zlib-devel`

3. ```bash
   ./configure --preifx=/usr/local/nginx
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

5. 关闭防火墙：`systemctl stop firewalld.service`

   禁止防火墙开机启动：`systemctl disable firewalld.serviece`

   重启防火墙：`firewall-cmd --reload`

6. 将`ngnix`安装成系统服务

   1. 创建服务脚本

      `vim /usr/lib/systemd/system/nginx.service`

   2. 写入内容

      ![image-20240514235918175](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240514235918175.png)

   3. 重新加载系统服务：`systemctl daemon-reload`

   4. 启动服务：`systemctl start nginx.service`

   5. 开机启动：`systemctl enable firewalld.service`

## 使用Docker安装

```bash
docker run -d --name nginx -p 80:80 [-e 环境变量] [-v 宿主文件:挂载文件路径] nginx
```

