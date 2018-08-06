# MyBatis
[TOC]

## 传统JDBC的缺点
JDBC 连接和操作数据的API（个人理解）
+ 硬编码 SQL语句嵌入代码中，扩展性能差
+ 数据库频繁的建立和断开连接
+ 查询结果集取数据硬编码，不能直接转换为对象

## MyBatis的运行流程
+ 通过Resource资源加载类加载Mybatis全局配置文件
+ 通过配置文件创建SqlSessionFactory对象，在用工厂方法创建sqlSession对象
+ sqlSession通过内部的Executeor执行器，使用预编译的sql语句执行对数据的操作
```java
InputStream inputStream = Resource.getResourceAsStream("myBatis.xml");
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
SqlSession sqlSession = sqlSessionFactory.build();
```
## Mybatis配置文件详解
**详见SpringMVC+MyBatis开发从入门到项目实战**
### 别名typeAliases
>使用别名的原因：在Mybatis的mapper映射文件中，经常用到一个类的全类名如paramterType，resultType，因此给一个类取一个别名

>配置方式：
```java
<typeAliases>
   <typeAlias alias='user' type="com.muyuxi.bean.user"/>
</typeAliases>
```
或者
```java
<typeAliases>
   <package name="com.muyuxi.bean"/>
</typeAliases>
```
默认使用类名首字母小写作为其别名
### 驼峰命名
>如果Java包装类使用驼峰式命名，则需要在全局配置文件将mapUnderScoreToCamelCase属性设置为true，否则自动映射机制无法将查询出的非驼峰命名方式的字段名映射为Java包装类中的属性。

### mapper配置输入映射
parameterType = ""
```java
<delete id="deleteUser" parameterType="com.muyuxi.bean.User">
   delete from user where userName = #{userName}
</delete>
```
### mapper配置输出映射文件
#### resultType
Mybatis中，不管输出的是JavaBean单个对象还是一个列表包含javaBean，在mapper映射文件汇总resultType指定的类型都是该JavaBean
```java
<select id="findUser" parameterType="string" resultType="com.muyuxi.bean.User">
   select * from user where userName = #{userName}
</select>
```
#### resultMap
如果返回的结果集没有一个完全与结果值匹配的封装类一一接收，则使用一个容器接收数据，再进行属性的映射,即进行列名和字段名的映射
```java
<resultMap type="Departments" id="DepartmentsResult">  
    <id property="id" column="id" />  
    <result property="departmentName" column="departmentName" />  
    <collection property="userInfos" column="id" select="com.csdn.ingo.dao.UserInfoDao.findUserInfoByDepartmentId"></collection>  
</resultMap>  
```

- 关联的嵌套结果（association） 包装类可能嵌套其他的类是，就要使用关联嵌套，比如联合查询，查询员工的信息要关联到其所在的部门，在查询的结果集使用resultMap关联association到Java类型(关联写**JavaType**不是Type)

```java
<resultMap type="user" id="UserResult">  
        <id property="id" column="id" />  
        <result property="password" column="password" />  
        <association property="userInfo" column="userid"  
            select="com.csdn.ingo.dao.UserInfoDao.findUserInfoById"></association>  
        <association property="department" column="department"  
            select="com.csdn.ingo.dao.DepartmentsDao.findDepartmentById"></association>  
</resultMap>
```
- 集合的嵌套结果 包装类中含有List，使用collection，如上上例，集合嵌套结果使用**ofType="User"**，也可以在collection中嵌套查询，只要指定查询的条件 column，然后执行select为sql语句还是复用其他的sql映射配置。
```java
<resultMap type="Departments" id="DepartmentsResult">  
    <id property="id" column="id" />  
    <result property="departmentName" column="departmentName" />  
    <collection property="userInfos" column="id" select="com.csdn.ingo.dao.UserInfoDao.findUserInfoByDepartmentId" column="id"></collection>  
</resultMap>  
```
### mapper自动映射
可在mybatis全局配置文件中 
<setting name="autoMappingbehavior" value="FULL"/>设置自动映射的方式
Mybatis有三种自动映射的方式：
- NONE 不启动自动映射
- PARTIAL 只对非嵌套的resultMap进行自动映射（默认的映射方式）
- FULL 对所有的resultMap进行自动映射  慎用会造成数据混乱
partial自动映射用于在resultMap中有的属性名与数据库字段一致，因此不用写出来，可以通过自动映射进行，但是只能在**不包含嵌套的resultMap中使用**

