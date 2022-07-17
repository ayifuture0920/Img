## MyBatis

---

### 1. MyBatis有哪些特性

==参考答案==

- MyBatis 是支持定制化 SQL、存储过程以及高级映射的优秀的持久层框架

- MyBatis 避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集

- MyBatis可以使用简单的XML或注解用于配置和原始映射，将接口和Java的POJO（Plain Old Java Objects，普通的Java对象）映射成数据库中的记录

- MyBatis 是一个 半自动的ORM（Object Relation Mapping）框架



### 2. 为什么说 MyBatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

```xml
<!--相关概念：-->
ORM(Object Relationship Mapping) 对象关系映射
  * 对象：Java实体类对象
  * 关系：关系型数据库
  * 映射：二至之间的对应关系
- 类<------------->数据库中的一张表(table)
- 对象<----------->表的一条记录
- 对象.属性<------->记录某一个字段的值
```

==参考答案==

Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 MyBatis 在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。



### 3. MyBatis输入输出支持的类型有哪些？

==参考答案==

**`parameterType`（输入）和`resultType` / `resultMap`（输出）：**

- 基本数据类型，如整数、小数等

- 集合类型，如Map等

- 自定义的JavaBean

其中，简单的类型，其数值直接映射到参数上。对于Map或JavaBean则将其属性按照名称映射到参数上。

```xml
查询的标签select必须设置属性resultType或resultMap，用于设置实体类和数据库表的映射关系  
 - resultType：自动映射，用于属性名和表中字段名一致的情况  
 - resultMap：自定义映射，用于一对多或多对一或字段名和属性名不一致的情况  
```

### 4. MyBatis中的`${}`和`#{}`有什么区别？

==参考答案==

使用`#{}`设置参数时，MyBatis会创建预编译的SQL语句，然后在执行SQL语句时MyBatis会为预编译SQL中的占位符（`?`）赋值。预编译的SQL语句执行效率高，并且可以防止注入攻击。（常用）

使用`${}`设置参数时，MyBatis只是创建普通的SQL语句，然后在执行SQL语句时MyBatis将参数直接拼入到SQL里。这种方式在效率、安全性上均不如前者，但是可以解决一些特殊情况下的问题。例如，在一些动态表格（根据不同的条件产生不同的动态列）中，我们要传递SQL的列名，根据某些列进行排序，或者传递列名给SQL都是比较常见的场景，这就无法使用预编译的方式了。

### 5.MyBatis的xml文件和Mapper接口是怎么绑定的？

==参考答案==

是通过xml文件中，`<mapper> `根标签的namespace属性进行绑定的，即namespace属性的值需要配置成接口的全限定名称，MyBatis内部就会通过这个值将这个接口与这个xml关联起来。



### 5. MyBatis 执行批量插入，能返回数据库主键列表吗？

==参考答案==

能，在`<insert>`中设置`useGeneratedKeys="true"  keyProperty="id"`

```xml
<insert useGeneratedKeys="true"  keyProperty="id">
    insert into table values(null, #{username}, #{password}, #{age}, #{gender})
</insert>
```

### 6.Xml 映射文件中，除了常见的 select|insert|update|delete 标签之外，还有哪些标签？

==参考答案==

答：还有很多其他的标签， `<resultMap>` 、 `<parameterMap>` 、 `<sql>` 、 `<include>` 、 `<selectKey>` ，加上动态 sql 的 9 个标签， `trim|where|set|foreach|if|choose|when|otherwise|bind` 等，其中 `<sql>` 为 sql 片段标签，通过 `<include>` 标签引入 sql 片段， `<selectKey>` 为不支持自增的主键生成策略标签。



### 7. 简述 MyBatis 的 Xml 映射文件和 MyBatis 内部数据结构之间的映射关系？

==参考答案==

