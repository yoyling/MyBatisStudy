# MyBatisStudy

### MyBatis 3 学习记录4阶段

--------------------------------------------------------

**（1）mybatis入门**

1. mybatis的概述
2. mybatis的环境搭建
3. mybatis入门案例
4. 自定义mybatis框架（主要的目的是为了让大家了解mybatis中执行细节） 

---------------------------------------------------------------

**（2）mybatis基本使用**

1. mybatis的单表crud操作
2. mybatis的参数和返回值
3. mybatis的dao编写
4. mybatis配置的细节
5. 几个标签的使用

**（3）mybatis的深入和多表**

1. mybatis的连接池

2. mybatis的事务控制及设计的方法

3. mybatis的多表查询

   一对多-----多对一------多对多

**（4）mybatis的缓存和注解开发**

1. mybatis中的加载时机（查询的时机）

2. mybatis中的一级缓存和二级缓存

3. mybatis的注解开发

   单表CRUD------多表查询

### 什么是框架？

它是我们软件开发中的一套解决方案，不同的框架解决的是不同的问题。

使用框架的好处：框架封装了很多的细节，使开发者可以使用极简的方式实现功能。大大提高开发效率。

### 三层架构

表现层：是用于展示数据的

业务层：是处理业务需求

持久层：是和数据库交互的

### 持久层技术解决方案

1. JDBC技术：**Connection**、**PreparedStatement**、**ResultSet**

2. Spring的**JdbcTemplate**：Spring中对jdbc的简单封装

3. Apache的**DBUtils**：它和Spring的JdbcTemplate很像，也是对Jdbc的简单封装

   以上这些都不是框架，JDBC是规范，Spring的JdbcTemplate和Apache的DBUtils都只是工具类

### mybatis的概述

mybatis是一个持久层框架，用java编写的。

它封装了jdbc操作的很多细节，使开发者只需要关注sql语句本身，而无需关注注册驱动，创建连接等繁杂过程。

它使用了ORM思想实现了结果集的封装。

**ORM：Object Relational Mappging 对象关系映射**

简单的说：就是把数据库表和实体类及实体类的属性对应起来，让我们可以操作实体类就实现操作数据库表。



## 01_01mybatis入门

**建库建表：mybatistest.sql**

