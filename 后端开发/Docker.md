# 1 Docker的安装

**https://docs.docker.com/engine/install/ubuntu/**

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