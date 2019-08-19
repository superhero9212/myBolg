# Mybatis2-课堂笔记

## 一、Mybatis的参数和结果集

### 2. parameterType

#### 2.1 简单类型

- 参数写法：

​	例如：int, double, short 等基本数据类型，或者string

​	或者：java.lang.Integer, java.lang.Double, java.lang.Short,   java.lang.String

- SQL语句里获取参数：

  如果是一个简单类型参数，写法是：`#{随意}`

#### 2.2 POJO（JavaBean）

- 参数写法：

​	如果parameterType是POJO类型，在SQL语句中，可以在`#{}`或者`${}`中使用OGNL表达式来获取JavaBean的属性值。

​	例如：parameterType是com.itheima.domain.User， 在SQL语句中可以使用`#{username}`获取username属性值。

- SQL语句里获取参数：`#{xxx}`

#### 2.3 POJO包装类（复杂JavaBean）--QueryVO

- 参数写法：

​	在web应用开发中，通常有综合条件的搜索功能，例如：根据商品名称 和 所属分类 同时进行搜索。这时候通常是把搜索条件封装成JavaBean对象；JavaBean中可能还有JavaBean。

- SQL里取参数：`#{xxx.xx.xx}`

##### 2.3.1  功能需求

​	根据用户名搜索用户信息，查询条件放到QueryVO的user属性中。QueryVO如下：

```java
public class QueryVO {
    private User user;

    public User getUser() {
        return user;
    }

    public void setUser(User user) {
        this.user = user;
    }
}
```

##### 2.3.1 功能实现

###### 1) 在映射器UserDao中增加方法

```java
List<User> findByVO(QueryVO vo);
```

###### 2) 在映射配置文件中增加statement

```xml
<select id="findByVO" parameterType="com.itheima.domain.QueryVO"     resultType="com.itheima.domain.User">
   select * from user where username like #{user.username}
</select>
```

###### 3) 在单元测试类中编写测试代码

```java
@Test
public void testFindByVO(){
    QueryVO vo = new QueryVO();
    User user = new User();
    user.setUsername("%王%");
    vo.setUser(user);

    List<User> users = dao.findByVO(vo);
    for (User user1 : users) {
        System.out.println(user1);
    }
}
```

### 3. resultType

> 注意：resultType是查询select标签上才有的，用来设置查询的结果集要封装成什么类型的

#### 3.1 简单类型

​	例如：int, double, short 等基本数据类型，或者string

​	或者：java.lang.Integer, java.lang.Double, java.lang.Short,   java.lang.String

#### 3.2 POJO（JavaBean）

​	例如：com.itheima.domain.User

> 注意：JavaBean的属性名要和字段名保持一致

#### 3.3 JavaBean中属性名和字段名不一致的情况处理

##### 3.3.1 功能需求

​	有JavaBean类User2，属性名和数据库表的字段名不同。要求查询user表的所有数据，封装成User2的集合。其中User2如下：

```java
public class User2 {
    private Integer userId;
    private String username;
    private Date userBirthday;
    private String userSex;
    private String userAddress;

    public Integer getUserId() {
        return userId;
    }

    public void setUserId(Integer userId) {
        this.userId = userId;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getUserBirthday() {
        return userBirthday;
    }

    public void setUserBirthday(Date userBirthday) {
        this.userBirthday = userBirthday;
    }

    public String getUserSex() {
        return userSex;
    }

    public void setUserSex(String userSex) {
        this.userSex = userSex;
    }

    public String getUserAddress() {
        return userAddress;
    }

    public void setUserAddress(String userAddress) {
        this.userAddress = userAddress;
    }

    @Override
    public String toString() {
        return "User2{" +
                "userId=" + userId +
                ", username='" + username + '\'' +
                ", userBirthday=" + userBirthday +
                ", userSex='" + userSex + '\'' +
                ", userAddress='" + userAddress + '\'' +
                '}';
    }
}
```

##### 3.3.2 实现方案一：SQL语句中使用别名，别名和JavaBean属性名保持一致（用的少）

###### 1) 在映射器UserDao中增加方法

```java
/**
 * JavaBean属性名和字段名不一致的情况处理---方案一
 * @return
 */
List<User2> queryAll_plan1();
```

###### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<select id="queryAll_plan1" resultType="com.itheima.domain.User2">
    select id as userId, username as username, birthday as userBirthday, address as userAddress, sex as userSex from user
</select>
```

###### 3) 在单元测试类中编写测试代码

```java
/**
 * JavaBean属性名和字段名不一致的情况处理---方案一 单元测试代码
 */
@Test
public void testQueryAllUser2_plan1(){
    List<User2> user2List = dao.queryAll_plan1();
    for (User2 user2 : user2List) {
        System.out.println(user2);
    }
}
```

##### 3.3.3 实现方案二：使用resultMap配置字段名和属性名的对应关系（推荐）

###### 1) 在映射器UserDao中增加方法

```java
/**
 * JavaBean属性名和字段名不一致的情况处理--方案二
 * @return
 */
List<User2> queryAll_plan2();
```

###### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<select id="queryAll_plan2" resultMap="user2Map">
    select * from user
</select>

<!-- 
 resultMap标签：设置结果集中字段名和JavaBean属性的对应关系
     id属性：唯一标识
   type属性：要把查询结果的数据封装成什么对象，写全限定类名 
 -->
<resultMap id="user2Map" type="com.itheima.domain.User2">
    <!--id标签：主键字段配置。  property：JavaBean的属性名；  column：字段名-->
    <id property="userId" column="id"/>
    <!--result标签：非主键字段配置。 property：JavaBean的属性名；  column：字段名-->
    <result property="username" column="username"/>
    <result property="userBirthday" column="birthday"/>
    <result property="userAddress" column="address"/>
    <result property="userSex" column="sex"/>
</resultMap>
```

###### 3) 在单元测试类中编写测试代码

```java
/**
 * JavaBean属性名和字段名不情况处理--方案二  单元测试代码
 */
@Test
public void testQueryAllUser2_plan2(){
    List<User2> user2List = dao.queryAll_plan2();
    for (User2 user2 : user2List) {
        System.out.println(user2);
    }
}
```