### mapper配置动态sql
```java
<where>
  <if test = "" !=null>
    and 
  </if>
  <include refid=""></include>  //复用sql
</where>

//复用的sql
<sql id = "">
   //复用的语句
   <where>
      <if test=""!=null>
         and
      </if>
   </where>
</sql>
```
**choose when otherwise**的形式即if elseif else的形式
```java
<choose>
	<when test="terminalLog.num != null">
		SELECT t.Num, t.TerminalID, t.Time, t.StationID,
		t.LogType, t.Length,
		t.Event,
		DATE_FORMAT(t.TimeSave,'%Y-%m-%d
		%h:%m:%s') TimeSave,d.title
		FROM (
		SELECT max(t.Num) AS num1,
		t.TerminalID FROM terminallog t GROUP
		BY
		t.TerminalID ) t1,
		terminallog t, deploy d WHERE t1.num1 = t.num
		AND t1.TerminalID =
		d.TerminalID
	</when>
	<otherwise>
		SELECT
		t.Num, t.TerminalID, t.Time, t.StationID, t.LogType, t.Length,
		t.Event,
		DATE_FORMAT(t.TimeSave,'%Y-%m-%d %H:%i:%S') TimeSave,d.title
		FROM terminallog t, deploy d
		WHERE t.StationID = d.StationID
	</otherwise>
</choose>
```
**当查询语句包含多个信息时可以使用<foreach>**
```java
<sql id="query_user_where">
  <if test="ids"!=null>
    //实现 where (id = 1 or id =2 or id =3)
    <foreach collections = "ids" item="user_id" open="and (" close=")" separtor="or">  //open代表循环开始时要拼接的字符串 close代表循环结束时要拼接的字符串 separtor表示中间拼接的字符串
      id=#{user_id}
  </if>
</sql>
```

