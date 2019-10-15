## Mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>    
    <properties  resource="jdbcConfig.properties">
    </properties>
    <settings>
        <setting name="cacheEnabled" value="true"/>
    </settings>
    <typeAliases>
        <package name="com.minghai.domain"></package>
    </typeAliases>
    <!--配置环境-->
    <environments default="mysql">
        <!--配置mysql的环境-->
        <environment id="mysql">
           <!--配置事务-->
            <transactionManager type="JDBC"></transactionManager>
            <!--配置连接池-->
            <dataSource type="POOLED">
                <property name="driver" value="${jdbc.driver}"></property>
                <property name="url" value="${jdbc.url}"></property>
                <property name="username" value="${jdbc.username}"></property>
                <property name="password" value="${jdbc.password}"></property>
            </dataSource>
        </environment>
    </environments>
    <!--配置映射文件的位置-->
    <mappers>
        <package name="com.minghai.dao"></package>
    </mappers>
</configuration>
```

## mybatis-mapper.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.minghai.dao.IUserDao">

    <!--开启user支持二级缓存-->
    <cache/>
</mapper>
```



## jdbcConfig.peroperties

```propertiess
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/database_name
jdbc.username=root
jdbc.password=root
```

