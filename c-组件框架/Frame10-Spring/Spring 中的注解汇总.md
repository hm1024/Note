[TOC]
# Spring 中的注解汇总

## IOC&DI
### **@Bean**

**作用**：注解在方法上，声明当前方法的返回值为一个Bean。返回的Bean对应的类中可以定义init()方法和destroy()方法，然后在@Bean(initMethod=“init”,destroyMethod=“destroy”)定义，在构造之后执行init，在销毁之前执行destroy。

### **@Component**

表示一个带注释的类是一个“组件”，成为Spring管理的Bean。当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。同时@Component还是一个元注解。

### **@Controller**

组合注解（组合了@Component注解），应用在MVC层（控制层）,DispatcherServlet会自动扫描注解了此注解的类，然后将web请求映射到注解了@RequestMapping的方法上。

### **@Service**

组合注解（组合了@Component注解），应用在service层（业务逻辑层）

### **@Reponsitory**

组合注解（组合了@Component注解），应用在dao层（数据访问层）

### **@Autowired**

Spring提供的工具自动注入。**默认按类型装配**（这个注解是属业spring的），默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false，如：@Autowired(required=false) ，如果我们想使用名称装配可以结合@Qualifier注解进行使用，如下：

```java
@Autowired
private Student student;// 此时先按类型匹配，在按明名称匹配。student必须存在
@Autowired(required = false)
private Student student;//此时先按类型匹配，在按明名称匹配。student可以不存在，student为null
@Autowired
@Qualifier("student")
private Student student;// 此时先按名称匹配，如果不存在则报错，
```

### @Resources

JSR-250提供的注解,**是JDK1.6支持的注解**，**默认按照名称进行装配**，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找，如果注解写在setter方法上默认取属性名进行装配。当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。

```java
@Resource(name="student")    
private Student student;// 先按名称注入，如果按名称找不到，在按类型注入
```

注意：如果没有指定name属性，并且按照默认的名称仍然找不到依赖的对象时候，会回退到按照类型装配，但一旦指定了name属性，就只能按照名称 装配了.

## AOP 相关注解

### @Aspect 

声明一个切面（就是说这是一个额外功能）



##  配置相关注解

### @Configuration

声明当前类是一个配置类（相当于一个Spring配置的xml文件）

### @ComponentScan

默认会扫描该类所在的包下所有的配置类，相当于之前的` <context:component-scan>`，也可以指定要扫描的包。

### @EnableAspectJAutoProxy

开启 @AspectJ 的注解配置方式，相当于`<aop:aspectj-autoproxy/>`