### 4. 小结

#### 4.1 parameterType

* 简单类型：8种基本数据类型， String。  `#{随意写}`
* POJO对象：普通的JavaBean   SQL语句里取值：`#{JavaBean的属性名}`
* POJO包装类：JavaBean嵌套JavaBean  SQL语句里取值：`#{JavaBean的属性名.属性名....}`

#### 4.2 resultType

* 简单类型：查询结果是 8种基本数据类型， String
* POJO对象：查询结果是JavaBean对象。要求：JavaBean的属性名和表的字段名要一致

#### 4.3 resultMap

* 用于解决JavaBean属性名和表字段名不一致的问题：

```xml
<select id="" resultMap="唯一标识">
</select>

<resultMap id="唯一标识" type="JavaBean的全限定类名">
	<!-- 设置主键字段的对应关系：使用id标签 -->
    <id property="javabean的属性名"  column="字段名"/>
    <!-- 设置非主键字段的对应关系：使用result标签 -->
    <result property="javabean的属性名"  column="字段名"/>
</resultMap>
```



## 二、Mybatis实现传统开发CURD（了解）

​	Mybatis中提供了两种dao层的开发方式：一是使用映射器接口代理对象的方式；二是使用映射器接口实现类的方式。其中代理对象的方式是主流，也是我们主要学习的内容。

### 1. 相关类介绍

#### 1.1 SqlSession

##### 1.1.1 SqlSession简介

​	SqlSession是一个面向用户的接口，定义了操作数据库的方法，例如：selectList, selectOne等等。

​	每个线程都应该有自己的SqlSession对象，它不能共享使用，也是**线程不安全**的。因此最佳的使用范围是在请求范围内、或者方法范围内，绝不能将SqlSession放到静态属性中。

​	SqlSession使用原则：要做到SqlSession：**随用随取，用完就关，一定要关**

##### 1.1.2 SqlSession的常用API	

​	SqlSession操作数据库的常用方法有：

| 方法                                       | 作用                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| selectList(String statement, Object param) | 查询多条数据，封装JavaBean集合                               |
| selectOne(String statement, Object param)  | 查询一条数据，封装JavaBean对象<br />查询一个数据，比如查询数量 |
| insert(String statement, Object param)     | 添加数据，返回影响行数                                       |
| update(String statement, Object param)     | 修改数据，返回影响行数                                       |
| delete(String statement, Object param)     | 删除数据，返回影响行数                                       |

> 以上方法中的参数statment，是映射配置文件中的namespace 和  id的值方法名组成的。
>
> 例如：
>
> ​	映射配置文件的namespace值为com.itheima.dao.UserDao，执行的方法名是queryAll
>
> ​	那么statement的值就是：com.itheima.dao.UserDao.queryAll

#### 1.2 SqlSessionFactory

​	是一个接口，定义了不同的openSession()方法的重载。SqlSessionFactory一旦创建后，可以重复使用，通常是以单例模式管理。

​	SqlSessionFactory使用原则：单例模式管理，一个应用中，只要有一个SqlSessionFactory对象即可。

#### 1.3 SqlSessionFactoryBuilder

​	用于构建SqlSessionFactory工厂对象的。一旦工厂对象构建完成，就不再需要SqlSessionFactoryBuilder了，通常是作为工具类使用。

​	SqlSessionFactoryBuilder：只要生产了工厂，builder对象就可以垃圾回收了

#### 1.4 API的小结

* SqlSession对象：
  * 提供了：操作数据库的不同方法
    * `selectList(statement, params)`：查询多条，得到JavaBean集合
    * `selectOne(statement,params)`：查询一条得到JavaBean对象，或者查询得到一个值
    * `insert(statement, params)`：插入数据
    * `update(statement, params)`：修改数据
    * `delete(statement, params)`：删除数据
  * 获取方式：从`SqlSessionFactory`里获取
  * 使用原则：随用随取，用完就关，一定要关
* SqlSessionFactory：
  * 提供了：获取SqlSession对象的方法：
    * `openSession()`：获取一个SqlSession对象
  * 获取方式：从`SqlSessionFactoryBuilder`里获取
  * 使用原则：通常使用单例模式进行管理
* SqlSessionFactoryBuilder：
  * 提供：构建SqlSessionFactory的方法
    * `build(InputStream is)`：构建一个SqlSessionFactory对象
  * 获取方式：`new SqlSessionFactoryBuilder()`
  * 使用原则：用完就扔，可以垃圾回收了

### 2. 需求说明

针对user表进行CURD操作，要求使用映射器接口实现类的方式实现：

- 查询全部用户，得到`List<User>`（上节课快速入门已写过，略）
- 保存用户（新增用户）
- 修改用户
- 删除用户
- 根据主键查询一个用户，得到`User`
- 模糊查询
- 查询数量

### 3. 准备Mybatis环境

#### 3.1 创建Maven的Java项目，准备JavaBean

##### 1) 创建Maven的Java项目，坐标为：

```xml
	<groupId>com.itheima</groupId>
    <artifactId>day47_mybatis02_dao</artifactId>
    <version>1.0-SNAPSHOT</version>
    <packaging>jar</packaging>
```

##### 2) 在pom.xml中添加依赖：

```xml
<dependencies>
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.12</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.46</version>
    </dependency>
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
    <dependency>
        <groupId>org.mybatis</groupId>
        <artifactId>mybatis</artifactId>
        <version>3.4.6</version>
    </dependency>
</dependencies>
```

##### 3) 创建JavaBean

```java
public class User {
    private Integer id;
    private String username;
    private Date birthday;
    private String sex;
    private String address;

    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public Date getBirthday() {
        return birthday;
    }

    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", birthday=" + birthday +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                '}';
    }
}
```

#### 3.2 准备Mybatis的映射器和配置文件

##### 1) 创建映射器接口UserDao（暂时不需要增加方法，备用）

```java
public interface UserDao{
    
}
```