MyBatis 将所有 Xml 配置信息都封装到 All-In-One 重量级对象 Configuration 内部。在 Xml 映射文件中， `<parameterMap>` 标签会被解析为 `ParameterMap` 对象，其每个子元素会被解析为 ParameterMapping 对象。 `<resultMap>` 标签会被解析为 `ResultMap` 对象，其每个子元素会被解析为 `ResultMapping` 对象。每一个 `<select>、<insert>、<update>、<delete>` 标签均会被解析为 `MappedStatement` 对象，标签内的 sql 会被解析为 BoundSql 对象。



### 8. MyBatis 是如何将 sql 执行结果封装为目标对象并返回的？都有哪些映射形式？

==参考答案==

- 第一种是使用 `<resultType>` 标签，为每个字段设置别名，将列别名书写为对象属性名*（一对一）*

- 第二种是使用 `<resultMap>` 标签，逐一定义列名和对象属性名之间的映射关系*（一对一、多对一、一对多）*

有了列名与属性名的映射关系后，MyBatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。



### 9. MyBatis是如何实现多对一的映射

==参考答案==

- 第一种是使用`<resultMap>`标签和级联方式处理映射关系

```xml
<resultMap id="empAndDeptResultMapOne" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
	<result property="dept.did" column="did"></result>
	<result property="dept.deptName" column="dept_name"></result>
</resultMap>
<!--Emp getEmpAndDept(@Param("eid")Integer eid);-->
<select id="getEmpAndDept" resultMap="empAndDeptResultMapOne">
	select * from t_emp left join t_dept on t_emp.eid = t_dept.did where t_emp.eid = #{eid}
</select>
```

- 第二种是使用 `<resultMap>` 标签和`<association>`子标签处理映射关系

```xml
<resultMap id="empAndDeptResultMapTwo" type="Emp">
	<id property="eid" column="eid"></id>
	<result property="empName" column="emp_name"></result>
	<result property="age" column="age"></result>
	<result property="sex" column="sex"></result>
	<result property="email" column="email"></result>
	<association property="dept" javaType="Dept">
		<id property="did" column="did"></id>
		<result property="deptName" column="dept_name"></result>
	</association>
</resultMap>
<!--Emp getEmpAndDept(@Param("eid")Integer eid);-->
<select id="getEmpAndDept" resultMap="empAndDeptResultMapTwo">
	select * from t_emp left join t_dept on t_emp.eid = t_dept.did where t_emp.eid = #{eid}
</select>
```

- 第三种是使用`<resultMap>` 标签和`<association>`子标签分步查询

```xml
<!--DeptMapper.xml -->
<select id="getEmpAndDeptByStepTwo" resultType="Dept">
    select * from t_dept where did = #{did};
</select>
```

```xml
<!--EmpMapper.xml
	select:设置分部查询的sql的唯一标识(namespace.SQLID或mapper接口的全类名.方法名)
	column:设置分步查询的条件
-->
<resultMap id="getEmpAndDeptByStep1" type="Emp">
    <id property="eid" column="eid" />
    <result property="empName" column="emp_name" />
    <result property="age" column="age" />
    <result property="gender" column="gender" />
    <result property="email" column="email" />
    <association property="dept"
                 select="com.mybatis.mapper.DeptMapper.getEmpAndDeptByStepTwo"
                 column="did" />
</resultMap>
<select id="getEmpAndDeptByStep" resultMap="getEmpAndDeptByStep1">
    select * from t_emp where eid = #{eid};
</select>
```



### 10. MyBatis 是否支持延迟加载？如果支持，它的实现原理是什么？

==参考答案==

MyBatis 仅支持 association 关联对象和 collection 关联集合对象的延迟加载，association 指的就是一对一，collection 指的就是一对多查询。在 MyBatis 配置文件中，可以配置是否启用延迟加载 `lazyLoadingEnabled=true|false。`

```xml
<!--全局设置-->
<settings>
    <setting name="lazyLoadingEnabled" value="true" />
</settings>
```

```xml
<!--afetchType：当开启了全局延迟加载之后，通过此属性手动控制延迟加载的效果
	fetchType=lazy(表示开启)|eager(表示关闭)
-->
<association fetchType="eager/lazy" />
```

