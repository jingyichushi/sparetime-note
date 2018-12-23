# MyBatis Plus学习

## MyBatis Plus介绍



[MyBatis-Plus](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis](http://www.mybatis.org/mybatis-3/) 的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

### MyBatis Plus特性

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer2005、SQLServer 等多种数据库
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 XML 热加载**：Mapper 对应的 XML 支持热加载，对于简单的 CRUD 操作，甚至可以无 XML 启动
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **支持关键词自动转义**：支持数据库关键词（order、key......）自动转义，还可自定义关键词
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作
- **内置 Sql 注入剥离器**：支持 Sql 注入剥离，有效预防 Sql 注入攻击

### MyBatis Plus框架结构

![framework](http://img.bcoder.top/2018.12.22/1.jpg)

## MyBatis实践

### MyBatis环境准备

1.pom文件添加依赖

```xml
<!--mybatis plus整合(与mybatis整合不共存)-->
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.0.6</version>
</dependency>
```

2.准备数据表

```sql
CREATE TABLE demo_user (
  id                   BIGINT AUTO_INCREMENT PRIMARY KEY NOT NULL  COMMENT '角色ID',
  user_name            VARCHAR(16)                         NULL    COMMENT '用户名',
  password             VARCHAR(32)                         NULL    COMMENT '密码',
  mail                 VARCHAR(64)                         NULL    COMMENT '邮箱',
  phone                VARCHAR(32)                         NULL    COMMENT '电话',
  job                  VARCHAR(16)                         NULL    COMMENT '职位'
) ENGINE=InnoDB DEFAULT CHARSET=UTF8MB4 COMMENT='用户表';
```

3.构建实体bean

```java
@Data
@Builder
public class DemoUser implements Serializable {
    private static final long serialVersionUID = 1L;

    private Long id;

    private String userName;

    private String password;

    private String mail;

    private String phone;

    private String job;
}
```

4.准备假数据

```sql
INSERT INTO iot.demo_user (user_name, password, mail, phone, job) VALUES ('aa', '123', 'aa@163.com', '15651775511', '程序员');
INSERT INTO iot.demo_user (user_name, password, mail, phone, job) VALUES ('bb', '123', 'bb@163.com', '15651775512', '测试');
INSERT INTO iot.demo_user (user_name, password, mail, phone, job) VALUES ('cc', '123', 'cc@163.com', '15651775513', '产品');
INSERT INTO iot.demo_user (user_name, password, mail, phone, job) VALUES ('dd', '123', 'dd@163.com', '15651775514', '程序员');
INSERT INTO iot.demo_user (user_name, password, mail, phone, job) VALUES ('ee', '123', 'ee@163.com', '15651775515', '程序员');
```



5.application.yml文件配置：

```yml
mybatis-plus:
  config-location: classpath:mybatis/mybatis‐config.xml
  global-config:
    db-config:
      id-type: auto
```



6.创建DemoUserMapper.java

```java
public interface DemoUserMapper extends BaseMapper<DemoUser> {
}
```



7.注解学习

```java
@Data
@TableName(value="iot_user")
public class DemoUser{
   @TableId(value="companyId",type=IdType.AUTO)
	private Integer id; 
    
    @TableField(value="email")
    private String mail;
}




```







### SQL实战

### 插入

插入指定值(非空属性)：

INSERT INTO demo_user ( user_name, password, phone, job ) VALUES ( ?, ?, ?, ? ) 

```java
    public Result test() {
        DemoUser user = DemoUser.builder().userName("ff")
                .password("123").job("经理").mail("123@qq.com")
                .phone("15651775518").build();
        int insert = demoUserMapper.insert(user);
        log.info(""+insert);
        //主键回填
        return HttpResult.buildSuccessResult(user);
    }
```



### 更新

1.根据主键进行更新(只更新非空属性)：

UPDATE demo_user SET phone=? WHERE id=? 

```java
public Result test() {
    DemoUser user = DemoUser.builder().id(1L)
        .phone("15651775518").build();
    demoUserMapper.updateById(user);
    return HttpResult.buildSuccessResult(user);
}
```



2.条件更新进行更新(只更新非空属性)：

UPDATE demo_user SET phone=? WHERE id = ? 

```java
public Result test() {
    DemoUser user = DemoUser.builder()
        .phone("156517755166").build();
    demoUserMapper.update(user,new QueryWrapper<DemoUser>().eq("id","2"));
    return HttpResult.buildSuccessResult(user);
}
```



### 查找

1.根据主键查询

SELECT id,user_name,password,mail,phone,job FROM demo_user WHERE id=? 

```java
public Result test() {
    DemoUser user = demoUserMapper.selectById(5);
    return HttpResult.buildSuccessResult(user);
}
```



2.批量查询（根据多个ID）：

SELECT id,user_name,password,mail,phone,job FROM demo_user WHERE id IN ( ? , ? , ? ) 

```java
public Result test() {
    List<DemoUser> demoUsers = demoUserMapper.selectBatchIds(Arrays.asList(new Long[]{1L, 2L, 3L}));
    return HttpResult.buildSuccessResult(demoUsers);
}

```



3.多条件查询（查询多个结果）：

SELECT id,user_name,password,mail,phone,job FROM demo_user WHERE password = ? AND user_name = ? 

``` java
public Result test() {
    Map<String, Object> condition = new HashMap<>();
    condition.put("user_name", "cc");
    condition.put("password", "123");
    List<DemoUser> demoUsers = demoUserMapper.selectByMap(condition);
    return HttpResult.buildSuccessResult(demoUsers);
}
```

第二种是用条件查询器（更丰富的限制）：

SELECT id,user_name,password,mail,phone,job FROM demo_user WHERE id > ? 

```java
public Result test() {
    List<DemoUser> users = demoUserMapper.selectList(new QueryWrapper<DemoUser>().gt("id", 3L));
    return HttpResult.buildSuccessResult(users);
}
```

4.查询总记录数（或者带上条件）

SELECT COUNT(1) FROM demo_user 

```java
public Result test() {
    Integer count = demoUserMapper.selectCount(null);
    return HttpResult.buildSuccessResult(count);
}
```



## 条件构造器

`allEq({id:1,name:"老王",age:null})`--->`id = 1 and name = '老王' and age is null`

`allEq({id:1,name:"老王",age:null}, false)`--->`id = 1 and name = '老王'`

`eq("name", "老王")`--->`name = '老王'`

`ne("name", "老王")`--->`name <> '老王'`

`gt("age", 18)`--->`age > 18`

`ge("age", 18)`--->`age >= 18`

`lt("age", 18)`--->`age < 18`

`le("age", 18)`--->`age <= 18`

`between("age", 18, 30)`--->`age between 18 and 30`

`notBetween("age", 18, 30)`--->`age not between 18 and 30`

`like("name", "王")`--->`name like '%王%'`

`notLike("name", "王")`--->`name not like '%王%'`

`likeLeft("name", "王")`--->`name like '%王'`

`likeRight("name", "王")`--->`name like '王%'`

`isNull("name")`--->`name is null`

`isNotNull("name")`--->`name is not null`

`in("age",{1,2,3})`--->`age in (1,2,3)`

`notIn("age",{1,2,3})`--->`age not in (1,2,3)`

`notIn("age", 1, 2, 3)`--->`age not in (1,2,3)`

`inSql("age", "1,2,3,4,5,6")`--->`age in (1,2,3,4,5,6)`

`inSql("id", "select id from table where id < 3")`--->`id in (select id from table where id < 3)`

`notInSql("age", "1,2,3,4,5,6")`--->`age not in (1,2,3,4,5,6)`

`notInSql("id", "select id from table where id < 3")`--->`age not in (select id from table where id < 3)`

`groupBy("id", "name")`--->`group by id,name`

`orderByAsc("id", "name")`--->`order by id ASC,name ASC`

`orderByDesc("id", "name")`--->`order by id DESC,name DESC`

`orderBy(true, true, "id", "name")`--->`order by id ASC,name ASC`

`having("sum(age) > 10")`--->`having sum(age) > 10`

`having("sum(age) > {0}", 11)`--->`having sum(age) > 11`

`eq("id",1).or().eq("name","老王")`--->`id = 1 or name = '老王'`

`or(i -> i.eq("name", "李白").ne("status", "活着"))`--->`or (name = '李白' and status <> '活着')`

`and(i -> i.eq("name", "李白").ne("status", "活着"))`--->`and (name = '李白' and status <> '活着')`

`apply("id = 1")`--->`having sum(age) > 10`

`apply("date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`--->`date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`

`apply("date_format(dateColumn,'%Y-%m-%d') = {0}", "2008-08-08")`--->`date_format(dateColumn,'%Y-%m-%d') = '2008-08-08'")`

`exists("select id from table where age = 1")`--->`exists (select id from table where age = 1)`

`notExists("select id from table where age = 1")`--->`not exists (select id from table where age = 1)`



`nested(i -> i.eq("name", "李白").ne("status", "活着"))`--->`(name = '李白' and status <> '活着')`

select("id", "name", "age")





































