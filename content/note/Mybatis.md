# Mybatis

MyBatis 是一款优秀的持久层框架，它支持自定义 SQL、存储过程以及高级映射。MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录.

XML映射文件/Java类 映射文件 ； 

映射语句： 使用mapper 元素的 id 关联SQL 语句.

映射器是一些绑定映射语句的接口。

# 使用手册

XML配置 https://mybatis.org/mybatis-3/zh_CN/sqlmap-xml.html

Java API解读 https://mybatis.org/mybatis-3/zh_CN/java-api.html

动态SQL : if , choose,foreach ,where 

# SqlSessionFactory

指定生成的SqlSession的特性

1. 是否开启事务处理，事务隔离级别
2. 从何处获取数据库连接
3. 语句执行器的类型【为每个语句的执行创建一个新的预处理语句；会复用预处理语句；批量执行所有更新语句，如果 SELECT 在多个更新中间执行，将在必要时将多条更新语句分隔开来，以方便理解】


# SqlSession

它包含了所有执行语句、提交或回滚事务以及获取映射器实例的方法。

