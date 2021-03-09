### Consul Agent

Consul Agent是Consul核心进程，这个agent负责节点管理，注册发现，健康管理，响应检查等一系列功能，是Consul cluster都要运行的进程



服务端集群是基于选举模式的



agent有两个模式可选 客户端和服务端的模式  服务端要负责选举 同步 保证强一致 比较吃资源 建议独立部署 客户端就个是工具人连服务端就好了 所以很轻巧



启动的时候有些关键字

node name 唯一名字，一般是hostname 用-node改

Datacenter -datacenter配置 默认dc1 也支持多dc

Server 上面讲了能客户端模式或者服务端模式 服务端开销较大 负责选举 服务端还有个bootstrap模式 在一个集群里面只能有一个bootstrap模式 不然会导致不一致



Client Addr 客户端访问agent的方法 可以是HTTP模式和dns模式 通常绑定在localhost -http-addr改绑定 



Cluster Addr 服务端之间的连接通道

