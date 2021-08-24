# 🏆深入浅出MyBatis缓存



## 缓存的意义

众所周知，和数据库打交道避免不了磁盘IO操作，那如果频繁的IO操作一定会对性能造成影响，所以减少与数据库的交互次数从而降低数据库压力进而提升查询效率是必要的。缓存是其中一种实现方式，简单的理解其实缓存就是内存中专门的一块区域，当从数据库中查询到一些数据将其放入缓存中，下次查询相同的数据时可以直接从缓存中获取数据即可，这样可减少了一步和数据库交互的过程。



MyBatis提供了三级缓存机制，虽然MyBatis的缓存机制有些鸡肋，大部分开发人员多数情况都只会使用MyBatis默认缓存配置，虽然MyBatis缓存机制有一些不足之处，出于学习还是决定写下文章。



## 一级缓存

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











## 二级缓存











