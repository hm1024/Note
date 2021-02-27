# Mybatis中的注解汇总

**@Insert**

```java
@Insert("insert into students(stud_id,name,email,addr_id, phone) values(#{studId},#{name},#{email},#{address.addrId},#{phone})")
int insertStudent(Student student);
```

自动生成主键

可以使用 @Options注解的userGeneratedKeys和keyProperty属性让数据库产生 auto_increment（自增长）列的值，然后将生成的值设置到输入参数对象的属性中。

```java
@Insert("insert into students(name,email,addr_id, phone)
         values(#{name},#{email},#{address.addr Id},#{phone})")
@Options(useGeneratedKeys = true, keyProperty = "studId")
int insertStudent(Student student);
```

有一些数据库如Oracle，并不支持AUTO_INCREMENT列属性，它使用序列（SEQUENCE）来产生主键的值。

我们可以使用 @SelectKey注解来为任意SQL语句来指定主键值，作为主键列的值。假设我们有一个名为STUD_ID_SEQ的序列来生成STUD_ID主键值。

```java
@Insert("insert into students(stud_id,name,email,addr_id,phone) values(#{studId},#{name},#{email},#{address.addrId},#{phone})")
@SelectKey(statement="select my_seq.nextval from dual",  
keyProperty="studId", resultType=int.class, before=true)
int insertStudent(Student student);
```

这里使用了@SelectKey来生成主键值，并且存储到了student对象的studId属性上。由于我们设置了before=true,该语句将会在执行INSERT语句之前执行.

**@Delete**

我们可以使用 @Delete注解来定义一个DELETE映射语句，如下所示：

```java
@Delete("delete from students where stud_id=#{studId}")
int deleteStudent(int studId);
```
**@Update**

 我们可以使用 @Update注解来定义一个UPDATE映射语句，如下所示：

   ```java
@Update("update students set name=#{name}, email=#{email},  
phone=#{phone} where stud_id=#{studId}")
int updateStudent(Student student);
   ```

**@Select**

```java
@Select("select stud_id as studid, name, email, phone from
students where stud_id=#{studId}")
Student findStudentById(Integer studId);
```

为了将列名和对象中属性名匹配，为stud_id起了一个studId的别名。如果返回了多行结果，将抛出TooManyResultsException异常。

**结果映射**

可以将查询结果通过别名或者是@Results注解与实体类的属性映射起来。

```java
@Select("select * from students")
@Results(
    {
        @Result(id = true, column = "stud_id", property = "studId"),
        @Result(column = "name", property = "name"),
        @Result(column = "email", property = "email"),
        @Result(column = "addr_id", property = "address.addrId")
    })
List<Student> findAllStudents();
```

也可以在xml中定义结果集，然后在注解中引用,为了结果集能更好的重用。

**一对一映射**

```java
@Select("select * from students where stud_id=#{studId} ")
@Results(
    {
        @Result(id = true, column = "stud_id", property = "studId"),
        @Result(column = "name", property = "name"),
        @Result(column = "email", property = "email"),
        @Result(property = "address", column = "addr_id",
        one = @One(select = "com.briup.mappers.Student Mapper.findAddressById"))
     })
Student selectStudentWithAddress(int studId);
```

**一对多映射**

MyBatis提供了@Many注解，用来使用嵌套Select语句加载一对多关联查询。

```java
 public interface TutorMapper{
        @Select("select addr_id as addrId, street, city, state, zip,
                country from addresses where addr_id=#{id}")
        Address findAddressById(int id);

        @Select("select * from courses where tutor_id=#{tutorId}")
        @Results(
        {
            @Result(id = true, column = "course_id", property = "courseId"),
            @Result(column = "name", property = "name"),
            @Result(column = "description", property = "description"),
            @Result(column = "start_date" property = "startDate"),
            @Result(column = "end_date" property = "endDate")
        })
        List<Course> findCoursesByTutorId(int tutorId);

        @Select("SELECT tutor_id, name as tutor_name, email, addr_id
                FROM tutors where tutor_id=#{tutorId}")
        @Results(
        {
            @Result(id = true, column = "tutor_id", property = "tutorId"),
            @Result(column = "tutor_name", property = "name"),
            @Result(column = "email", property = "email"),
            @Result(property = "address", column = "addr_id",
            one = @One(select = "com.briup.mappers.TutorMapper.findAddressById")),
            @Result(property = "courses", column = "tutor_id",
            many = @Many(select = "com.briup.mappers.TutorMapper.findCoursesByTutorId"))
        })
        Tutor findTutorById(int tutorId);
    }
```

 这里使用了@Many注解的select属性来指向一个方法，该方法将返回一个List<Course>对象。

 **@Alias**

+ 作用：为JavaBeans起别名。

+ 用法：在 JavaBean 类上使用，在括号中指定别名。

  ```java
  @Alias("author")
  public class Author {
      ...
  }
  ```


**@MappedJdbcTypes**

+ 作用：重写类型处理器或创建你自己的类型处理器来处理不支持的或非标准的类型，
+ 用法：在自定义的类型处理器上，@MappedJdbcTypes(JdbcType.VARCHAR)
















































**