##### 2) 创建映射器接口的实现类UserDaoImpl

```java
public class UserDaoImpl implements UserDao{
    private SqlSessionFactory factory;

    /**
     * 构造方法。因为工厂对象，是整个应用只要一个就足够了，所以这里不要创建SqlSessionFactory对象
     * 而是接收获取到工厂对象来使用。
     */
    public UserDaoImpl(SqlSessionFactory factory) {
        this.factory = factory;
    }
}
```

##### 3) 创建映射配置文件UserDao.xml（暂时不需要配置statement，备用）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.itheima.dao.UserDao">
    
</mapper>
```

##### 4) 创建Mybatis的核心配置文件

```xml
 
<configuration>
    <typeAliases>
        <package name="com.itheima.domain"/>
    </typeAliases>
    <environments default="mysql">
        <environment id="mysql">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="com.mysql.jdbc.Driver"/>
                <property name="url" value="jdbc:mysql:///mybatis"/>
                <property name="username" value="root"/>
                <property name="password" value="root"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/itheima/dao/UserDao.xml"/>
    </mappers>
</configuration>
```

##### 4) 准备log4j.properties日志配置文件

```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=debug, CONSOLE, LOGFILE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n

# LOGFILE is set to be a File appender using a PatternLayout.
log4j.appender.LOGFILE=org.apache.log4j.FileAppender
log4j.appender.LOGFILE.File=d:\axis.log
log4j.appender.LOGFILE.Append=true
log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
```

#### 3.3 准备单元测试类

```java
public class MybatisDaoCURDTest {
    private InputStream is;
    private SqlSessionFactory factory;
    private UserDao dao;

    @Before
    public void init() throws IOException {
        is = Resources.getResourceAsStream("SqlMapConfig.xml");
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        factory = builder.build(is);
        //创建Dao实现类对象时，把factory作为构造参数传递进去---一个应用只要有一个factory就够了
        dao = new UserDaoImpl(factory);
    }

    @After
    public void destory() throws IOException {
        is.close();
    }
}
```

### 4. 编写代码实现需求

#### 4.1 查询全部用户

##### 1) 在映射器UserDao中增加方法

```java
List<User> queryAll();
```

##### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<select id="queryAll" resultType="User">
    select * from user
</select>
```

##### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public List<User> queryAll() {
    SqlSession session = factory.openSession();
    List<User> users = session.selectList("com.itheima.dao.UserDao.queryAll");
    session.close();
    return users;
}
```

##### 4) 在单元测试类中编写测试代码

```java
@Test
public void testQueryAll(){
    List<User> users = dao.queryAll();
    for (User user : users) {
        System.out.println(user);
    }
}
```

#### 4.2 保存/新增用户

##### 1) 在映射器UserDao中增加方法

```java
void save(User user);
```

##### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<insert id="save" parameterType="User">
    <selectKey resultType="int" keyProperty="id" order="AFTER">
        select last_insert_id()
    </selectKey>
    insert into user (id, username, birthday, address, sex) 
    values (#{id}, #{username}, #{birthday},#{address},#{sex})
</insert>
```

##### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public void save(User user) {
    SqlSession session = factory.openSession();
    session.insert("com.itheima.dao.UserDao.save", user);
    session.commit();
    session.close();
}
```

##### 4) 在单元测试类中编写测试代码

```java
@Test
public void testSaveUser(){
    User user = new User();
    user.setUsername("tom");
    user.setAddress("广东深圳");
    user.setBirthday(new Date());
    user.setSex("男");

    System.out.println("保存之前：" + user);
    dao.save(user);
    System.out.println("保存之后：" + user);
}
```

#### 4.2 修改用户

##### 1) 在映射器UserDao中增加方法

```java
void edit(User user);
```

##### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<update id="edit" parameterType="User">
    update user set username = #{username}, birthday = #{birthday}, 
    address = #{address}, sex = #{sex} where id = #{id}
</update>
```

##### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public void edit(User user) {
    SqlSession session = factory.openSession();
    session.update("com.itheima.dao.UserDao.edit", user);
    session.commit();
    session.close();
}
```

##### 4) 在单元测试类中编写测试代码

```java
@Test
public void testEditUser(){
    User user = new User();
    user.setId(71);
    user.setUsername("jerry");
    user.setAddress("广东深圳宝安");
    user.setSex("女");
    user.setBirthday(new Date());

    dao.edit(user);
}
```

#### 4.3 删除用户

##### 1) 在映射器UserDao中增加方法

```java
void delete(Integer id);
```

##### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<delete id="delete" parameterType="int">
    delete from user where id = #{uid}
</delete>
```

##### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public void delete(Integer id) {
    SqlSession session = factory.openSession();
    session.delete("com.itheima.dao.UserDao.delete", id);
    session.commit();
    session.close();
}
```

##### 4) 在单元测试类中编写测试代码

```java
@Test
public void testDeleteUser(){
    dao.delete(71);
}
```

#### 4.4 根据主键查询一个用户

##### 1) 在映射器UserDao中增加方法

```java
User findById(Integer id);
```

##### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<select id="findById" parameterType="int" resultType="User">
    select * from user where id = #{id}
</select>
```

##### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public User findById(Integer id) {
    SqlSession session = factory.openSession();
    User user = session.selectOne("com.itheima.dao.UserDao.findById", id);
    session.close();
    return user;
}
```

##### 4) 在单元测试类中编写测试代码

```java
@Test
public void testFindUserById(){
    User user = dao.findById(48);
    System.out.println(user);
}
```

#### 4.5 模糊查询

##### 4.5.1 使用`#{}`方式进行模糊查询

###### 1) 在映射器UserDao中增加方法

```java
/**使用#{}方式进行模糊查询*/
List<User> findByUsername1(String username);
```

###### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<!-- 使用#{}方式进行模糊查询 -->
<select id="findByUsername1" parameterType="string" resultType="User">
    select * from user where username like #{username}
</select>
```

###### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public List<User> findByUsername1(String username) {
    SqlSession session = factory.openSession();
    List<User> users = session.selectList("com.itheima.dao.UserDao.findByUsername1", username);
    session.close();
    return users;
}
```

