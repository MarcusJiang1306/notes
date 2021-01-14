## 初识soul网关

#### soul简介
soul是一个异步的, 高性能的, 跨语言的, 响应式的API网关, 作者在参考了Kong, Spring-Cloud-Gateway等优秀的网关后, 站在巨人的肩膀上, Soul由此诞生!  

#### soul核心Features
* 支持各种语言(http协议), 支持 dubbo, springcloud协议.
* 插件化设计思想, 插件热插拔, 易扩展.
* 灵活的流量筛选, 能满足各种流量控制.
* 内置丰富的插件支持, 鉴权, 限流, 熔断, 防火墙等等.
* 流量配置动态化, 性能极高, 网关消耗在 1~2ms.
* 支持集群部署, 支持 A/B Test, 蓝绿发布

#### 架构图
![Image text](https://camo.githubusercontent.com/2be38565b7e097e61fdcae39f77c5066c820e307e9a1d03743d7f8dda2d6b410/68747470733a2f2f79753139393139352e6769746875622e696f2f696d616765732f736f756c2f736f756c2d6672616d65776f726b2e706e67)
