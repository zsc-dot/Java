# 1、Nacos配置管理

Nacos除了可以做注册中心，同样可以做配置管理来使用。



## 1.1、统一配置管理

当微服务部署的实例越来越多，达到数十、数百时，逐个修改微服务配置就会让人抓狂，而且很容易出错。

这个时候就需要一种统一配置管理方案，可以集中管理所有实例的配置。

**配置更改热更新**：完成配置文件改动后，微服务不需要重启，这些配置就能立马生效。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814194648349.png" alt="image-20220814194648349" style="zoom:67%;" />

Nacos一方面可以将配置集中管理，另一方可以在配置变更时，及时通知微服务，实现配置的热更新。

<img src="https://raw.githubusercontent.com/zsc-dot/pic/master/img/Git/image-20220814194952564.png" alt="image-20220814194952564" style="zoom:67%;" />