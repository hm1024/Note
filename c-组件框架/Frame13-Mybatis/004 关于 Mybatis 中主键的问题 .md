```xml
    <!--保存用户-->
    <insert id="saveUser" parameterType="com.minghai.domain.User">
        <!-- 配置插入数据后，获取插入数据的id-->
        <selectKey keyProperty="id" keyColumn="id" resultType="int" order="AFTER">
            select last_insert_id();
        </selectKey>
        insert into user(username,address,sex,birthday) 
        values(#{username},#{address},#{sex},#{birthday});
    </insert>
```

