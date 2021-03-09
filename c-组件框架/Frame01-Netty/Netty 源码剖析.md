# Nettty 源码剖析

## 服务端启动流程

> 两个问题：
>
> * 服务端的 Socket 在哪里初始化
> * 在哪里accept ?

服务端启动流程

1. 创建服务端 Channel
2. 初始化服务端Channel
3. 注册 Selectot
4. 端口绑定

### 创建服务端 Channel

```java
b.bind(8888)
```