**xml文档中的特殊符号**
![](https://gss0.baidu.com/7Po3dSag_xI4khGko9WTAnF6hhy/zhidao/wh%3D600%2C800/sign=f7aaf36e3dadcbef016176009c9f02e5/a1ec08fa513d26977ee6c2f759fbb2fb4316d867.jpg)
## 延迟加载
在使用resultMap中的关联结果和集合结果时就可以用到延迟加载来优化sql的性能，因为查询单表的速度比较快
```java
<association property="customer" javaType="com.muyuxi.bean.User" select="" column="id">
</association>
<select id="" parameterType="int" resultType="User">
  select * from User where userid=#{id}
</select>

```
开启延迟加载的方法：
在全局配置文件中
```java
<configuration>
<settings>
<setting name="lazyLoadingEnable" value="true"/>
<setting name="aggressiveLazyLoading" value="false"/>
</settings>
</configuration>
```
开启延迟加载后在业务逻辑中需要使用包装类中的集合或者对象属性时才会进行延迟加载，调用mapper配置文件中的sql进行数据查询
## mapper动态代理
mapper动态代理即不用写DAO层实现类代码，使用mapper接口和mapper映射文件代替传统的DAO层，接口替代原来DAO层的位置
- mapper接口的名称与xml配置文件名一致，接口中方法的定义与参数返回类型都与mapper.xml配置文件中的sql映射的id，输入输出类型一致
- mapper.xml文件中的namespace与mapper接口的路径一致
即可使用SqlSession类获取mapper的动态代理

使用动态代理省去了sqlSession的操作，可以直接使用mapper接口变量操作mapper映射文件的sql。而mapper代理对象是通过sqlSession对象获得的。
```java
UsreMapper userMapper = sqlSession.getMapper("User.class");
User user = userMapper.findUser(1);
//代替了user = sqlSession.selectOne("findUser",1);使用起来更加灵活方便
```
## Mybatis缓存机构

### 一级缓存
Mybatis默认支持一级缓存，一级缓存是基于同一个SQLSession对象的，即基于一次与数据库的会话
- 第一次查询id为1的用户数据，会将查询的结果保存在一级缓存中
- 下一次查询id为1的用户数据，会直接从缓存汇总取出查询的结果集
- 如果在此期间，执行了commit操作（修改、删除、添加）会清空一级缓存区域

![](https://images2017.cnblogs.com/blog/1120165/201708/1120165-20170810230553636-190248238.png)

### 二级缓存
二级缓存的开启方法：

在MyBatis的全局配置文件中添加setting标签
- <setting name="cacheEnabled" value="true"/>
- 在相应的mapper.xml文件中添加一个<cache/>标签即可

二级缓存是基于Mapper文件的，多个sqlSession对象共享一个mapper文件，因此共享一个二级缓存。
- 增删改会清空Mapper.xml中的二级缓存
- 使用二级缓存需要谨慎，不同的xml文件可能对同一个表有不同的操作，如果在一个mapper文件缓存了user表的数据，当在另一个xml中改变了user表的数据，则此mapper中的缓存并不会被刷新，就可能导致数据不一致！因此**在使用二级缓存的时候，需要保证该mapper中的数据不会在其他mapper.xml文件中有缓存即有操作，如果非要使用就需要使用拦截器判断sql执行了哪些表，然后把相关表的缓存删除**
![](https://ss0.bdstatic.com/70cFvHSh_Q1YnxGkpoWK1HF6hhy/it/u=3794218594,1920538344&fm=27&gp=0.jpg)

## Mybatis技术扩展

### Mybatis与Spring的整合
- 引入mabtis与Spring整合的jar包mybatis-spring-4.3.7
- 数据源交给Spring管理，db.properties文件引入Spring配置文件
- 整合mybatis即mapper对象由spring容器管理创建，因此需要sqlSessionFactory来创建sqlsession对象
- 设置sqlSessionFactory的bean 在此bean下加载mybatis的全局配置文件，以及数据源和mapper映射文件的位置**创建sqlSessionFactory实例对象需要**
- 编写mybatyis全局配置文件：比如别名控制，驼峰式命名、延迟加载、二级缓存等等，相当于把数据源以及mapper映射托管给了spring来管理
- 使用mybatis动态代理，编写mapper映射文件，然后测试

### 关于mapper映射文件的配置

1. 在sqlSessionFactory中配置mapper映射文件的位置 使用
> <property name="mapperLocations" value+"">
2. spring容器有自己的mapper批量扫描器
```java
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value=""/> //扫描多个包包之间用半角逗号隔开
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory">
</bean>
```

### MBG（mybatis逆向工程）
1. 添加MBG的jar包
2. 使用xml配置文件+java代码的形式实现mybatis逆向工程
3. xml中的属性重点  **谓语书中Page：122**
```java
<commentGenerator>
     //是否去除所有的注释
     <property name="suppressAllComments" value="true">
</commentGenerator>
大概其他的配置信息的内容如下：
//数据源的配置
//Po类的生成位置   有一个trimString的属性 在自动映射时可以对从数据库返回的值清理前后的空格，即避免定长char类型的前后补空格的情况
//mapper映射文件的生成位置
//mapper接口的生成位置
//数据库表指定
```
4. 逆向数据文件生成类
> springMVC+mybatis书124页
5. 关于mbg组合查询条件的方法
```java
UserExample = new UserExample();
//通过Criteria构造查询条件
UserExample.Criteria criteria = usewExample.createCriteria();
criteria.andUsernameEqualTo("muyuxi");
criteria.andGenderNotEqualTo("女");
criteria.andBirthDayBetween("1","2");
criteria.andEmailIsNotNull();

```
6. mbg生成的mapper映射文件中的根据主键的增删改查方法，只有查与删是传入主键ID，其他的是传入整个对象修改和添加比较方便。
## #{}与${}
- #{} 占位符 相当于sql中的?
- ${} 拼接符号，拼接字符串 比如'%${value}%'，容易引起SQL注入

**因此可以使用concat('%',#{},'%')进行字符串的拼接用来进行模糊查询**
## sqlSession与数据库连接的关系
联系：两者是对同一行为的不同方面的描述

区别：
- 数据库连接是应用程序到数据可的一条物理链路
- sqlSession回话对象是逻辑层面上用户同数据库的一次回话，即为对数据库的增删改查操作

## Log4j日志

### 日志的等级
- error 不影响程序运行的错误事件
- warn 潜在的错误情形
- info 程序运行的步骤信息
- debug 程序的调试信息

### 日志输出端的格式
- %m 代码中指定的消息
- %p 日志的输出的优先级 info、debug
- %r 应用启动到日志输出的毫秒数
- %c 日志输出所在的全类名
- %t 输出该日志的线程名
- %n 换行 windows平台下为%r&n
- %d 日志输出的时间，格式为&d{yyyy-MM-dd HH:mm:ss}
- &l 日至时间发生的位置信息

### 日志log4j.properties配置文件的写法
```java

### 设置###
log4j.rootLogger = debug,stdout,D,E
 
### 输出信息到控制抬 ###
log4j.appender.stdout = org.apache.log4j.ConsoleAppender
log4j.appender.stdout.Target = System.out
log4j.appender.stdout.layout = org.apache.log4j.PatternLayout
log4j.appender.stdout.layout.ConversionPattern = [%-5p] %d{yyyy-MM-dd HH:mm:ss,SSS} method:%l%n%m%n
 
### 输出DEBUG 级别以上的日志到=E://logs/error.log ###
log4j.appender.D = org.apache.log4j.DailyRollingFileAppender
log4j.appender.D.File = E://logs/log.log
log4j.appender.D.Append = true
log4j.appender.D.Threshold = DEBUG 
log4j.appender.D.layout = org.apache.log4j.PatternLayout
log4j.appender.D.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
 
### 输出ERROR 级别以上的日志到=E://logs/error.log ###
log4j.appender.E = org.apache.log4j.DailyRollingFileAppender
log4j.appender.E.File =E://logs/error.log 
log4j.appender.E.Append = true
log4j.appender.E.Threshold = ERROR 
log4j.appender.E.layout = org.apache.log4j.PatternLayout
log4j.appender.E.layout.ConversionPattern = %-d{yyyy-MM-dd HH:mm:ss}  [ %t:%r ] - [ %p ]  %m%n
```

### 日志对象的调用
```java
private Logger logger = Logger.getLogger(this.getClass());
logger.error("xiixixiixii");
```