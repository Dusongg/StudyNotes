# 1 Docker的安装

**https://docs.docker.com/engine/install/ubuntu/**

![image-20240511135034985](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511135034985.png)

# 2 常用命令

![image-20240510222738923](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240510222738923.png)

1. `docker run`

```bash
docker run -d --name nginx -p 80:80 [-e 环境变量] [-v 宿主文件:挂载文件路径] nginx
```

2. `docker exec`

```bash
#以bash命令行的方式进入容器内部
#-it: interface terminal
docker exec -it nginx bash
#退出
exit
```

3. 解决`docker ps --format`命令复杂问题

```bash
#编辑.bashrc
vim ~/.bashrc

#取别名
alias dps='docker ps --format "table {{.ID}\t{{.Image}}\t{{.Port}}\t{{.Status}}\t{{.Names}}}"'

#执行
source ~/.bashrc
```



# 3 数据卷挂载

- 如何在容器内修改文件？ ---> 数据卷`docker volume`

![image-20240510225547785](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240510225547785.png)

| 命令                    | 说明                     |
| ----------------------- | ------------------------ |
| `docker volume create`  | 创建数据卷               |
| `docker volume ls`      | 查看所有数据卷           |
| `docker volume rm`      | 删除指定数据卷           |
| `docker volume inspect` | 查看某个数据卷的详细信息 |
| `docker volume prune`   | 清除数据卷               |

## 如何挂载数据卷？

- 在创建容器时，用`-v 数据卷名：容器内目录`
- 容器创建时，如果数据卷不存在，自动创建（执行`docker colume create`）

`docker run -d --name nginx -p 80:80 -v html:/usr/share/nginx/html nginx`

## 本地目录挂载

![image-20240511134833376](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511134833376.png)

# 4 Dockerfile —— 自定义镜像

- 镜像结构

![image-20240511140400860](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511140400860.png)

- dockerfile指令

![image-20240511140425319](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511140425319.png)

- 构建镜像

`docker build -t 镜像名 Dockerfile的目录`



# 5 Docker网络

- 默认情况下通过网桥链接，ip自动分配
- `ip addr`查看

![image-20240511142108739](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511142108739.png)

- 自定义网络 ：加入自定义网络的容器可以通过容器名互相访问

![image-20240511142337816](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511142337816.png)

![image-20240511155044792](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511155044792.png)



# 6 DockerCompose

![image-20240511163231272](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511163231272.png)

![image-20240511164003893](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240511164003893.png)