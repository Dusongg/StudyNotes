

# 基础

- goctl安装
  - `go install github.com/zeromicro/go-zero/tools/goctl@latest`
  - `export PATH=\$PATH:\$(go env GOPATH)/bin`
- protoc安装
  - `brew install protobuf`
  - `go install google.golang.org/protobuf/cmd/protoc-gen-go@latest` 生成go代码
  - `go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest` 生成gRPC代码



1. `go api new [name]`:生成api





## 目录说明

![image-20250117下午20531847](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250117%E4%B8%8B%E5%8D%8820531847.png)



# 处理跨域

## 什么是跨域？

- **跨域（Cross-Origin）**是指一种浏览器的安全策略限制，<u>涉及从一个源（域名、协议或端口）对另一个源的资源进行请求时的行为</u>。为了保护用户数据和防止恶意攻击（如跨站请求伪造 CSRF 和跨站脚本 XSS），浏览器会对跨域请求施加一定的限制。
- **同源策略（Same-Origin Policy）**是浏览器的核心安全机制，它要求<u>协议、域名、端口号</u>均相同

<img src="https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250118%E4%B8%8B%E5%8D%8813256686.png" alt="image-20250118下午13256686" style="zoom:50%;" />

## go-zero跨域配置

1. 

```go
server := rest.MustNewServer(c.RestConf,
		rest.WithCors("http://localhost:5173"))
```

2. 控制跨域域名与自定义header

```go
server := rest.MustNewServer(c.RestConf,
        //顺序不能颠倒
		rest.WithCors("http://localhost:5173"),
		rest.WithCorsHeaders("xxxx"),
	)
```

3. 自定义跨域

```go
server := rest.MustNewServer(c.RestConf,
		rest.WithCustomCors(func(header http.Header) {
			var allowOrigin = "Access-Control-Allow-Origin"
			var allOrigins = "http://localhost:5173"
			var allowMethods = "Access-Control-Allow-Methods"
			var allowHeaders = "Access-Control-Allow-Headers"
			var exposeHeaders = "Access-Control-Expose-Headers"
			var methods = "GET, HEAD, POST, PATCH, PUT, DELETE, OPTIONS"
			var allowHeadersVal = "xxxx, Content-Type, Origin, X-CSRF-Token, Authorization, AccessToken, Token, Range"
			var exposeHeadersVal = "Content-Length, Access-Control-Allow-Origin, Access-Control-Allow-Headers"
			var maxAgeHeader = "Access-Control-Max-Age"
			var maxAgeHeaderVal = "86400"
			header.Set(allowOrigin, allOrigins)
			header.Set(allowMethods, methods)
			header.Set(allowHeaders, allowHeadersVal)
			header.Set(exposeHeaders, exposeHeadersVal)
			header.Set(maxAgeHeader, maxAgeHeaderVal)
		}, func(w http.ResponseWriter) {

		}),
	)
```



# 中间件

## 鉴权

### 什么是

### JWT，他和cookie、session的区别

- JWT：

**存储位置**：通常存储在客户端（Cookie 或 LocalStorage）。

**传输方式**：通过 HTTP Header（如 Authorization）传递，也可用作 URL 参数。

**存储内容**：以 JSON 格式存储信息，包括：

​	**Header**：元数据（如签名算法）。

​	**Payload**：用户数据（如用户 ID、角色等）。

​	**Signature**：基于 Header 和 Payload 的签名，用于校验数据完整性。

**持久性**：由客户端决定（如存储在 LocalStorage、SessionStorage 或 Cookie 中）。

### 服务端怎么验证JWT的合法性

## 熔断

**熔断**（Circuit Breaker）是计算机系统中的一种保护机制，主要用于在分布式系统或微服务架构中**防止故障传播**并提高系统的稳定性和可靠性。

go-zero 提供了内置的熔断功能，基于以下策略：

​	• **错误率阈值**：当下游服务的错误率超过一定比例时，熔断器开启。

​	• **请求时间阈值**：下游服务响应超时时，计入失败统计。

​	• **冷却时间**：熔断器打开后，系统进入冷却阶段，拒绝新请求。

​	• **半开状态**：冷却时间过后，允许少量请求通过，检测下游服务恢复情况。

​	• **自动恢复**：如果下游服务恢复正常，熔断器会关闭，重新允许所有请求。





# gRPC getway