###### 4) 在单元测试类中编写测试代码

```java
@Test
public void testFindUserByUsername1(){
    List<User> users = dao.findByUsername1("%王%");
    for (User user : users) {
        System.out.println(user);
    }
}
```

##### 4.5.2 使用`${value}`方式进行模糊查询

###### 1) 在映射器UserDao中增加方法

```java
/**使用${value}方式进行模糊查询*/
List<User> findByUsername2(String username);
```

###### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<!-- 使用${value}方式进行模糊查询 -->
<select id="findByUsername2" parameterType="string" resultType="User">
    select * from user where username like '%${value}%'
</select>
```

###### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public List<User> findByUsername2(String username) {
    SqlSession session = factory.openSession();
    List<User> users = session.selectList("com.itheima.dao.UserDao.findByUsername2", username);
    session.close();
    return users;
}
```

###### 4) 在单元测试类中编写测试代码

```java
@Test
public void testFindUserByUsername2(){
    List<User> users = dao.findByUsername2("王");
    for (User user : users) {
        System.out.println(user);
    }
}
```

#### 4.6 查询数量

##### 1) 在映射器UserDao中增加方法

```java
Integer findTotalCount();
```

##### 2) 在映射配置文件UserDao.xml中增加statement

```xml
<select id="findTotalCount" resultType="int">
    select count(*) from user
</select>
```

##### 3) 在映射器实现类UserDaoImpl中实现方法

```java
@Override
public Integer findTotalCount() {
    SqlSession session = factory.openSession();
    Integer count = session.selectOne("com.itheima.dao.UserDao.findTotalCount");
    session.close();
    return count;
}
```

##### 4) 在单元测试类中编写测试代码

```java
@Test
public void testFindTotalCount(){
    Integer totalCount = dao.findTotalCount();
    System.out.println(totalCount);
}
```

### 5. 映射器接口代理对象方式和实现类方式运行原理

#### 5.1 映射器接口代理对象的方式  运行过程

​	debug跟踪--->实现类传统方式的sqlsession的方法--->底层是JDBC的代码

#### 5.2 映射器接口实现类的方式  运行过程

​	debug跟踪--->底层是JDBC的代码



### 6. 小结

1. 创建映射器接口，在映射器里增加方法（和之前的一样）

2. 创建映射配置文件，在配置文件里提供statement（和之前的一样）

3. 创建映射器接口的实现类：

   ```java
   class UserDaoImpl implements UserDao{
       private SqlSessionFactory factory;
       public UserDaoImpl(SqlSessionFactory factory){
           this.factory = factory;
       }
       
       public List<User> queryAll(){
           SqlSession session = factory.openSession();
           List<User> userList = session.selectList("");
           
           //如果执行的是增、删、改，在关闭SqlSession之前，要提交事务
           
           session.close();
           return userList;
       }
   }
   ```

   

## 三、SqlMapConfig.xml核心配置文件

SqlMapConfig.xml中配置的内容和顺序如下：

```text
properties（属性） ★
settings（全局配置参数） 
typeAliases（类型别名） ★ 
typeHandlers（类型处理器） 
objectFactory（对象工厂） 
plugins（插件） 
environments（环境集合属性对象） 
environment（环境子属性对象） 
transactionManager（事务管理） 
dataSource（数据源） 
mappers（映射器） ★
```

### 1. properties属性（了解）

可以在SqlMapConfig.xml中引入properties文件的配置信息，实现配置的**热插拔**效果。例如：

#### 1.1 准备jdbc.properties放在resources下

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///mybatis63
jdbc.username=root
jdbc.password=root
```

#### 1.2 在SqlMapConfig.xml中引入jdbc.properties

```xml
<configuration>
    <!-- 
 	properties标签：
		resource属性：properties资源文件的路径，从类加载路径下开始查找
		url属性：url地址
	-->
    <properties resource="jdbc.properties"/>

    <environments default="mysql_mybatis">
        <environment id="mysql_mybatis">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <!-- 使用${OGNL}表达式，获取从properties中得到的配置信息 -->
                <property name="driver" value="${jdbc.driver}"/>
                <property name="url" value="${jdbc.url}"/>
                <property name="username" value="${jdbc.username}"/>
                <property name="password" value="${jdbc.password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper resource="com/itheima/dao/UserDao.xml"/>
    </mappers>
</configuration>
```

#### 1.3 注意加载顺序

假如：

有jdbc.properties配置文件在resource下

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql:///mybatis49
jdbc.username=root
jdbc.password=root
```

SqlMapConfig.xml配置如下：

```xml
<properties resurce="jdbc.properties">
	<property name="jdbc.username" value="root"/>
    <property name="jdbc.password" value="123456"/>
</properties>
```

那么：

​	Mybatis会首先加载xml配置文件里的的property标签，得到了jdbc.username和jdbc.password

​	然后再加载外部jdbc.properties配置文件，覆盖掉刚刚得到的jdbc.username和jdbc.password的值

总结：

​	后加载覆盖先加载，外部properties配置文件的优先级，要高于xml里property标签的配置

### 2. typeAlias类型别名

​	在映射配置文件中，我们要写大量的parameterType和resultType，如果全部都写全限定类名的话，代码就太过冗余，开发不方便。可以使用类型别名来解决这个问题。



​	类型别名：是Mybatis为Java类型设置的一个短名称，目的仅仅是为了减少冗余。

​	注意：**类型别名不区分大小写**

Mybatis提供的别名有：

| **别名**   | **映射的类型** |
| ---------- | -------------- |
| _byte      | byte           |
| _long      | long           |
| _short     | short          |
| _int       | int            |
| _integer   | int            |
| _double    | double         |
| _float     | float          |
| _boolean   | boolean        |
| string     | String         |
| byte       | Byte           |
| long       | Long           |
| short      | Short          |
| int        | Integer        |
| integer    | Integer        |
| double     | Double         |
| float      | Float          |
| boolean    | Boolean        |
| date       | Date           |
| decimal    | BigDecimal     |
| bigdecimal | BigDecimal     |
| object     | Object         |
| map        | Map            |
| hashmap    | HashMap        |
| list       | List           |
| arraylist  | ArrayList      |
| collection | Collection     |
| iterator   | Iterator       |