它的原理是，使用 `CGLIB` 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 `a.getB().getName()` ，拦截器 `invoke()` 方法发现 `a.getB()` 是 null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 `a.getB().getName()` 方法的调用。这就是延迟加载的基本原理。



### 11. MyBatis里如何实现一对多关联查询？

==参考答案==

一对多映射有两种配置方式，都是使用collection标签实现的。在此之前，为了能够存储一对多的数据，需要在主表对应的实体类中增加集合属性，用于封装子表对应的实体类。

- 第一种是使用 `<resultMap>` 标签和`<collection>`子标签处理映射关系

```xml
<resultMap id="deptAndEmp" type="Dept">
    <id property="did" column="did" />
    <result property="deptName" column="dept_name" />
    <collection property="emps" ofType="Emp">
        <id property="eid" column="eid" />
        <result property="empName" column="emp_name" />
        <result property="age" column="age" />
        <result property="gender" column="gender" />
        <result property="email" column="email" />
    </collection>
</resultMap>
<select id="getDeptAndEmp" resultMap="deptAndEmp">
    select * from t_dept d left join t_emp e on d.did = e.did where d.did = #{did};
</select>
```

- 第二种是使用`<resultMap>` 标签和`<mapper>`子标签分步查询

```xml
<!--EmpMapper.xml -->
<resultMap id="deptAndEmpStepTwo" type="Emp">
    <id property="eid" column="eid" />
    <result property="empName" column="emp_name" />
    <result property="age" column="age" />
    <result property="gender" column="gender" />
    <result property="email" column="email" />
</resultMap>
<select id="getDeptAndEmpByStepTwo" resultMap="deptAndEmpStepTwo">
    select * from t_emp where did = #{did};
</select>
```

```xml
<!--DeptMapper.xml
	select:设置分部查询的sql的唯一标识(namespace.SQLID或mapper接口的全类名.方法名)
	column:设置分步查询的条件
-->
<resultMap id="deptAndEmpStepOne" type="Dept">
    <id property="did" column="did" />
    <result property="deptName" column="dept_name" />
    <collection property="emps"
                select="com.mybatis.mapper.EmpMapper.getDeptAndEmpByStepTwo"
                column="did"
                fetchType="lazy"/>
</resultMap>
<select id="getDeptAndEmpByStepOne" resultMap="deptAndEmpStepOne">
    select * from t_dept where did = #{did};
</select>
```

### 12. MyBatis 动态 sql 是做什么的？都有哪些动态 sql？能简述一下动态 sql 的执行原理不？

==参考答案==

答：MyBatis 动态 sql 可以让我们在 Xml 映射文件内，以标签的形式编写动态 sql，完成逻辑判断和动态拼接 sql 的功能，MyBatis 提供了 9 种动态 sql 标签 `if|where|trim|set|foreach|choose-when-otherwise|bind` 。

其执行原理为，使用 OGNL 从 sql 参数对象中计算表达式的值，根据表达式的值动态拼接 sql，以此来完成动态 sql 的功能。

*OGNL http://www.yiidian.com/struts2/struts2-ognl.html*



### 13. 了解MyBatis缓存机制吗？

==参考答案==

MyBatis的缓存分为一级缓存和二级缓存：

- 一级缓存：

一级缓存也叫本地缓存，它默认会启用，并且不能关闭。一级缓存存在于SqlSession的生命周期中，即它是SqlSession级别的缓存。在同一个 SqlSession 中查询时，MyBatis 会把执行的方法和参数通过算法生成缓存的键值，将键值和查询结果存入一个Map对象中。如果同一个SqlSession 中执行的方法和参数完全一致，那么通过算法会生成相同的键值，当Map 缓存对象中己经存在该键值时，则会返回缓存中的对象。

- 二级缓存：

二级缓存存在于SqlSessionFactory 的生命周期中，即它是SqlSessionFactory级别的缓存。若想使用二级缓存，需要在进行两处配置：