```SQL
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user` (
  `id` int(11) NOT NULL auto_increment,
  `username` varchar(32) NOT NULL COMMENT '用户名称',
  `birthday` datetime default NULL COMMENT '生日',
  `sex` char(1) default NULL COMMENT '性别',
  `address` varchar(256) default NULL COMMENT '地址',
  PRIMARY KEY  (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user`(`id`,`username`,`birthday`,`sex`,`address`) 
values 
(41,'老王','2018-02-27 17:47:08','男','北京'),
(42,'小二王','2018-03-02 15:09:37','女','福建'),
(43,'小二王','2018-03-04 11:34:34','女','厦门'),
(45,'老六','2018-03-04 12:04:06','男','西藏'),
(46,'老王','2018-03-07 17:37:26','男','新疆'),
(48,'小马宝莉','2018-03-08 11:44:00','女','湖南');
```

### mybatis的环境搭建：

1. 创建maven工程并导入坐标

2. 创建实体类和dao的接口

3. 创建Mybatis的主配置文件

   SqlMapConifg.xml

4. 创建映射配置文件

   UserDao.xml

### 环境搭建的注意事项：

1. 创建UserDao.xml 和 UserDao.java时名称是为了和我们之前的知识保持一致。在Mybatis中它把持久层的操作接口名称和映射文件也叫做：Mapper

   所以：UserDao 和 UserMapper是一样的

2. 在idea中创建目录的时候，它和包是不一样的

   包在创建时：com.yoyling.dao它是三级结构

   目录在创建时：com.yoyling.dao是一级目录

3. mybatis的映射配置文件位置必须和dao接口的包结构相同

4. 映射配置文件的mapper标签namespace属性的取值必须是dao接口的全限定类名

5. 映射配置文件的操作配置（select），id属性的取值必须是dao接口的方法名

   当我们遵从了第三，四，五点之后，我们在开发中就无须再写dao的实现类。

### mybatis的入门案例：

1. 读取配置文件
2. 第二步：创建SqlSessionFactory工厂
3. 创建SqlSession
4. 创建Dao接口的代理对象
5. 执行dao中的方法
6. 释放资源



实体类**User.java**

接口**UserDao**，一个 **List<User> findAll()**

**测试main方法：**

```Java
//1.读取配置文件
InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
//2.创建SqlSessionFactory工厂
SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
SqlSessionFactory factory = builder.build(in);
//3.使用工厂生产SqlSession对象
SqlSession session = factory.openSession();
//4.使用SqlSession创建Dao接口的代理对象
UserDao userDao = session.getMapper(UserDao.class);
//5.使用代理对象执行方法
List<User> users = userDao.findAll();
for (User user : users) {
    System.out.println(user);
}
//6.释放资源
session.close();
in.close();
```

**SqlMapConfig.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration
		PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-config.dtd">
<!-- mybatis的主配置文件 -->
<configuration>
	<!-- 配置环境 -->
	<environments default="mysql">
		<!-- 配置mysql环境 -->
		<environment id="mysql">
			<!-- 配置事务的类型 -->
			<transactionManager type="JDBC"/>
			<!-- 配置数据源（连接池） -->
			<dataSource type="POOLED">
				<!-- 配置连接数据库的4个基本信息的 -->
				<property name="driver" value="com.mysql.jdbc.Driver"/>
				<property name="url" value="jdbc:mysql://localhost:3306/mybatistest"/>
				<property name="username" value="root"/>
				<property name="password" value="root"/>
			</dataSource>
		</environment>
	</environments>

	<!-- 指定映射配置文件的位置，映射配置文件指的是每个dao独立的配置文件 -->
	<mappers>
		<mapper resource="com/yoyling/dao/UserDao.xml"/>
	</mappers>
</configuration>
```

**UserDao.xml**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
		PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yoyling.dao.UserDao">
	<select id="findAll" resultType="com.yoyling.domain.User">
		select * from user;
	</select>
</mapper>
```



### 注意事项：

不要忘记在映射配置中告知mybatis要封装到哪个实体类中

配置的方式：指定实体类的全限定类名

## 01_02mybatis_annotation

**mybatis基于注解的入门案例：**

把UserDao.xml移除，在dao接口的方法上使用@Select注解，并且指定SQL语句。

同时需要在SqlMapConfig.xml中的mapper配置时，使用class属性指定dao接口的全限定类名。

```java
public interface UserDao {
    @Select("select * from user")
    List<User> findAll();
}
```

```xml
<!-- SqlMapConfig.xml中的<configuration>标签中的<mappers>需要改动 -->
<mappers>
   <mapper class="com.yoyling.dao.UserDao"/>
</mappers>
```

**明确：**

我们在实际开发中，都是越简便越好，所以都是采用不写dao实现类的方式。

不管使用XML还是注解配置。

但是Mybatis它是支持写dao实现类的。

## 01_03mybatis_dao

Mybatis的dao实现类

## 01_04mybatis_design

**自定义Mybatis的分析：**

mybatis在使用代理dao的方式实现增删改查时做什么事呢？

1. 创建代理对象
2. 在代理对象中调用selectList

**自定义mybatis能通过入门案例看到类：**

- class Resources
- class SqlSessionFactoryBuilder
- interface SqlSessionFactory
- interface SqlSession

## 02_01mybatisCRUD

UserDao.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
		PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
		"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.yoyling.dao.UserDao">

	<!-- 配置查询结果的列名和实体类的属性名的对应关系 -->
	<resultMap id="userMap" type="user">
		<!-- 主键字段对应 -->
		<id property="userId" column="id"/>
		<!-- 非主键字段对应 -->
		<result property="userName" column="username"/>
		<result property="userAddress" column="address"/>
		<result property="userSex" column="sex"/>
		<result property="userBirthday" column="birthday"/>
	</resultMap>

	<!-- 查询所有 -->
	<select id="findAll" resultMap="userMap">
		<!--select id as userId,username as userName,address as userAddress,sex as userSex,birthday as userBirthday from user;-->
		select * from user;
	</select>

	<!-- 保存用户 -->
	<insert id="saveUser" parameterType="user">
		<!-- 配置插入操作后，获取插入数据的id -->
		<selectKey keyProperty="userId" keyColumn="id" resultType="int" order="AFTER">
			select last_insert_id();
		</selectKey>
		insert into user(username,address,sex,birthday)values(#{userName},#{userAddress},#{userSex},#{userBirthday});
	</insert>

	<!-- 更新用户 -->
	<update id="updateUser" parameterType="user">
		update user set username=#{userName},address=#{userAddress},sex=#{userSex},birthday=#{userBirthday} where id=#{userId};
	</update>

	<!-- 删除用户 -->
	<delete id="deleteUser" parameterType="int">
		delete from user where id = #{uid}
	</delete>

	<!-- 根据id查询用户 -->
	<select id="findById" parameterType="int" resultMap="userMap">
		select * from user where id = #{uid}
	</select>

	<!-- 根据名称模糊查询 -->
	<select id="findByName" parameterType="String" resultMap="userMap">
 		select * from user where username like #{username}
		<!-- select * from user where username like '%${value}%' -->
	</select>

	<!-- 查询总用户数 -->
	<select id="findTotal" resultType="int">
		select count(id) from user;
	</select>

	<!-- 根据queryVo的条件查询用户 -->
	<select id="findUserByVo" parameterType="com.yoyling.domain.QueryVo" resultMap="userMap">
		select * from user where username like #{user.userName}
	</select>
</mapper>
```



MybatisTest.java

```java
package com.yoyling.test;

/**
 * 测试mybatis的crud操作
 */
public class MybatisTest {

    private InputStream in;
    private SqlSession sqlSession;
    private UserDao userDao;

    @Before     //用于在测试方法执行之前执行
    public void init() throws Exception {
        in = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        sqlSession = factory.openSession();
        userDao = sqlSession.getMapper(UserDao.class);
    }

    @After     //用于在测试方法执行之后执行
    public void destroy() throws Exception {
        //提交事务
        sqlSession.commit();

        sqlSession.close();
        in.close();
    }

    @Test
    public void testFindAll(){
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
    }

    /**
     * 测试保存操作
     */
    @Test
    public void testSave() {
        User user = new User();
        user.setUserName("yoyling 最后插入");
        user.setUserAddress("福建省厦门市");
        user.setUserSex("男");
        user.setUserBirthday(new Date());

        System.out.println("保存操作前：" + user);
        userDao.saveUser(user);
        System.out.println("保存操作后：" + user);
    }

    /**
     * 测试更新操作
     */
    @Test
    public void testUpdate() {
        User user = new User();
        user.setUserId(50);
        user.setUserName("mybatis");
        user.setUserAddress("福建省漳州市");
        user.setUserSex("女");
        user.setUserBirthday(new Date());

        userDao.updateUser(user);
    }

    /**
     * 测试删除操作
     */
    @Test
    public void testDelete() {
        userDao.deleteUser(50);
    }

    /**
     * 测试查询一个操作
     */
    @Test
    public void testFindOne() {
        User user = userDao.findById(48);
        System.out.println(user);
    }

    /**
     * 测试模糊查询操作
     */
    @Test
    public void testFindByName() {
        List<User> users = userDao.findByName("%王%");
//        List<User> users = userDao.findByName("王");
        for (User user : users) {
            System.out.println(user);
        }
    }

    /**
     * 测试查询总记录条数操作
     */
    @Test
    public void testFindTotal() {
        int count = userDao.findTotal();
        System.out.println(count);
    }

    /**
     * 测试使用QueryVo作为查询条件
     */
    @Test
    public void testFindByVo() {
        QueryVo vo = new QueryVo();
        User user = new User();
        user.setUserName("%王%");
        vo.setUser(user);
        List<User> users = userDao.findUserByVo(vo);
        for (User u : users) {
            System.out.println(u);
        }
    }
}
```

**resultMap**

配置**查询结果**的列名和实体类的属性名的对应关系

```xml
<resultMap id="userMap" type="user">
   <!-- 主键字段对应 -->
   <id property="userId" column="id"/>
   <!-- 非主键字段对应 -->
   <result property="userName" column="username"/>
   <result property="userAddress" column="address"/>
   <result property="userSex" column="sex"/>
   <result property="userBirthday" column="birthday"/>
</resultMap>
```

```xml
<!-- 查询所有 -->
<select id="findAll" resultMap="userMap">
```



**SqlMapConfig.xml中的<typeAliases>**

**typeAlias**用于配置别名。**type**属性指定的是实体类全限定类名。**alias**属性指定别名，当指定了别名就不再区分大小写。

```xml
<typeAliases>
   <typeAlias type="com.yoyling.domain.User" alias="user"/>
</typeAliases>
```

用于指定要配置别名的包，当指定之后该包下的实体类都会注册别名，并且类名就是别名，不再区分大小写。

```xml
<typeAliases>
   <package name="com.yoyling.domain"/>
</typeAliases>
```

```xml
<!-- 保存用户中的parameterType则可以直接写不区分大小写的类名 -->
<insert id="saveUser" parameterType="user">
```



**SqlMapConfig.xml中把jdbc连接信息提取到外面的properties文件**

```xml
<properties resource="jdbcConfig.properties"/>
<environments default="mysql">
	<environment id="mysql">
		<transactionManager type="JDBC"/>
		<dataSource type="POOLED">
			<property name="driver" value="${jdbc.driver}"/>
			<property name="url" value="${jdbc.url}"/>
			<property name="username" value="${jdbc.username}"/>
			<property name="password" value="${jdbc.password}"/>
		</dataSource>
	</environment>
</environments>
```

**jdbcConfig.properties**

```properties
jdbc.driver = com.mysql.jdbc.Driver
jdbc.url = jdbc:mysql://localhost:3306/mybatistest
jdbc.username = root
jdbc.password = root
```

## 02_02mybatisDAO

mybatis也可以自己编写Dao实现或者使用代理Dao实现。

**UserDaoImpl.java**

```java
package com.yoyling.dao.impl;

public class UserDaoImpl implements UserDao {

    private SqlSessionFactory factory;

    public UserDaoImpl(SqlSessionFactory factory) {
        this.factory = factory;
    }

    @Override
    public List<User> findAll() {
        //1.根据factory获取SqlSession对象
        SqlSession session = factory.openSession();
        //2.调用SqlSession中的方法，实现查询列表
        List<User> users = session.selectList("com.yoyling.dao.UserDao.findAll");//参数就是能获取配置信息的key
        //3.释放资源
        session.close();
        return users;
    }

    @Override
    public void saveUser(User user) {
        //1.根据factory获取SqlSession对象
        SqlSession session = factory.openSession();
        //2.调用方法实现保存
        session.insert("com.yoyling.dao.UserDao.saveUser",user);
        //3.提交事务
        session.commit();
        //4.释放资源
        session.close();
    }

    @Override
    public void updateUser(User user) {
        //1.根据factory获取SqlSession对象
        SqlSession session = factory.openSession();
        //2.调用方法实现更新
        session.update("com.yoyling.dao.UserDao.updateUser",user);
        //3.提交事务
        session.commit();
        //4.释放资源
        session.close();
    }
}
```

**MybatisTest.java**

```java
package com.yoyling.test;

/**
 * 测试mybatis的crud操作
 */
public class MybatisTest {

    private InputStream in;
    private UserDao userDao;

    @Before     //用于在测试方法执行之前执行
    public void init() throws Exception {
        in = Resources.getResourceAsStream("sqlMapConfig.xml");
        SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(in);
        userDao = new UserDaoImpl(factory);
    }

    @After     //用于在测试方法执行之后执行
    public void destroy() throws Exception {
        in.close();
    }

    @Test
    public void testFindAll(){
        List<User> users = userDao.findAll();
        for (User user : users) {
            System.out.println(user);
        }
    }

    /**
     * 测试保存操作
     */
    @Test
    public void testSave() {
        User user = new User();
        user.setUsername("yoyling 最后插入");
        user.setAddress("福建省厦门市");
        user.setSex("男");
        user.setBirthday(new Date());

        System.out.println("保存操作前：" + user);
        userDao.saveUser(user);
        System.out.println("保存操作后：" + user);
    }

    /**
     * 测试更新操作
     */
    @Test
    public void testUpdate() {
        User user = new User();
        user.setId(49);
        user.setUsername("mybatis");
        user.setAddress("福建省漳州市");
        user.setSex("女");
        user.setBirthday(new Date());

        userDao.updateUser(user);
    }
}
```

## 03_01datasourceAndTx

**连接池：**

* 我们在实际开发中都会使用连接池。
* 因为它可以减少我们获取连接所消耗的时间。

**mybatis中的连接池：**

主配置文件SqlMapConfig.xml中的dataSource标签，type属性就是表示采用何种连接池方式。



**type属性的取值：**

**POOLED ** 采用传统的javax.sql.DataSource规范中的连接池，mybatis中有针对规范的实现。

**UNPOOLED**  采用传统的获取连接的方式，虽然也实现Javax.sql.DataSource接口，但是并没有使用池的思想。

**JNDI ** 采用服务器提供的JNDI技术实现，来获取DataSource对象，不同的服务器所能拿到DataSource是不一样。



**mybatis中的事务**

什么是事务、事务的四大特性ACID、不考虑隔离性会产生的3个问题、解决办法：四种隔离级别

它是通过sqlsession对象的commit方法和rollback方法实现事务的提交和回滚

## 03_02dynamicSQL

MyBatis动态SQL（select查询）常用标签： <where><if><foreach>

```xml
<!-- 了解的内容:抽取重复的sql语句 -->
<sql id="defaultUser">
   select * from user
</sql>

<!-- 查询所有 -->
<select id="findAll" resultMap="userMap">
   <include refid="defaultUser"/>
</select>

<!-- 根据条件查询
<select id="findUserByCondition" resultMap="userMap" parameterType="user">
   select * from user where 1 = 1
   <if test="userName != null">
      and username = #{userName}
   </if>
   <if test="userSex != null">
      and sex = #{userSex}
   </if>
</select> -->
<select id="findUserByCondition" resultMap="userMap" parameterType="user">
   <include refid="defaultUser"/>
   <where>
      <if test="userName != null">
         and username = #{userName}
      </if>
      <if test="userSex != null">
         and sex = #{userSex}
      </if>
   </where>
</select>

<!-- 根据queryvo中提供的id集合查询用户列表 -->
<select id="findUserInIds" resultMap="userMap" parameterType="queryvo">
   <include refid="defaultUser"/>
   <where>
      <if test="ids != null and ids.size()>0">
         <foreach collection="ids" open="and id in (" close=")" item="id" separator=",">
            #{id}
         </foreach>
      </if>
   </where>
</select>
```



## 03_03one2many

多表关系操作，添加账户 **account** 表：

```SQL
DROP TABLE IF EXISTS `account`;

CREATE TABLE `account` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `UID` int(11) default NULL COMMENT '用户编号',
  `MONEY` double default NULL COMMENT '金额',
  PRIMARY KEY  (`ID`),
  KEY `FK_Reference_8` (`UID`),
  CONSTRAINT `FK_Reference_8` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `account`(`ID`,`UID`,`MONEY`) values (1,46,1000),(2,45,1000),(3,46,2000);
```

**MyBatis中的多表查询举例：**

* **一对多(多对一)： **订单和用户

* **一对一 ：** 一人一个身份证号

* **多对多： **老师和学生

以下代码示例：**用户**和**账户**

* 一个用户有多个账户

* 一个账户只能属于一个用户

1.建立两表，用户表、账户表

* 之间一对多关系用外键关联

2.建立两实体类，用户类、账户类

* 让实体类体现出一对多关系

3.建立两配置文件

4.实现配置

* 当我们查询用户时，可以同时得到用户下所包含的账户信息
* 当我们查询账户时，可以同时得到账户的所属用户信息

### 一对一：

**第一种关系体现方式：新建AccountUser类**，继承Account，然后加入用户名和地址的属性；在toString()加上super.toString

```Java
public class AccountUser extends Account {

    private String username;
    private String address;

    public String getUsername() {return username;}

    public void setUsername(String username) {this.username = username;}

    public String getAddress() {return address;}

    public void setAddress(String address) {this.address = address;}

    @Override
    public String toString() {
        return super.toString()+"     AccountUser{" +
                "username='" + username + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}

<!-- 查询所有账户并且包含用户名和地址信息 -->
<select id="findAllAccount" resultType="accountuser">
   select a.*,u.username,u.address from account a,user u where u.id = a.uid;
</select>
```

**第二种关系体现方式：从表实体包含主表实体的对象引用**

Account.java

```Java
public class Account implements Serializable {
    private Integer id;
    private Integer uid;
    private double money;

    //从表实体应该包含一个主表实体的对象引用
    private User user;

    public User getUser() {return user;}
    public void setUser(User user) {this.user = user;}
    
    public Integer getId() {return id;}
    public void setId(Integer id) {this.id = id;}
    public Integer getUid() {return uid;}
    public void setUid(Integer uid) {this.uid = uid;}
    public double getMoney() {return money;}
    public void setMoney(double money) {this.money = money;}

    @Override
    public String toString() {
        return "Account{" +
                "id=" + id +
                ", uid=" + uid +
                ", money=" + money +
                '}';
    }
}
```

AccountDao.xml

```xml
<!-- 定义封装account和user的resultMap -->
<resultMap id="accountUserMap" type="account">
   <id property="id" column="aid"/>
   <result property="uid" column="uid"/>
   <result property="money" column="money"/>
   <!-- 一对一的关系映射：配置封装user的内容 -->
   <association property="user" column="uid" javaType="user">
      <id    property="id" column="id"></id>
      <result column="username" property="username"></result>
      <result column="address" property="address"></result>
      <result column="sex" property="sex"></result>
      <result column="birthday" property="birthday"></result>
   </association>
</resultMap>

<!-- 查询所有 -->
<select id="findAll" resultMap="accountUserMap">
   select u.*,a.id as aid,a.uid,a.money from account a,user u where u.id = a.uid;
</select>
```

AccountTest.java

```java
@Test
public void testFindAll(){
    List<Account> users = accountDao.findAll();
    for (Account account : users) {
        System.out.println(account);
        System.out.println(account.getUser());
    }
}
```

### 一对多：

User.java

```Java
public class User implements Serializable {
    private Integer id;
    private String username;
    private String sex;
    private String address;
    private Date birthday;

    //一对多关系映射：主表实体应该包含从表实体的集合引用
    private List<Account> accounts;

    public List<Account> getAccounts() {
        return accounts;
    }

    public void setAccounts(List<Account> accounts) {
        this.accounts = accounts;
    }
    ......
}
```

UserDao.xml

```xml
<!-- 定义User的resultMap -->
<resultMap id="userAccountMap" type="user">
   <id    property="id" column="id"/>
   <result property="username" column="username"/>
   <result property="address" column="address"/>
   <result property="sex" column="sex"/>
   <result property="birthday" column="birthday"/>
   <!-- 配置user对象中accounts集合的映射 -->
   <collection property="accounts" ofType="account">
      <id    column="aid" property="id"/>
      <result column="uid" property="uid"/>
      <result column="money" property="money"/>
   </collection>
</resultMap>
<!-- 查询所有 -->
<select id="findAll" resultMap="userAccountMap">
   select * from user u left outer join account a on u.id = a.uid
</select>
```

UserTest.java

```Java
@Test
public void testFindAll(){
    List<User> users = userDao.findAll();
    for (User user : users) {
        System.out.println(user);
        System.out.println(user.getAccounts());
    }
}
```

## 03_04many2many

### 多对多：

示例：用户和角色

一个用户多个角色，一个角色可赋予多个用户

使用中间表，包含各自主键。在中间表为外键。两实体类各自包含对方一个集合引用。

数据库新建两个表 **role** 和 **user_role**，为角色表，和用户角色中间表。

```sql
DROP TABLE IF EXISTS `role`;

CREATE TABLE `role` (
  `ID` int(11) NOT NULL COMMENT '编号',
  `ROLE_NAME` varchar(30) default NULL COMMENT '角色名称',
  `ROLE_DESC` varchar(60) default NULL COMMENT '角色描述',
  PRIMARY KEY  (`ID`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `role`(`ID`,`ROLE_NAME`,`ROLE_DESC`) values (1,'院长','管理整个学院'),(2,'总裁','管理整个公司'),(3,'校长','管理整个学校');



DROP TABLE IF EXISTS `user_role`;

CREATE TABLE `user_role` (
  `UID` int(11) NOT NULL COMMENT '用户编号',
  `RID` int(11) NOT NULL COMMENT '角色编号',
  PRIMARY KEY  (`UID`,`RID`),
  KEY `FK_Reference_10` (`RID`),
  CONSTRAINT `FK_Reference_10` FOREIGN KEY (`RID`) REFERENCES `role` (`ID`),
  CONSTRAINT `FK_Reference_9` FOREIGN KEY (`UID`) REFERENCES `user` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert  into `user_role`(`UID`,`RID`) values (41,1),(45,1),(41,2);
```

Role.java

```java
private Integer roleId;
private String roleName;
private String roleDesc;

//多对多的关系映射：一个角色可以赋予给多个用户
private List<User> users;

public List<User> getUsers() {return users;}

public void setUsers(List<User> users) {this.users = users;}
```

RoleDao.xml

```xml
<!-- 定义role表的ResultMap -->
<resultMap id="roleMap" type="role">
   <id property="roleId" column="id"/>
   <result property="roleName" column="role_name"/>
   <result property="roleDesc" column="role_desc"/>
   <collection property="users" ofType="user">
      <id column="id" property="id"/>
      <result column="username" property="username"/>
      <result column="address" property="address"/>
      <result column="sex" property="sex"/>
      <result column="birthday" property="birthday"/>
   </collection>
</resultMap>
<!-- 查询所有 -->
<select id="findAll" resultMap="roleMap">
   select u.*,r.id as rid,r.role_name,r.role_desc from role r
   left outer join user_role ur on r.id = ur.rid
   left outer join user u on u.id = ur.uid
</select>
```

得到输出结果：

```shell
Role{roleId=41, roleName='院长', roleDesc='管理整个学院'}
[User{id=41, username='老王', sex='男', address='北京', birthday=Tue Feb 27 17:47:08 CST 2018}]
Role{roleId=45, roleName='院长', roleDesc='管理整个学院'}
[User{id=45, username='老六', sex='男', address='西藏', birthday=Sun Mar 04 12:04:06 CST 2018}]
Role{roleId=null, roleName='校长', roleDesc='管理整个学校'}
```

------

User.java

```java
private Integer id;
private String username;
private String sex;
private String address;
private Date birthday;

//多对多的关系映射：一个用户可以具备多个角色
private List<Role> roles;

public List<Role> getRoles() {return roles;}

public void setRoles(List<Role> roles) {this.roles = roles;}
```

UserDao.xml

```xml
<!-- 定义User的resultMap -->
<resultMap id="userAccountMap" type="user">
   <id    property="id" column="id"/>
   <result property="username" column="username"/>
   <result property="address" column="address"/>
   <result property="sex" column="sex"/>
   <result property="birthday" column="birthday"/>
   <collection property="roles" ofType="role">
      <id property="roleId" column="rid"/>
      <result property="roleName" column="role_name"/>
      <result property="roleDesc" column="role_desc"/>
   </collection>
</resultMap>

<!-- 查询所有 -->
<select id="findAll" resultMap="userAccountMap">
   select u.*,r.id as rid,r.role_name,r.role_desc from user u
       left outer join user_role ur on u.id = ur.uid
       left outer join role r on r.id = ur.rid
</select>
```

测试得到输出：

```Java
@Test
public void testFindAll(){
    List<User> users = userDao.findAll();
    for (User user : users) {
        System.out.println(user);
        System.out.println(user.getRoles());
    }
}
```

```shell
User{id=41, username='老王', sex='男', address='北京', birthday=Tue Feb 27 17:47:08 CST 2018}
[Role{roleId=1, roleName='院长', roleDesc='管理整个学院'}, Role{roleId=2, roleName='总裁', roleDesc='管理整个公司'}]
User{id=42, username='小二王', sex='女', address='福建', birthday=Fri Mar 02 15:09:37 CST 2018}
[]
User{id=43, username='小二王', sex='女', address='厦门', birthday=Sun Mar 04 11:34:34 CST 2018}
[]
User{id=45, username='老六', sex='男', address='西藏', birthday=Sun Mar 04 12:04:06 CST 2018}
[Role{roleId=1, roleName='院长', roleDesc='管理整个学院'}]
User{id=46, username='老王', sex='女', address='新疆', birthday=Wed Mar 07 17:37:26 CST 2018}
[]
User{id=48, username='小马宝莉', sex='女', address='湖南', birthday=Thu Mar 08 11:44:00 CST 2018}
[]
```

## 03_05jndi

SqlMapConfig.xml

```xml
<environments default="mysql">
   <environment id="mysql">
      <transactionManager type="JDBC"/>
      <dataSource type="JNDI">
         <property name="data_source" value="java:comp/env/jdbc/mybatistest"/>
      </dataSource>
   </environment>
</environments>
```

META-INF -> context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Context>
	<Resource 
		name="jdbc/mybatistest"
		type="javax.sql.DataSource"
		auth="Container"
		maxActive="20"
		maxWait="10000"
		maxIdle="5"
		username="root"
		password="root"
		driverClassName="com.mysql.jdbc.Driver"
		url="jdbc:mysql://localhost:3306/mybatistest"
	/>
</Context>
```

index.jsp

```java
<%
   //1.读取配置文件
   InputStream in = Resources.getResourceAsStream("SqlMapConfig.xml");
   //2.创建SqlSessionFactory工厂
   SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
   SqlSessionFactory factory = builder.build(in);
   //3.使用工厂生产SqlSession对象
   SqlSession sqlSession = factory.openSession();
   //4.使用SqlSession创建Dao接口的代理对象
   UserDao userDao = sqlSession.getMapper(UserDao.class);
   //5.使用代理对象执行方法
   List<User> users = userDao.findAll();
   for (User user : users) {
      System.out.println(user);
   }
   //6.释放资源
   sqlSession.close();
   in.close();
%>
```

## 04_01lazy

**延迟加载：**

在真正使用数据时才发起查询，不用时不查询。按需加载（懒加载）。

**立即加载：**

不管用不用只要一调用方法，马上发起查询。

**一对多，多对多：通常用延迟加载。**

**多对一，一对一：通常用立即加载。**

用的时候调用对方配置文件中的一个配置来实现延迟查询功能。

AccountDao.xml

```xml
<!-- 定义封装account和user的resultMap -->
<resultMap id="accountUserMap" type="account">
   <id property="id" column="id"/>
   <result property="uid" column="uid"/>
   <result property="money" column="money"/>
   <!-- 一对一的关系映射：配置封装user的内容
   select属性指定的内容：查询用户的唯一标识。
   column属性指定的内容：用户根据id查询时所需的参数值。
   -->
   <association property="user" column="uid" javaType="user" select="com.yoyling.dao.UserDao.findById"/>
</resultMap>
<!-- 查询所有 -->
<select id="findAll" resultMap="accountUserMap">
   select * from account;
</select>
```

SqlMapConfig.xml

```xml
<!-- 配置参数 -->
<settings>
   <!-- 开启Mybatis支持延迟加载 -->
   <setting name="lazyLoadingEnabled" value="true"/>
   <setting name="aggressiveLazyLoading" value="false"/>
</settings>
```

## 04_02cache

**缓存：**减少和数据库的交互次数，提高执行效率。

经常查询并且不经常改变、数据的正确与否对最终结果影响不大的数据。

**一级缓存：**

SqlSession对象的缓存。当我们执行查询时，查询结果会同时存入到SqlSession提供的一区域，结构为map。

当调用SqlSession的修改，添加，删除，commit()，close()等方法时，就会清空一级缓存。

```java
@Test
public void testFirstLevelCache(){
    User user1 = userDao.findById(41);
    System.out.println(user1);
    User user2 = userDao.findById(41);
    System.out.println(user2);
    
    System.out.println(user1==user2);
}
```

```java
@Test
public void testFirstLevelCache(){
    User user1 = userDao.findById(41);
    System.out.println(user1);
    
    //再次获取SqlSession对象
    //sqlSession.close();
    //sqlSession = factory.openSession();
    sqlSession.clearCache();
    
    userDao = sqlSession.getMapper(UserDao.class);
    
    User user2 = userDao.findById(41);
    System.out.println(user2);
    System.out.println(user1==user2);
}
```

**二级缓存：**

Mybatis中SqlSessionFactory对象的缓存，由同一个SqlSessionFactory对象创建的SqlSession共享其缓存。

二级缓存存放的内容是数据不是对象，因此前后不是同一个对象。

**步骤：**

1. 让Mybatis框架支持二级缓存（在SqlMapConfig.xml中配置）
2. 让当前的映射文件支持二级缓存（在UserDao.xml中配置）
3. 让当前的操作支持二级缓存（在select标签中配置）

SqlMapConfig.xml

```xml
<settings>
   <setting name="cacheEnabled" value="true"/>
</settings>
```

UserDao.xml

```xml
<!-- 开启user支持二级缓存 -->
<cache/>

<!-- 根据id查询用户 -->
<select id="findById" parameterType="int" resultType="user" useCache="true">
   select * from user where id = #{uid}
</select>
```

## 04_03annotation_mybatis

UserDao.java

```java
package com.yoyling.dao;

/**
 * CRUD四个注解: @Select @Insert @Update @Delete
 */
public interface UserDao {

	/**
	 * 查询所有用户
	 * @return
	 */
	@Select("select * from user")
	List<User> findAll();

	/**
	 * 保存用户
	 * @param user
	 */
	@Insert("insert into user(username,address,sex,birthday)values(#{username},#{address},#{sex},#{birthday})")
	void saveUser(User user);

	/**
	 * 更新用户
	 * @param user
	 */
	@Update("update user set username = #{username},address = #{address},sex = #{sex},birthday = #{birthday} where id = #{id}")
	void updateUser(User user);

	/**
	 * 删除用户
	 * @param userId
	 */
	@Delete("delete from user where id = #{id}")
	void deleteUser(Integer userId);

	/**
	 * 根据id查询用户
	 * @param userId
	 * @return
	 */
	@Select("select * from user where id = #{id}")
	User findById(Integer userId);

	/**
	 * 根据用户名称模糊查询
	 * @param userName
	 * @return
	 */
	//@Select("select * from user where username like #{username}")
	@Select("select * from user where username like '%${value}%'")
	List<User> findUserByName(String userName);

	/**
	 * 查询总用户数
	 * @return
	 */
	@Select("select count(*) from user")
	int findTotalUser();
}
```

## 04_04annoOne2Many

**实体属性和数据库表列名的对应：**

UserDao.java

```java
/**
 * 查询所有用户
 * @return
 */
@Select("select * from user")
@Results(id="userMap",value={
      @Result(id = true,column = "id",property = "userId"),
      @Result(column = "username",property = "userName"),
      @Result(column = "address",property = "userAddress"),
      @Result(column = "sex",property = "userSex"),
      @Result(column = "birthday",property = "userBirthday")
})
List<User> findAll();
/**
 * 根据id查询用户
 * @param userId
 * @return
 */
@Select("select * from user where id = #{id}")
@ResultMap("userMap")
User findById(Integer userId);
```

Account.java

```java
private Integer id;
private Integer uid;
private Double money;

//多对一（Mybatis中称为一对一）的映射：一个账户只能属于一个用户
private User user;

public User getUser() {return user;}

public void setUser(User user) {this.user = user;}
```

AccoundDao.java

```java
/**
 * 查询所有账户，并且获取每个账户所属的用户信息
 * @return
 */
@Select("select * from account")
@Results(id = "accountMap",value = {
      @Result(id = true,column = "id",property = "id"),
      @Result(column = "uid",property = "uid"),
      @Result(column = "money",property = "money"),
      @Result(property = "user",column = "uid",one = @One(select = "com.yoyling.dao.UserDao.findById",fetchType = FetchType.EAGER))
})
List<Account> findAll();
```

**一对多：**

User.java

```java
//一对多关系映射：一个用户对应多个账户
private List<Account> accounts;
```

AccountDao.java

```java
/**
 * 根据用户id查询账户信息
 * @param userId
 * @return
 */
@Select("select * from account where uid = #{userId}")
List<Account> findAccountByUid(Integer userId);
```

UserDao.java

```java
/**
 * 查询所有用户
 * @return
 */
@Select("select * from user")
@Results(id="userMap",value={
      @Result(id = true,column = "id",property = "userId"),
      @Result(column = "username",property = "userName"),
      @Result(column = "address",property = "userAddress"),
      @Result(column = "sex",property = "userSex"),
      @Result(column = "birthday",property = "userBirthday"),
      @Result(property = "accounts",column = "id",many = @Many(select = "com.yoyling.dao.AccountDao.findAccountByUid",fetchType = FetchType.LAZY))

})
List<User> findAll();
```

**注解开启二级缓存：**

UserDao.java

```java
@CacheNamespace(blocking = true)
public interface UserDao {...}
```

SqlMapConfig.xml

```xml
<!-- 配置开启二级缓存 -->
<settings>
   <setting name="cacheEnabled" value="true"/>
</settings>
```

SecondLevelCacheTest.java

```java
@Test
public void testFindOne() {
   SqlSession session = factory.openSession();
   UserDao userDao = session.getMapper(UserDao.class);
   User user = userDao.findById(41);
   System.out.println(user);
   session.close();//释放一级缓存

   SqlSession session1 = factory.openSession();//再次打开session
   UserDao userDao1 = session1.getMapper(UserDao.class);
   User user1 = userDao1.findById(41);
   System.out.println(user1);
   session1.close();
}
```