### 3. 自定义类型别名

​	例如：自己定义的JavaBean，全限定类名太长，可以自定义类型别名。

#### 3.1 给一个类指定别名

##### 3.1.1 在SqlMapConfig.xml中配置一个类的别名

```xml
    <typeAliases>
        <!-- type：要指定别名的全限定类名    alias：别名 -->
        <typeAlias type="com.itheima.domain.QueryVO" alias="vo"/>
        <typeAlias type="com.itheima.domain.User" alias="user"/>
        <typeAlias type="com.itheima.domain.User2" alias="user2"/>
    </typeAliases>
```

##### 3.1.2 在映射配置文件中使用类型别名

```xml
<!-- parameterType使用别名：vo， resultType使用别名：user -->    
<select id="findByVO" parameterType="vo" resultType="user">
    select * from user where username like #{user.username}
</select>
```

#### 3.2 指定一个包名

##### 3.2.1 在SqlMapConfig.xml中为一个package下所有类注册别名：<span style="color:red;">类名即别名</span>

```xml
    <typeAliases>
        <!-- 把com.itheima.domain包下所有JavaBean都注册别名，类名即别名，不区分大小写 -->
        <package name="com.itheima.domain"/>
    </typeAliases>
```

##### 3.2.2 在映射配置文件中使用类型别名

```xml
<!-- parameterType使用别名：queryvo， resultType使用别名：user -->    
<select id="findByVO" parameterType="queryvo" resultType="user">
    select * from user where username like #{user.username}
</select>
```

### 4. mappers映射器

​	用来配置映射器接口的配置文件位置，或者映射器接口的全限定类名

#### 4.1 `<mapper resource=""/>`

用于指定映射配置文件xml的路径，支持xml开发方式，例如：

`<mapper resource="com/itheima/dao/UserDao.xml"/>`

注意：

​	映射配置文件的名称，和映射器接口类名   可以不同

​	映射配置文件的位置，和映射器接口位置   可以不同

> 配置了xml的路径，Mybatis就可以加载statement信息，并且根据namespace属性找到映射器

#### 4.2 `<mapper class=""/>`

用于指定映射器接口的全限定类名，支持XML开发和注解开发，例如：

`<mapper class="com.itheima.dao.UserDao"/>`

如果是使用xml方式开发，那么要注意：

​	映射配置文件的名称 要和 映射器接口的类名相同

​	映射配置文件的位置 要和 映射器接口的位置相同

> Mybatis只知道映射器的名称和位置，不知道配置文件的名称和位置。只能查找同名同路径的配置文件

#### 4.3 `<package name=""/>`

用于自动注册指定包下所有的映射器接口，支持XML开发和注解开发，例如：

`<package class="com.itheima.dao"/>`

如果是使用XML方式开发，那么要注意：

​	映射配置文件的名称 要和 映射器接口的类名相同

​	映射配置文件的位置 要和 映射器接口的位置相同

> Mybatis只能根据包名找到所有的映射器的类名和位置， 不知道配置文件的名称和位置。只能查找同名同路径的配置文件

## 四、数据源和事务（了解）

### 1. Mybatis的数据源

　　Mybatis中的数据源，是指核心配置文件中`<dataSouce></dataSouce>`的配置。Mybatis为了提高数据库操作的性能，也使用了连接池的技术，但是它采用的是自己开发的连接池技术。

#### 1.1 Mybatis中dataSouce的分类

##### 1.1.1 三种dataSouce介绍

- **UNPOOLED**：不使用连接池技术的数据源

  对应Mybatis的`UnpooledDataSouce`类，虽然也实现了`javax.sql.DataSource`接口，但是它的`getConnection()`方法中，并没有真正的使用连接池技术，而是直接从数据库中创建的连接对象，即：`DriverManager.getConnection()`方法

- **POOLED**：使用连接池技术的数据源

  对应Mybatis的`PooledDataSouce`类，它实现了`javax.sql.DataSouce`接口，同时采用了Mybatis自己开发的连接池技术，是我们使用最多的一种数据源

- **JNDI**：使用JNDI技术的数据源

  采用服务器软件提供的JNDI技术实现，从服务器软件中获取数据源。从不同服务器软件中得到的`DataSouce`对象不同。例如：Tomcat中配置了数据连接信息，我们的web应用部署到Tomcat里，就可以获取Tomcat中配置的数据源，而不需要web应用自己再配置数据库连接信息。

##### 1.1.2 三种dataSouce的关系与源码分析

- `UnpooledDataSource`和`PooledDataSource`都实现了`javax.sql.DataSource`接口
- `UnpooledDataSource`没有使用连接池技术，它的`getConnection()`方法是从数据库中创建的连接
- `PooledDataSource`采用了连接池技术
  - 它内部有`UnpooledDataSource`的引用,当需要创建新连接时,是调用`UnpooledDataSource`来获取的
  - 它只是提供了用于存放连接对象的池子

##### 1.1.3 `PooledDataSource`获取新连接的过程源码分析

<img src="img/Mybatis%E8%BF%9E%E6%8E%A5%E6%B1%A0%E7%B1%BB%E5%9B%BE+%E8%8E%B7%E5%8F%96%E8%BF%9E%E6%8E%A5%E6%B5%81%E7%A8%8B%E5%9B%BE.png"/>

#### 1.2  Mybatis中dataSouce的配置

```xml
<dataSource type="POOLED">
    <property name="driver" value="com.mysql.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql:///mybatis49"/>
    <property name="username" value="root"/>
    <property name="password" value="root"/>
</dataSource>
```

​	当Mybatis读取核心配置文件时，会根据我们配置的dataSource标签的type，来生成对应的dataSource对象

### 2. Mybatis的事务

#### 2.1 JDBC的事务管理

##### 2.1.1 事务管理

