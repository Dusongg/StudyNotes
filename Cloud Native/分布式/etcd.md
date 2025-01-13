# 架构

**etcd** 是一个高可用的分布式键值存储系统，主要用于分布式系统中的配置管理、服务发现和分布式锁等场景。它是由 [CoreOS](https://coreos.com/) 开发的，基于 Raft 共识算法保证数据的强一致性。

![image-20250111下午43423923](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250111%E4%B8%8B%E5%8D%8843423923.png)

其中黑框内的为一个raft集群节点

# grpc中使用etcd作为服务的注册与发现

![image-20250112下午63915090](https://typora-dusong.oss-cn-chengdu.aliyuncs.com/image-20250112%E4%B8%8B%E5%8D%8863915090.png)

## 服务注册

registerEndPointToEtcd 方法给出了将 grpc 服务端节点注册到 etcd 模块的示例：

-  eclient.NewFromURL 创建 etcd 客户端 etcdClient
- endpoints.NewManager 创建 etcd 服务端节点管理模块 etcdManager
- etcdClient.Grant 申请一份租约，租约的有效时间为 ttl
-  etcdManager.AddEndpoint 将当前节点注册到 etcd 中，同时会和租约进行关联
- etcdClient.KeepAliveOnce 对租约进行一轮续期，重置租约失效的 ttl

```go
func registerEndPointToEtcd(ctx context.Context, addr string) {
    // 创建 etcd 客户端
    etcdClient, _ := eclient.NewFromURL(MyEtcdURL)
    etcdManager, _ := endpoints.NewManager(etcdClient, MyService)


    // 创建一个租约，每隔 10s 需要向 etcd 汇报一次心跳，证明当前节点仍然存活
    var ttl int64 = 10
    lease, _ := etcdClient.Grant(ctx, ttl)
    
    // 添加注册节点到 etcd 中，并且携带上租约 id
    _ = etcdManager.AddEndpoint(ctx, fmt.Sprintf("%s/%s", MyService, addr), endpoints.Endpoint{Addr: addr}, eclient.WithLease(lease.ID))


    // 每隔 5 s进行一次延续租约的动作
    for {
        select {
        case <-time.After(5 * time.Second):
            // 续约操作
            resp, _ := etcdClient.KeepAliveOnce(ctx, lease.ID)
            fmt.Printf("keep alive resp: %+v", resp)
        case <-ctx.Done():
            return
        }
    }
}
```





# watch机制

1. 概念

**Watch** 是 etcd 的一种长连接机制，基于 gRPC 流协议实现， 客户端通过 Watch 接口订阅键或键的范围，当这些键发生更改（如创建、修改或删除）时，etcd 会将事件通知客户端。

2. 用途

**分布式锁**：通过监听锁的释放事件，实现高效的分布式锁管理。

**配置热更新**：应用程序可以监听配置键的变化，实时加载最新的配置。

**服务发现**：服务可以监听特定路径下的键变更，实现服务注册与发现。

3. 原理

服务端为每个 Watch 请求创建一个 Watch 实例，绑定到特定键或键范围。

服务端在内存中维护 Watcher 列表，并将这些 Watcher 挂载到相关键或范围的事件触发器上。