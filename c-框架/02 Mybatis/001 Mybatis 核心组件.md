#  Mybatis 核心组件

**Mybatis 的优点**

优点

+ 相比于 JDBC 需要编写的代码更少
+ 使用灵活，支持动态 SQL
+ 提供映射标签，支持对象与数据库的字段关系映射

缺点

+ SQL 语句依赖于数据库，数据库移植性差
+ SQL 语句编写工作量大，尤其在表、字段比较多地情况下

Mybatis 是一个非常优秀和灵活的 ORM 数据库持久化框架。

## Mybatis 重要组件

Mybatis 中的重要组件如下：

+ Mybatis-config 配置：Mybatsi 全局配置文件，定义数据源，设置 Mybatis 的基本参数。

+ Mapper 配置：用于组织具体的业务查询和映射数据库字段关系，可以使用 XML 格式或 Java 注解格式来实现。

+ Mapper 接口：数据操作接口，常说的 DAO 接口，要和 Mapper 配置文件中的方法一一对应。
+ SqlSessionFactoryBuilder ：通过构建者模式，利用全局配置文件创建 SqlSessionFactory。
+ SqlSessionFactory：SqlSessionFactory 是创建 SqlSession 的工厂，可以通过`SqlSession openSession()`方法创建 SqlSession 对象。
+ SqlSession：类似于 JDBC 中的Connection，可以用 SqlSession 实例来直接执行被映射的 SQL 语句。
+ Executor：Mybatis 中所有的 Mapper 语句的执行都是通过 Executor 执行的；

## Mybatis 的执行流程

MyBatis 完整执行流程如下图所示：
![Snipaste_2019-10-15_09-04-08.jpg](https://i.loli.net/2019/10/15/7LJvx5lf4YS8Ttm.jpg)
Mybatis的执行流程说明：
1. 首先加载Mapper配置的 SQL 隐射文件，或者是注解的相关 SQL 内容。
2. 创建会话工厂，Mybatis 通过读取配置文件的信息来构造出会话工厂（SqlSessionFactory)。
3. 创建会话，根据会话工厂，Mybatis 就可以通过它来创建会话对象（SqlSession），会话对象是一个接口，该接口包含了对数据库操作的增、删、改查方法。
4. 创建执行器，因为会话对象本身不能直接操作数据库，所以它使用了一个叫数据库执行器（Executor）的接口来帮他执行操作。
5. 封装 SQL 对象，在这一步，执行器将待处理的 SQL 信息映封住到一个对象中（MappedStatement)该对象包括 SQL 语句、输入参数映射信息（Java 简单类型、HashMap 或 POJO）和输出映射信息（Java 简单类型、HashMap 或 POJO）.
6. 操作数据库，拥有了执行器和 SQL 信息封装对象就使用它们访问数据库了，最后在返回操作结果，结束流程。


**非常重要的一张图-分析代理dao的执行过程**
![非常重要的一张图-分析代理dao的执行过程.png](https://i.loli.net/2019/10/15/LwE7BOYqsmGpbPo.png)
**非常重要的一张图-分析编写dao实现类Mybatis的执行过程**
![非常重要的一张图-分析编写dao实现类Mybatis的执行过程.png](https://i.loli.net/2019/10/15/839a4pCisRXSKnH.png)