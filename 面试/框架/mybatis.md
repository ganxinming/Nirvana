# 什么是 Mybatis？ 

1. Mybatis 是一个半 ORM（对象关系映射）框架，它内部封装了 JDBC，开发时 只需要关注 SQL 语句本身，不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。程序员直接编写原生态 sql，可以严格控制 sql 执行性 能，灵活度高。
2. MyBatis 可以使用 XML 或注解来配置和映射原生信息，将 POJO 映射成数 据库中的记录，避免了几乎所有的 JDBC 代码和手动设置参数以及获取结果集。
3. 通过 xml 文件或注解的方式将要执行的各种 statement 配置起来，并通过 java 对象和 statement 中 sql 的动态参数进行映射生成最终执行的 sql 语句，最 后由 mybatis 框架执行 sql 并将结果映射为 java 对象并返回。（从执行 sql 到返 回 result 的过程）。

(半ORM框架，只关注SQL本身，不关注JDBC加载，建立连接等)

# #{}和${}的区别是什么？

1. `#{}`是预编译处理，`${}`是字符串替换。
2. Mybatis 在处理#{}时，会将 sql 中的#{}替换为?号，调用 PreparedStatement 的 set 方法来赋值；
3. `Mybatis 在处理`{}时，就是把${}替换成变量的值。
4. 使用#{}可以有效的防止 SQL 注入，提高系统安全性。

# 当实体类中的属性名和表中的字段名不一样 ，怎么办 ？

第 1 种：通过在查询的 sql 语句中定义字段名的别名，让字段名的别名和实体类 的属性名一致

```
<select id=”selectorder” parametertype=”int” resultetype=”me.gacl.domain.order”>
        select order_id id, order_no orderno ,order_price price form
        orders where order_id=#{id};
        </select>
```

第 2 种：通过来映射字段名和实体类属性名的一一对应的关系。

```
<select id="getOrder" parameterType="int"  resultMap="orderresultmap">
    select * from orders where order_id=#{id}
</select>

<resultMap type=”me.gacl.domain.order” id=”orderresultmap”>
<!–用 id 属性来映射主键字段–>
<id property=”id” column=”order_id”>
<!–用 result 属性来映射非主键字段，property 为实体类属性名，column
        为数据表中的属性–>
<result property = “orderno” column =”order_no”/>
<result property=”price” column=”order_price” />
        </reslutMap>
```

# Mapper 接口的工作原理是什么？Mapper 接口里的方法，参数不同时，方法能重载吗？

Dao 接口即 Mapper 接口。接口的全限名，就是映射文件中的 namespace 的值；接口的方法名，就是映射文件中 Mapper 的 Statement 的 id 值；接口方法内的参数，就是传递给 sql 的参数。

Mapper 接口是没有实现类的，当调用接口方法时，接口全限名+方法名拼接字符串作为 key 值，可唯一定位一个 MapperStatement。在 Mybatis 中，每一个 `<select>`、`<insert>`、`<update>`、`<delete>`标签，都会被解析为一个MapperStatement 对象。

举例：com.mybatis3.mappers.StudentDao.findStudentById，可以唯一找到 namespace 为com.mybatis3.mappers.StudentDao 下面 id 为findStudentById 的 MapperStatement。

Mapper 接口里的方法，是不能重载的，因为是使用 全限名+方法名 的保存和寻找策略。Mapper 接口的工作原理是 JDK 动态代理，Mybatis 运行时会使用 JDK动态代理为 Mapper 接口生成代理对象 proxy，代理对象会拦截接口方法，转而执行 MapperStatement 所代表的 sql，然后将 sql 执行结果返回。

(1.一个select，insert等会被解析成一个MapperStatement对象

2.接口+方法作为key，确定对应一个MapperStatement(没有根据参数作为key，所以不能重载)

3.Mapper 接口的工作原理是 JDK 动态代理)

# Mybatis是如何将sql执行结果封装为目标对象并返回的？都有哪些映射形式？

第一种是使用标签，逐一定义数据库列名和对象属性名之间的映射关系。

第二种是使用 sql 列的别名功能，将列的别名书写为对象属性名。

有了列名与属性名的映射关系后，Mybatis 通过反射创建对象，同时使用反射给对象的属性逐一赋值并返回，那些找不到映射关系的属性，是无法完成赋值的。

(1.resultMap2.别名)

# Mybatis 是否支持延迟加载？如果支持，它的实现原理是什么？

Mybatis ==仅支持 association 关联对象和 collection 关联集合对象的延迟加载==，association 指的就是一对一，collection 指的就是一对多查询。在 Mybatis 配置文件中，可以配置是否启用延迟加载 lazyLoadingEnabled=true|false。