　　JDBC的事务管理是基于`Connection`对象实现的：

- 开启事务：connection.setAutoCommit(false)
- 提交事务：connection.commit()
- 回滚事务：connection.rollback()

##### 2.1.2 事务概念

###### 事务的特性：ACID

* 原子性：事务是不可分割的。一个事务里的操作，不可能成功一半
* 一致性：事务提交前后，数据/状态是一致的
* 隔离性：事务并发时，事务应该是互不干扰相互独立的
* 持久性：事务一旦提交，数据就永久保存到磁盘上。

###### 事务并发时可能存在的问题

* 脏读：一个事务里读取到另外一个事务未提交的数据
* 不可重复读：一个事务里，多次读取的数据不一致。是受到了其它事务update的干扰
* 虚读/幻读：一个事务里，多次读取的数据不一致。是受到了其它事务insert、delete干扰

> 事务之间的隔离级别不够高，会导致事务并发问题

###### 使用隔离级别解决事务并发问题

| 隔离级别           | 脏读 | 不可重复读 | 虚读 |
| ------------------ | ---- | ---------- | ---- |
| `read uncommitted` | 有   |            | 有   |
| `read committed`   | 无   | 有         | 有   |
| `repeatable read`  | 无   | 无         | 有   |
| `serializable`     | 无   | 无         | 无   |



#### 2.2 Mybatis的事务管理

　　因为Mybatis的是对JDBC的封装，所以Mybatis在本质上也是基于`Connection`对象实现的事务管理，只是把管理的代码封装起来了，是使用`SqlSession`对象进行事务管理的。

##### 2.2.1 Mybatis的默认事务管理方式

　　默认情况下，我们使用工厂对象的`openSession()`方法得到的`SqlSession`对象，是关闭了事务自动提交的，即：**默认情况下，`SqlSession`是开启了事务的**。

​	    获取session对象：`factory.openSession()`

　　操作完数据库之后，需要手动提交事务：`sqlSession.commit();`

　　如果要回滚事务，就使用方法：`sqlSession.rollback();`	

##### 2.2.2 Mybatis的自动提交事务实现

　　Mybatis也支持自动提交事务，操作方法如下：

1. 获取SqlSession对象：`factory.openSession(true)`
2. 操作数据库，事务会自动提交

```java
InputStream is = Resources.getResourceAsStream("SqlMapConfig.xml");
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(is);
//获取的是自动提交事务的SqlSession，所以不需要再手动关闭事务
SqlSession session = factory.openSession(true);
UserDao dao = session.getMapper(UserDao.class);

//操作数据库
dao.delete(48);

//释放资源，事务会自动提交，所以不需要再手动提交事务
session.close();
is.close();
```

## 五、SQL深入-动态sql

### 1. Mybatis动态SQL简介

​	我们在前边的学习过程中，使用的SQL语句都非常简单。而在实际业务开发中，我们的SQL语句通常是动态拼接而成的，比如：条件搜索功能的SQL语句。

　　在Mybatis中，SQL语句是写在映射配置的XML文件中的。Mybatis提供了一些XML的标签，用来实现动态SQL的拼接。	常用的标签有：

- `<if></if>`：用来进行判断，相当于Java里的if判断
- `<where></where>`：通常和if配合，用来代替SQL语句中的`where 1=1`
- `<foreach></foreach>`：用来遍历一个集合，把集合里的内容拼接到SQL语句中。例如拼接：`in (value1, value2, ...)`
- `<sql></sql>`：用于定义sql片段，达到重复使用的目的

### 2. Mybatis环境准备

​	我们以对user表的操作为例，演示这些标签的用法。先准备Mybatis的环境如下

1. 创建Maven的java项目，配置好坐标，引入Mybatis的依赖

   ```xml
   	<!-- 项目坐标 -->
   	<groupId>com.itheima</groupId>
       <artifactId>mybatis_day03_sql</artifactId>
       <version>1.0-SNAPSHOT</version>
   
   	<!-- 引入jar包依赖 -->
   	<dependencies>
           <dependency>
               <groupId>mysql</groupId>
               <artifactId>mysql-connector-java</artifactId>
               <version>5.1.46</version>
           </dependency>
           <dependency>
               <groupId>org.mybatis</groupId>
               <artifactId>mybatis</artifactId>
               <version>3.4.6</version>
           </dependency>
           <dependency>
               <groupId>log4j</groupId>
               <artifactId>log4j</artifactId>
               <version>1.2.17</version>
           </dependency>
           <dependency>
               <groupId>junit</groupId>
               <artifactId>junit</artifactId>
               <version>4.12</version>
           </dependency>
       </dependencies>
   ```

2. 创建JavaBean实体类：User

   ```java
   public class User {
       private Integer id;
       private String username;
       private Date birthday;
       private String sex;
       private String address;
   
       public Integer getId() {
           return id;
       }
   
       public void setId(Integer id) {
           this.id = id;
       }
   
       public String getUsername() {
           return username;
       }
   
       public void setUsername(String username) {
           this.username = username;
       }
   
       public Date getBirthday() {
           return birthday;
       }
   
       public void setBirthday(Date birthday) {
           this.birthday = birthday;
       }
   
       public String getSex() {
           return sex;
       }
   
       public void setSex(String sex) {
           this.sex = sex;
       }
   
       public String getAddress() {
           return address;
       }
   
       public void setAddress(String address) {
           this.address = address;
       }
   
       @Override
       public String toString() {
           return "User{" +
                   "id=" + id +
                   ", username='" + username + '\'' +
                   ", birthday=" + birthday +
                   ", sex='" + sex + '\'' +
                   ", address='" + address + '\'' +
                   '}';
       }
   }
   ```

3. 创建映射器接口UserDao（准备好备用，暂时不需要加方法）

   ```java
   public interface UserDao {
   }
   ```

4. 创建映射配置文件UserDao.xml（准备好备用）

   ```xml
   <?xml version="1.0" encoding="utf-8"?>
   <!DOCTYPE mapper
           PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="com.itheima.dao.UserDao">
   </mapper>
   ```

