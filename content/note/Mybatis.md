# Mybatis

## 数据源

获取数据库连接的方式，与 DriverManager 获取数据库连接相比， 提供了更高一层的抽象。旨在封装数据库连接的创建过程和数据连接池的建立。

## 事务管理器

管理数据库事务，封装 管理 数据库事务特征的代码。

## Mybatis 作用

MyBatis 免除了几乎所有的 JDBC 代码以及设置参数和获取结果集的工作。

MyBatis 可以通过简单的 XML 或注解来配置和映射原始类型、接口和 Java POJO（Plain Old Java Objects，普通老式 Java 对象）为数据库中的记录.

## Mybatis 对象

### 通过读取配置创建 SqlSessionFactory

XML 配置文件中包含了对 MyBatis 系统的核心设置，包括获取数据库连接实例的数据源（DataSource）以及决定事务作用域和控制方式的事务管理器（TransactionManager）

### 通过SqlSessionFactory 创建 SqlSession

事务管理，执行语句

### 映射器的 XML 文件 以及 映射器的Java 代码 

### 参数处理器:

javaType --> Jdbc type，变量对象的属性值 --> sql 占位符 的值

### 结果集处理器: 

Jdbc type --> javaType，表的列名 --> 对象的属性值

## 如何扩展mybatis



