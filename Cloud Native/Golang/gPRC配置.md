# 1 安装依赖包

```cmd
go get google.golang.org/grpc
```



# 2 安装Protobuf ---- [protoc-21.5-win64.zip](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fprotocolbuffers%2Fprotobuf%2Freleases%2Fdownload%2Fv21.5%2Fprotoc-21.5-win64.zip)

安装解压，打开bin目录，复制路径添加到环境变量中

![image-20240716101337125](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240716101337125.png)

# 3 安装protoc-gen-go和protoc-gen-go-grpc

- 确保GOBIN在PATH中

  ![image-20240716101559865](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240716101559865.png)

```cmd
go install google.golang.org/protobuf/cmd/protoc-gen-go
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc
```



## 生成gPRC Code

```cmd
protoc --go_out=.  --go-grpc_out=. ./*.proto
```

![image-20240716101822982](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20240716101822982.png)