它的原理是，使用 CGLIB 创建目标对象的代理对象，当调用目标方法时，进入拦截器方法，比如调用 a.getB().getName()，拦截器 invoke()方法发现 a.getB()是null 值，那么就会单独发送事先保存好的查询关联 B 对象的 sql，把 B 查询上来，然后调用 a.setB(b)，于是 a 的对象 b 属性就有值了，接着完成 a.getB().getName()方法的调用。这就是延迟加载的基本原理。

当然了，不光是 Mybatis，几乎所有的包括 Hibernate，支持延迟加载的原理都是一样的。

(1.CGLIB代理对象，调用对象时为null，才会执行sql进行加载)



#### jdk代理：实现mapper接口代理

#### cglib代理：实现延迟加载





# 为什么说 Mybatis 是半自动 ORM 映射工具？它与全自动的区别在哪里？

Hibernate 属于全自动 ORM 映射工具，使用 Hibernate 查询关联对象或者关联集合对象时，可以根据对象关系模型直接获取，所以它是全自动的。而 Mybatis在查询关联对象或关联集合对象时，需要手动编写 sql 来完成，所以，称之为半自动 ORM 映射工具。

# 一对一、一对多的关联查询 ？

```
<mapper namespace="com.lcb.mapping.userMapper">
    <!--association 一对一关联查询 -->
    <select id="getClass" parameterType="int" resultMap="ClassesResultMap">
        select * from class c,teacher t where c.teacher_id=t.t_id and
        c.c_id=#{id}
    </select>

    <resultMap type="com.lcb.user.Classes" id="ClassesResultMap">
        <!-- 实体类的字段名和数据表的字段名映射 -->
        <id property="id" column="c_id"/>
        <result property="name" column="c_name"/>
        <association property="teacher" javaType="com.lcb.user.Teacher">
        <id property="id" column="t_id"/>
        <result property="name" column="t_name"/>
        </association>
    </resultMap>
    <!--collection 一对多关联查询 -->
    <select id="getClass2" parameterType="int" resultMap="ClassesResultMap2">
        select * from class c,teacher t,student s where c.teacher_id=t.t_id
        and c.c_id=s.class_id and c.c_id=#{id}
    </select>
    <resultMap type="com.lcb.user.Classes" id="ClassesResultMap2">
        <id property="id" column="c_id"/>
        <result property="name" column="c_name"/>
        <association property="teacher" javaType="com.lcb.user.Teacher">
        <id property="id" column="t_id"/>
        <result property="name" column="t_name"/>
    </association>
    <collection property="student" ofType="com.lcb.user.Student">
        <id property="id" column="s_id"/>
        <result property="name" column="s_name"/>
    </collection>
    </resultMap>
</mapper>
```

# Mybatis 的一级、二级缓存

1）一级缓存: 基于 PerpetualCache 的 HashMap 本地缓存，其存储作用域为 Session，当 Session flush 或 close 之后，该 Session 中的所有 Cache 就 将清空，默认打开一级缓存。

2）二级缓存与一级缓存其机制相同，默认也是采用 PerpetualCache，HashMap存储，不同在于其存储作用域为 Mapper(Namespace)，并且可自定义存储源，如 Ehcache。默认不打开二级缓存，要开启二级缓存，使用二级缓存属性类需要实现 Serializable 序列化接口(可用来保存对象的状态),可在它的映射文件中配置；

3）对于缓存数据更新机制，当某一个作用域(一级缓存 Session/二级缓存Namespaces)的进行了 C/U/D 操作后，默认该作用域下所有 select 中的缓存将被 clear。	

(1.一级缓存默认开始本地缓存，session级别。2.二级缓存默认关闭，需要实体类实现Serializable，NameSpace级别)

# 简述 Mybatis 的插件运行原理，以及如何编写一个插件

答：Mybatis 仅可以编写针对 ParameterHandler、ResultSetHandler、StatementHandler、Executor 这 4 种接口的插件，Mybatis 使用 JDK 的动态代理，为需要拦截的接口生成代理对象以实现接口方法拦截功能，每当执行这 4 种接口对象的方法时，就会进入拦截方法，具体就是 InvocationHandler 的 invoke()方法，当然，只会拦截那些你指定需要拦截的方法。

编写插件：实现 Mybatis 的 Interceptor 接口并复写 intercept()方法，然后在给插件编写注解，指定要拦截哪一个接口的哪些方法即可，记住，别忘了在配置文件中配置你编写的插件。

### 分页插件

三种分页：

逻辑分页：Mybatis是如何通过我们设置的RowBounds来返回分页结果的
原理：首先是将所有结果查询出来，然后通过计算offset和limit，只返回部分结果，操作在内存中进行，所以也叫内存分页。
弊端：当数据量大的时候，肯定是不行的。

物理分页：直接为sql添加limit

分页插件拦截器PageHelper，调用了startPage后，他会通过PageInterceptor对其后的第一个执行sql进行拦截拼接上limit语句。

(其实也是通过拦截器，去拦截sql并进行修改sql)



## 感觉好多事都是基于拦截器处理
