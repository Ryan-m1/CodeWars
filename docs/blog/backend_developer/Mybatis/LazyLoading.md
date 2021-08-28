# 🏆延迟加载原理剖析

## 1. 延迟加载的介绍及使用

本文将针对MyBatis提供的延迟加载（懒加载）原理剖析。

### 1.1 延迟加载是什么？

简单的来说延迟加载就是，在需要用到数据的时候进行加载，不需要用到数据就不进行加载。

假设数据库中涉及两张表，用户表AND订单表（一对多的关系），假设一个用户由很多订单，那么在查询用户的时候，需不需要当前用户关联的订单数据查询出来？

通常来说查询用户信息的时候，肯定是需要用到用户订单的时候在查询为好，尤其是一对多的多表查询，通常都建议采用延迟加载，因为单标查询肯定要比多表关联查询速度要快，先从单表查询，接着需要时在关联表进行关联查询，会提高数据库性能。

这就是延迟加载，又叫懒加载。

### 1.2 如何启用延迟加载

MyBatis默认不启用延迟加载，启用延迟加载则需要我们进行一些简单的配置即可。

首先可以MyBatis SqlMapper核心配置文件中设置setting属性 设置全局懒加载

```sml
 <settings>
        <!-- 开启全局延迟加载,默认值是true -->
        <setting name="lazyLoadingEnabled" value="true" />
        <!-- 设置全局积极懒加载,默认值是true -->
        <setting name="aggressiveLazyLoading" value="false"/>
 </settings>
```

或者在mapper配置文件中配置fetchType标签 设置局部懒加载

```xml
    <resultMap id="userMap" type="com.mryan.pojo.User">
        <id property="id" column="id"></id>
        <result property="username" column="username"></result>
        <!--fetchType="lazy"  懒加载策略  fetchType="eager"  立即加载策略 -->
        <collection property="orderList" ofType="com.mryan.pojo.Order"
                    select="com.mryan.mapper.IOrderMapper.findOrderByUid" column="id" fetchType="lazy">

            <id property="id" column="uid"/>
            <result property="orderTime" column="ordertime"/>
            <result property="total" column="total"/>
        </collection>
    </resultMap>


    <select id="findOrderByUid" resultType="com.mryan.pojo.Order">
        select *
        from orders
        where uid = #{uid}
    </select>
```

注意这里 局部的懒加载策略优先级药高于全局的懒加载策略

### 1.3 测试延迟加载的作用

为测试多表关联，表结构及pojo结构如下：

**User**

![image-20210829000936315](img/image-20210829000936315.png)

```java
public class User implements Serializable {
    private Integer id;
    private String username;
    // 用户关联的订单数据
    private List<Order> orderList;
  //省略
```

**Orders**

![image-20210829000958777](img/image-20210829000958777.png)

```java
public class Order implements Serializable {
    private Integer id;
    private String orderTime;
    private Double total;
    // 表明该订单属于哪个用户
    private User user;
  //省略
```

在完成上述配置之后，跑一个单元测试，试一下延迟加载是否生效

```java
    /*
    测试 延迟加载查询
   */
    @Test
    public void TEST_QUERY_LAZY() throws IOException {
        InputStream inputStream = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(inputStream);
        SqlSession sqlSession = factory.openSession();
        User user = sqlSession.selectOne("com.mryan.mapper.IUserMapper.findById", 1);
        //延迟加载生效 下方输出语句 不涉及到orders表 于是不会打印orders相关日志
        System.out.println("user：" + user.getUsername());
        //涉及orders表 才会执行相关SQL语句 加载orders执行日志 （延迟加载 什么时候用什么时候查）
        System.out.println("orders：" + user.getOrderList());
    }
```

![image-20210829002048767](img/image-20210829002048767.png)

可以发现user.getUsername()的时候只出发了查询用户的sql语句，而user.getOrderList()时才触发了查询订单的sql语句，由此可见延迟加载生效，做到了不用不查，用到在查。

## 2. 延迟加载源码剖析

下面将对源码进行阅读，剖析Mybatis延迟加载的原理。

在MyBatis系列文章第一篇[浅析MyBatis执行SQL流程](ExecuteSQL.md)就对SQL的执行流程进行了讲解。简单复习一下大致流程，SQL查询语句的执行是由SqlSession分发交由Executor托管执行，调度StatementHandler负责JDBC statement操作，之后下发给ParameterHandler负责对用户传递参数进行转化处理SQL参数，再接着执行SQL语句，最后通过ResultSetHandler对返回结果进行封装处理返回。

根据刚刚对延迟加载功能的测试，我们也能大致找到突破入口，通过最后的ResultSetHandler对结果封装处理返回的时候，根据调用的getting方法的实例名称，来相对应的加载目标对象结果，不就实现了延迟加载的功能吗。

MyBatis其实就是这么做的，接下来根据翻看源码来证实我们的猜想



> 本文未完待续，请等待作者更新









## 3. 总结





## 预告



