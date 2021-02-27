### 对于支持自动生成主键的数据库

比如 MySQL 和 SQL server，可以这只`useGeneratedKeys="true"`,然后再把 keyProperty 设置到目标属性就 OK 了。例如，对于可以自动生成主键的 Aurhor 表，

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```

如果你的数据库还支持多行插入, 你也可以传入一个 `Author` 数组或集合，并返回自动生成的主键。

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, 
     #{item.email}, #{item.bio})
  </foreach>
</insert>
```

###  对于不支持自动生成主键的数据库

对于不支持自动生成类型的数据库或可能不支持自动生成主键的 JDBC 驱动，MyBatis 可以用 <kbd><selectKey></kbd> 来生成主键。


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

此时， <kbd><selectKey></kbd>会先执行，User 的 id 被设置，然后插入语句会被调用。通过这种方式可以提供一个与数据库中自动生成主键类似的行为。

selectKey 元素描述如下：

```xml
<selectKey
  keyProperty="id"
  resultType="int"
  order="BEFORE"
  statementType="PREPARED">
```

+ keyProperty：selectKey 语句结果应该被设置的目标属性。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
+ keyColum：匹配属性的返回结果集中的列名称。如果希望得到多个生成的列，也可以是逗号分隔的属性名称列表。
+ resultType：结果的类型。MyBatis 通常可以推断出来，但是为了更加精确，写上也不会有什么问题。MyBatis 允许将任何简单类型用作主键的类型，包括字符串。如果希望作用于多个生成的列，则可以使用一个包含期望属性的 Object 或一个 Map。
+ order：这可以被设置为 BEFORE 或 AFTER。如果设置为 BEFORE，那么它会首先生成主键，设置 keyProperty 然后执行插入语句。如果设置为 AFTER，那么先执行插入语句，然后是 selectKey 中的语句 - 这和 Oracle 数据库的行为相似，在插入语句内部可能有嵌入索引调用。
+ statementType：与前面相同，MyBatis 支持 STATEMENT，PREPARED 和 CALLABLE 语句的映射类型，分别代表 PreparedStatement 和 CallableStatement 类型。