5. 创建Mybatis的核心配置文件，配置好类型别名和映射器

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE configuration
           PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <typeAliases>
           <package name="com.itheima.domain"/>
       </typeAliases>
   
       <environments default="mysql">
           <environment id="mysql">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql:///mybatis"/>
                   <property name="username" value="root"/>
                   <property name="password" value="root"/>
               </dataSource>
           </environment>
       </environments>
   
       <mappers>
           <package name="com.itheima.dao"/>
       </mappers>
   </configuration>
   ```

6. 准备log4j日志配置文件

   ```properties
   # Set root category priority to INFO and its only appender to CONSOLE.
   #log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
   log4j.rootCategory=debug, CONSOLE, LOGFILE
   
   # Set the enterprise logger category to FATAL and its only appender to CONSOLE.
   log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE
   
   # CONSOLE is set to be a ConsoleAppender using a PatternLayout.
   log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
   log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
   log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
   
   # LOGFILE is set to be a File appender using a PatternLayout.
   log4j.appender.LOGFILE=org.apache.log4j.FileAppender
   log4j.appender.LOGFILE.File=d:\axis.log
   log4j.appender.LOGFILE.Append=true
   log4j.appender.LOGFILE.layout=org.apache.log4j.PatternLayout
   log4j.appender.LOGFILE.layout.ConversionPattern=%d{ISO8601} %-6r [%15.15t] %-5p %30.30c %x - %m\n
   ```

7. 准备好单元测试类（准备好备用） 

   ```java
   public class MybatisSqlTest {
       private InputStream is;
       private SqlSession session;
       private UserDao dao;
   
       @Before
       public void init() throws IOException {
           is = Resources.getResourceAsStream("SqlMapConfig.xml");
           SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
           SqlSessionFactory factory = builder.build(is);
           session = factory.openSession();
           dao = session.getMapper(UserDao.class);
       }
   
       @After
       public void destory() throws IOException {
           //7. 释放资源
           session.close();
           is.close();
       }
   }
   ```

### 3. 动态SQL的标签应用

#### 3.1 `<if>`标签：

##### 3.1.1 语法介绍

```xml
<if test="判断条件，使用OGNL表达式进行判断">
	SQL语句内容， 如果判断为true，这里的SQL语句就会进行拼接
</if>
```

##### 3.1.2 需求描述

​	根据用户的名称和性别搜索用户信息。把搜索条件放到User对象里，传递给SQL语句

##### 3.1.3 需求实现

###### 1) 在映射器接口UserDao中增加方法

```java
public interface UserDao {
	//根据user搜索用户信息
    List<User> search(User user);
}
```

###### 2) 在映射配置文件UserDao.xml中配置statement

```xml
    <select id="search" parameterType="user" resultType="user">
        select * from user where 1=1
        <if test="username != null">
            and username like #{username}
        </if>
        <if test="sex != null">
            and sex = #{sex}
        </if>
    </select>
```

> 注意：
>
> 1. SQL语句中   where 1=1 不能省略
> 2. 在if标签的test属性中，直接写OGNL表达式，从parameterType中取值进行判断，不需要加#{}或者${}

###### 3) 在单元测试类中编写测试代码

```java
    @Test
    public void testSearch(){
        User user = new User();
        user.setUsername("%王%");
        user.setSex("男");
        
        List<User> users = dao.search(user);
        for (User u : users) {
            System.out.println(u);
        }
    }
```

##### 3.1.4 小结

```xml
<if test="用OGNL表达式进行判断">
	写SQL语句，如果判断为true，这里的SQL会生效拼接进去
</if>
```



#### 3.2 `<where>`标签

##### 3.2.1 语法介绍

　　在刚刚的练习的SQL语句中，我们写了`where 1=1`。如果不写的话，SQL语句会出现语法错误。Mybatis提供了一种代替`where 1=1`的技术：`<where></where>`标签。

##### 3.2.2 需求描述

​	把2.1.2中的实现代码	进行优化，使用`<where></where>`标签代替`where 1=1`

##### 3.2.3 需求实现

​	只需要把刚刚的映射配置文件进行修改，修改后：

```xml
    <select id="search" parameterType="user" resultType="user">
        select * from user
        <where>
            <if test="username != null">
                and username like #{username}
            </if>
            <if test="sex != null">
                and sex = #{sex}
            </if>
        </where>
    </select>
```

> 注意：
>
> 1. `<where>`标签代替了`where 1=1`
> 2. `<where>`标签内拼接的SQL没有变化，每个if的SQL中都有`and`
> 3. 使用`<where>`标签时，Mybatis会自动处理掉条件中的第1个`and`，以保证SQL语法正确

再次运行测试代码，查看结果仍然正常

#### 3.3 `<foreach>`标签

##### 3.3.1 语法介绍

​	foreach标签，通常用于循环遍历一个集合，把集合的内容拼接到SQL语句中。例如，我们要根据多个id查询用户信息，SQL语句：

```sql
select * from user where id = 1 or id = 2 or id = 3;
select * from user where id in (1, 2, 3);
```

​	假如我们传参了id的集合，那么在映射配置文件中，如何遍历集合拼接SQL语句呢？可以使用`foreach`标签实现。

```xml
foreach标签：
	属性：
		collection：被循环遍历的对象，使用OGNL表达式获取，注意不要加#{}
		open：SQL语句的开始部分
		item：定义变量名，代表被循环遍历中每个元素，生成的变量名
		separator：分隔符
		close：SQL语句的结束部分
	标签体：
		使用#{OGNL}表达式，获取到被循环遍历对象中的每个元素
<foreach collection="" open="id in(" item="id" separator="," close=")">
    #{id}
</foreach>
id in(第0个id的值, 第1个id的值,...)
```

##### 3.3.2 需求描述

　　QueryVO中有一个属性ids， 是id值的集合。根据QueryVO中的ids查询用户列表。

​	QueryVO类如下：

```java
public class QueryVO {
    
    private Integer[] ids;

    public Integer[] getIds() {
        return ids;
    }

