# Spring Framework

Spring 的宗旨：

+ 简化操作
+ 降低耦合

spring 提供的方案：Spring帮助创建对象，将原本写死的代码，抽取出来，在运行时动态的加入到运行的代码中。



### IOC 

1. 可以帮我们创建和管理 Bean 的生命周期
2. 可以

在典型的IOC场景中，容器创建了所有对象，并设置必要的属性将它们连接在一起，



## AOP

AOP 使用场景：

日志系统，安全统一校验

AOP 的优点

集中处理某一类问题，方便维护。逻辑更加清晰。降低模块间的耦合度。





获取spring的IOC核心容器，并根据ID获取对象

ApplicationContext 的三个常用实现类
- ClassPathXmlApplicationContext:它可以加载类路径下的配置文件，要求配置文件必须在类路径下，不在的话加载不了
- FileSystemXmlApplicationContext：它可以加载磁盘任意路径下的配置文件（必须有访问权限）
- AnnotationfigApplicationContext：它是用于读取注解创建容器的

