# Spring 官方文档笔记---Bean
> In Spring, the objects that form the backbone of your application and that are managed by the Spring IoC *container* are called *beans*.

在Spring中，构成应用程序的主干并由 Spring IOC 容器管理的对象称为 bean。`bean计算机中也称可重用组件`
bean是一个由Spring IoC容器实例化，组装和管理的对象。否则，bean只是应用程序中许多对象之一。Bean及其之间的依赖关系反映在容器使用的配置元数据中。

Spring 容器中的 bean 定义对应构成应用程序的实际对象。

bean 标签中的`ref`元素指的是另一个bean定义的名称。元素`id`和`ref`元素之间的联系表达了协作对象之间的依赖关系.

让bean定义跨越多个XML文件。通常，每个单独的XML配置文件都代表架构中的逻辑层或模块。通过`<import/>`元素从从另一个或多个文件加载bean定义。

```xml
<beans>
    <import resource="services.xml"/>
    <import resource="resources/messageSource.xml"/>

    <bean id="bean1" class="..."/>
    <bean id="bean2" class="..."/>
</beans>
```



## 命名bean
> <bean id = "person" name="person" class="com.minghai.Person" ></bean>

在基于XML的配置元数据中，使用`id`和/或`name`属性指定bean标识符。`id`属性允许指定一个id，在`name`属性中可以引入其他别名，用`,`、`;`或者`空格`分割。另外还可以在用`alias`元素引入别名。

> ```
> <alias name="fromName" alias="toName"/>
> ```

## 实例化Bean

Bean 实质是用于创建一个或多个对象的配方。