    public void setIds(Integer[] ids) {
        this.ids = ids;
    }
}
```

##### 3.3.3 需求实现

###### 1) 在映射器接口UserDao中增加方法

```java
List<User> findByIds(QueryVO vo);
```

###### 2) 在映射配置文件UserDao.xml中添加statement

```xml
<!--在核心配置文件中已经使用package配置了类型别名-->
<select id="findByIds" resultType="user" parameterType="queryvo">
    select * from user 
    <where>
        <foreach collection="ids" open="and id in (" item="id" separator="," close=")">
            #{id}
        </foreach>
    </where>
</select>
```

###### 3) 在单元测试类中编写测试代码

```java
    @Test
    public void testFindUserByIdsQueryVO(){
        QueryVO vo = new QueryVO();
        vo.setIds(new Integer[]{41, 42});

        List<User> userList = dao.findByIds(vo);
        for (User user : userList) {
            System.out.println(user);
        }
    }

```

#### 3.4 `<sql>`标签

　　在映射配置文件中，我们发现有很多SQL片段是重复的，比如：`select * from user`。Mybatis提供了一个`<sql>`标签，把重复的SQL片段抽取出来，可以重复使用。

##### 3.4.1 语法介绍

在映射配置文件中定义SQL片段：

```xml
<sql id="唯一标识">sql语句片段</sql>
```

在映射配置文件中引用SQL片段：

```xml
<include refid="sql片段的id"></include>
```

> 扩展：
>
> ​	如果想要引入其它映射配置文件中的sql片段，那么`<include>`标签的refid的值，需要在sql片段的id前指定namespace。例如：
>
> ​	`<include refid="com.itheima.dao.RoleDao.selectRole"></include>`
>
> ​	表示引入了namespace为`com.itheima.dao.RoleDao`的映射配置文件中id为`selectRole`的sql片段

##### 3.4.2 需求描述

在查询用户的SQL中，需要重复编写：`select * from user`。把这部分SQL提取成SQL片段以重复使用

##### 3.4.3 需求实现

###### 1) 在映射配置文件UserDao.xml中定义SQL片段

```xml
    <sql id="selectUser">
        select * from user
    </sql>
```

###### 2) 在映射配置文件UserDao.xml中使用SQL片段

```xml
<select id="findByIds" resultType="user" parameterType="queryvo">
    <!-- refid属性：要引用的sql片段的id -->
    <include refid="selectUser"></include> 
    <where>
        <foreach collection="ids" open="and id in (" item="id" separator="," close=")">
            #{id}
        </foreach>
    </where>
</select>
```

# 内容回顾

1. 参数深入：映射配置文件里的parameterType和resultType

   * parameterType：写参数的类型，可以写：

     * 简单类型：SQL里取值`#{随意写}`
     * POJO对象：SQL里取值`#{JavaBean的属性名}`
     * POJO包装类：SQL里取值`#{JavaBean属性名.key......}`

   * resultType：写查询结果集的封装类型，可以写：

     * 简单类型：直接写类型名称/别名

     * POJO对象：直接写JavaBean的全限定类名或者别名。

       > 注意：要求JavaBean的属性名和表字段名保持一致

   * resultMap：用于设置JavaBean属性名 和 表字段名的对应关系

     ```xml
     <select id="" resultMap="resultMap的唯一标识">
     	SQL语句
     </select>
     
     <resultMap id="唯一标识" type="JavaBean名称/别名">
     	<id property="属性名" column="字段名"/>
         <result property="属性名" column="字段名"/>
     </resultMap>
     ```

2. 传统方式CURD（了解）

   * 实现类的结构：

   ```java
   public class 实现类 implements 映射器接口{
       private SqlSessionFactory factory;
       public 实现类(SqlSessionFactory factory){
           this.factory = factory;
       }
       
       public List<User> queryAll(){
           //1.从factory里得到SqlSession对象
           SqlSession session = factory.openSession();
           //2.操作数据库
           List<User> userList = session.selectList("statement");
           //3.关闭session对象。如果是DML操作，在关闭前要提交：session.commit();
           session.close();
       }
   }
   ```

   * SqlSession对象：
     * 提供了操作数据库的一些方法
       * `selectList(statement, params)`
       * `selectOne(statement, params)`
       * `insert(statement, params)`
       * `update(statement, params)`
       * `delete(statement, params)`
     * 使用原则：随用随取，用完就关，一定要关
   * SqlSessionFactory工厂：
     * 提供获取SqlSession对象的方法：
       * `openSession()`：获取  需要手动提交事务的SqlSession
       * `openSession(true)`：获取 自动提交事务的SqlSession
     * 使用原则：通常是单例模式维护
   * SqlSessionFactoryBuilder对象：
     * 提供了构造一个工厂对象的方法
       * ` new SqlSessioNFactoryBuilder().build(InputStream is)`：得到一个SqlSessionFactory工厂对象
     * 使用原则：用完就扔，等待垃圾回收

3. 核心配置文件：

   * 配置别名：

     ```xml
     <typeAliases>
     	<package name="JavaBean所在的包名"/>
     </typeAliases>
     ```

   * 配置映射器：

     ```xml
     <mappers>
     	<package name="映射器所在的包名"/>
     </mappers>
     ```

     > 注意：映射器的名称和位置， 要和 映射配置文件的名称、位置  一样

4. SQL深入

   ```xml
   <select parameter="queryvo">
       <!-- 引入使用SQL片段 -->
   	<include refid="selectUser"/>
   	<where>
       	<if test="ids!=null and ids.length>0">
           	<foreach collection="ids" open=" and id in(" item="id" separator="," close=")">
                   #{id}
               </foreach>
           </if>
           
           <if test="user!=null">
           	<if test="user.username!=null and user.username != ''">
               	and username like #{user.username}
               </if>
               <if test="user.sex!=null and user.sex != ''">
               	and sex = #{user.sex}
               </if>
           </if>
       </where>
   </select>
   
   <!-- 定义SQL片段，用于重复引用 -->
   <sql id="selectUser">select * from user</sql>
   ```

   