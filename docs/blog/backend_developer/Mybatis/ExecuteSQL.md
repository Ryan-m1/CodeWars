# 🏆浅析MyBatis执行SQL流程

本文主要通过浅析MyBatis如何执行一个SQL语句（流程）为引，引出MyBatis的整体框架设计。

作为MyBatis系列第一篇文章，必然先了解一下MyBatis的由来，所以在文章开始之前，我们来思考一个问题，MyBatis为什么诞生？

## 1. 诞生

一个轮子的诞生定是有其原因的，JDBC的诞生链接了程序和数据库，可以为多种关系数据库提供统一访问。 但是传统的JDBC方式存在很多缺点

例如：

- 数据库连接创建、释放频繁造成系统资源浪费，从⽽影响系统性能。
- Sql语句在代码中硬编码，造成代码不易维护，实际应⽤中sql变化的可能较⼤，sql变动需要改变 java代码。
- 使⽤preparedStatement向占有位符号传参数存在硬编码，因为sql语句的where条件不⼀定，可能多也可能少，修改sql还要修改代码，系统不易维护。
- 对结果集解析存在硬编码(查询列名)，sql变化导致解析代码变化，系统不易维护。

为解决以上的问题，MyBatis被创造出来了，那它究竟是怎么规避解决以上原生JDBC的缺点呢，请往下看，正文来喽。

## 2. 架构设计

上面有提到JDBC的问题

1. 数据库连接创建，释放频繁导致的资源浪费，为解决这类问题，我们能想到的解决方案：肯定是池化，利用使用数据库连接池加载并初始化连接资源，池化管理。
2. SQL语句在代码中的硬编码问题，为解决这类问题，我们能想到的解决方案：将SQL语句抽离到XML配置文件中，不需要改动代码，配置作为分离SQL及业务之间的桥梁。
3. 占位符传参硬编码问题，为解决这类问题，我们能想到的解决方案：利用反射，内省等技术，自动将实体与表进行属性和字段自动映射。

当然我不是说说而已，MyBatis其实也是这么做的，ps：（这里我们先简单提一嘴，后面的文章《手撸简易版MBatis》会详细介绍）

重点来喽，MyBatis架构师如何设计的

![](https://img-blog.csdnimg.cn/31de983d94854a809c8edb21e8181546.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDE2MjE0,size_16,color_FFFFFF,t_70)

Mybatis的功能架构分为三层：

1. API接⼝层：提供给外部使⽤的接⼝ API，开发⼈员通过这些本地API来操纵数据库。接⼝层⼀接收 到调用请求就会调⽤数据处理层来完成具体的数据处理。

   MyBatis和数据库的交互有两种⽅式：

    - 使⽤传统的MyBatis提供的API

    - 使⽤Mapper代理的⽅式
2. 数据处理层：负责具体的SQL查找、SQL解析、SQL执⾏和执⾏结果映射处理等。它主要的目的是根据调用的请求完成⼀次数据库操作。
3. 基础⽀撑层：负责最基础的功能⽀撑，包括连接管理、事务管理、配置加载和缓存处理，这些都是共用的东⻄，将他们抽取出来作为最基础的组件。为上层的数据处理层提供最基础的支撑。

### 主要组件及其相互关系

![](https://img-blog.csdnimg.cn/f7c061fc7d97485d87a702b280d050c7.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3FxXzM1NDE2MjE0,size_16,color_FFFFFF,t_70)

SqlSession--》作为MyBatis⼯作的主要顶层API，表示和数据库交互的会话，完成必要数 据库增删改查功能。

Executor--》调度执行StatementHandler、ParameterHandler、ResultHandler执行相应的SQL语句

StatementHandler--》封装了JDBC Statement操作，负责对JDBC statement的操作，如设置参数、将Statement结果集转换成List集合。

ParameterHandler--》负责对用户传递的参数转换成JDBC Statement所需要的参数。处理SQL参数。

ResultSetHandler--》结果集ResultSet封装处理返回。

TypeHandler--》负责java数据类型和jdbc数据类型之间的映射和转换。

MappedStatement--》 MappedStatement维护了⼀条＜select | update | delete | insert＞节点的封装。

SqlSource--》负责根据⽤户传递的parameterObject，动态地⽣成SQL语句，将信息封装到BoundSql对象中，并返回。

BoundSql--》表示动态⽣成的SQL语句以及相应的参数信息。

## 3. 执行SQL语句总体流程

1. 加载配置并初始化 触发条件：加载配置⽂件 配置来源于两个地⽅，⼀个是配置⽂件(主配置⽂件conf.xml,mapper⽂件*.xml)
   ,—个是java代码中的注解，将主配置⽂件内容解析封装到Configuration,将sql的配置信息加载成为⼀个MappedStatement 对象，存储在内存之中
2. 接收调⽤请求 触发条件：调⽤Mybatis提供的API传⼊参数：为SQL的ID和传⼊参数对象处理过程：将请求传递给下层的请求处理层进⾏处理。
3. 处理操作请求 触发条件：API接⼝层传递请求过来传⼊参数：为SQL的ID和传⼊参数对象

处理过程：

- 根据SQL的ID查找对应的MappedStatement对象。
- 根据传⼊参数对象解析MappedStatement对象，得到最终要执⾏的SQL和执行传⼊参数。
- 获取数据库连接，根据得到的最终SQL语句和执⾏传⼊参数到数据库执⾏，并得到执⾏结果。
- 根据MappedStatement对象中的结果映射配置对得到的执⾏结果进⾏转换处理，并得到最终的处理结果。
- 释放连接资源。
- 返回处理结果

将最终的处理结果返回。