1. 在MyBatis 的全局配置settings 中有一个参数cacheEnabled，这个参数是二级缓存的全局开关，默认值是true ，初始状态为启用状态。
2. MyBatis 的二级缓存是和命名空间绑定的，即二级缓存需要配置在Mapper.xml 映射文件中。在保证二级缓存的全局配置开启的情况下，给Mapper.xml 开启二级缓存只需要在Mapper. xml 中添加如下代码：

```xml
<cache />
```

二级缓存具有如下效果：

- 映射文件中的所有SELECT 语句将会被缓存。

- 映射文件中的所有时INSERT 、UPDATE 、DELETE 语句会刷新缓存。

- 缓存会使用Least Recently Used ( LRU ，最近最少使用的）算法来收回。

- 根据时间表（如no Flush Interval ，没有刷新间隔），缓存不会以任何时间顺序来刷新。

- 缓存会存储集合或对象（无论查询方法返回什么类型的值）的1024 个引用。

- 缓存会被视为read/write（可读／可写）的，意味着对象检索不是共享的，而且可以安全地被调用者修改，而不干扰其他调用者或线程所做的潜在修改。

  

### 14. MyBatis缓存查询的顺序？

- 先查询二级缓存，因为二级缓存中可能会有其他程序已经查出来的数据，可以拿来直接使用  
- 如果二级缓存没有命中，再查询一级缓存  
- 如果一级缓存也没有命中，则查询数据库  
- SqlSession关闭之后，一级缓存中的数据会写入二级缓存

### 15. MyBatis 是如何进行分页的？分页插件的原理是什么？

==参考答案==

**(1)** MyBatis 使用 RowBounds 对象进行分页，它是针对 ResultSet 结果集执行的内存分页，而非物理分页；

**(2)** 可以在 sql 内直接书写带有物理分页的参数来完成物理分页功能，

**(3)** 也可以使用分页插件来完成物理分页。

分页插件的基本原理是使用 MyBatis 提供的插件接口，实现自定义插件，在插件的拦截方法内拦截待执行的 sql，然后重写 sql，根据 dialect 方言，添加对应的物理分页语句和物理分页参数。

举例： `select _ from student` ，拦截 sql 后重写为： `select t._ from （select \* from student）t limit 0，10`



### 16. 佳实践中，通常一个 Xml 映射文件，都会写一个 Dao 接口与之对应，请问，这个 Dao 接口的工作原理是什么？Dao 接口里的方法，参数不同时，方法能重载吗？

==参考答案==

Dao 接口，就是人们常说的 `Mapper` 接口。接口的全限名，就是映射文件中的 namespace 的值，接口的方法名，就是映射文件中 `MappedStatement` 的 id 值，接口方法内的参数，就是传递给 sql 的参数。 `Mapper` 接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个 `MappedStatement` 。*举例： `com.mybatis3.mappers. StudentDao.findStudentById` ，可以唯一找到 namespace 为 `com.mybatis3.mappers. StudentDao` 下面 `id = findStudentById` 的 `MappedStatement` 。在 MyBatis 中，每一个 `<select>` 、 `<insert>` 、 `<update>` 、 `<delete>` 标签，都会被解析为一个 `MappedStatement` 对象。*

**Mybatis 的 Dao 接口可以有多个重载方法，需要在映射文件中使用动态sql来实现，而且多个接口对应的映射必须只有一个，否则启动会报错。**

Mybatis 版本 3.3.0，亲测如下：

```java
/**
 * Mapper接口里面方法重载
 */
public interface StuMapper {

	List<Student> getAllStu();

	List<Student> getAllStu(@Param("id") Integer id);
}
```

然后在 `StuMapper.xml` 中利用 Mybatis 的动态 sql 就可以实现。

```xml
<select id="getAllStu" resultType="com.pojo.Student">
    select * from student
    <where>
        <if test="id != null">
            id = #{id}
        </if>
    </where>
</select>
```