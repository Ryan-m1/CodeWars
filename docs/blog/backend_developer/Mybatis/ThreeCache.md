# 🏆深入浅出MyBatis缓存



## 1. 缓存的意义

众所周知，和数据库打交道避免不了磁盘IO操作，那如果频繁的IO操作一定会对性能造成影响，所以减少与数据库的交互次数从而降低数据库压力进而提升查询效率是必要的。缓存是其中一种实现方式，简单的理解其实缓存就是内存中专门的一块区域，当从数据库中查询到一些数据将其放入缓存中，下次查询相同的数据时可以直接从缓存中获取数据即可，这样可减少了一步和数据库交互的过程。



MyBatis提供了三级缓存机制，虽然MyBatis的缓存机制有些鸡肋，大部分开发人员多数情况都只会使用MyBatis默认缓存配置，虽然MyBatis缓存机制有一些不足之处，出于学习还是决定写下文章。



## 2. 一级缓存

### 2.1 猜想

由一串测试代码引出今天的主角—一级缓存

`````java
    /**
     * 测试一级缓存
     */
    @Test
    public void TEST_QUERY_BY_FIRST_CACHE() {
        //代理模式获取代理类
        IUserMapper userMapper = sqlSession.getMapper(IUserMapper.class);
        //第⼀次sql语句查询 将查询结果放入缓存中
        User user1 = userMapper.findById(1);
        System.out.println("第一次查询：" + user1);
        //第⼆次sql语句查询，由于是同⼀个sqlSession,会去查询缓存 如果缓存中没有则查库 缓存中有则直接取缓存
        User user2 = userMapper.findById(1);
        System.out.println("第二次查询：" + user2);
        System.out.println(user1 == user2);
    }

`````

![image-20210824222859484](img/image-20210824222859484.png)

上述测试代码，在同一个sqlSession中执行2次查询操作并记录操作日志，不过会发现上图sql执行语句只记录了一次查询操作，言外之意其实在同一个session中第一次sql查询会查库，后将查询结果放入缓存中，第二次sql查询，就会直接去查询缓存。不过目前来说也只是猜想，稍后我们翻看源码的时候会验证此猜想。

***

不过话说回来，在同一个sqlSession这个前提下，多次查询走缓存不去查库，那如果库中数据更改了呢，数据被修改，删除，或者新增呢，岂不是数据不一致了，所以我猜想当执行了数据修改，删除，新增操作会重置缓存（重新查一次库），避免脏读，实际上MyBatis就是这么做的。

编写如下测试类，运行代码：

```java
  /**
     * 测试一级缓存 提交 是否重置缓存
     */
    @Test
    public void TEST_QUERY_COMMIT_BY_FIRST_CACHE() {
        //代理模式获取代理类
        IUserMapper userMapper = sqlSession.getMapper(IUserMapper.class);
        //第⼀次sql语句查询 将查询结果放入缓存中
        User user1 = userMapper.findById(1);
        System.out.println("第一次查询：" + user1);
        //更新操作 并提交sqlSession
        user1.setUsername("MRyan666");
        userMapper.updateById(user1);
        sqlSession.commit();
        User user2 = userMapper.findById(1);
        System.out.println("第二次查询：" + user2);
        System.out.println(user1 == user2);
    }
```

![image-20210824225915612](img/image-20210824225915612.png)

根据上述测试代码，初步证实了我的猜想在同一的sqlSession作用范围下，如果中间操作sqlSession执行了commit操作，其实也就是（删除，更新，新增）那么会清除sqlSession的缓存，保障缓存中拿到的数据一定是最新的，避免脏读。

到这里先透一个底，其实上面所提及到的缓存在MyBatis中被命名为一级缓存。

### 2.2 证明

说了这么多猜想，眼见为实，接下来跟随作者一起通过对MyBatis源码解读探究一级缓存的原理及本质，在开始之前，提出几个问题，带入问题分析源码事半功倍。

- 一级缓存是什么 数据结构是怎样的？
- 一级缓存什么时候被构建
- 一级缓存的不足及优化



>本文尚未结束，请等待作者更新本文



## 3. 二级缓存











