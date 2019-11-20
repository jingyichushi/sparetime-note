## 什么是事务传播行为？

事务传播行为（propagation behavior）指的就是当一个事务方法被另一个事务方法调用时，这个事务方法应该如何运行。

例如：methodA方法调用methodB方法时，methodB是继续在调用者methodA的事务中运行呢，还是为自己开启一个新事务运行，这就是由methodB的事务传播行为决定的。



 

## 事务的7种传播行为

 Spring在TransactionDefinition接口中规定了7种类型的事务传播行为。事务传播行为是Spring框架独有的事务增强特性。这是Spring为我们提供的强大的工具箱，使用事务传播行为可以为我们的开发工作提供许多便利。 

 7种事务传播行为如下： 

1.REQUIRED

如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，这是最常见的选择，也是Spring默认的事务传播行为。



2.SUPPORTS

支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。



3.MANDATORY

支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。



4.REQUIRES_NEW

创建新事务，无论当前存不存在事务，都创建新事务。



5.NOT_SUPPORTED

以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。



6.NEVER

以非事务方式执行，如果当前存在事务，则抛出异常。



7.NESTED

如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。





## 环境准备

先建两个表，用户表和用户角色表，一开始两个表里没有数据。

> 需要注意下，为了数据更直观，每次执行代码时 先清空下user和user_role表的数据。

user表：

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `password` varchar(255) DEFAULT NULL,
  `sex` int(11) DEFAULT NULL,
  `des` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

user_role表：

```sql
CREATE TABLE `user_role` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `user_id` int(11) DEFAULT NULL,
  `role_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



## 开始测试

### 1.REQUIRED测试

如果当前没有事务，就创建一个新事务，如果当前存在事务，就加入该事务，这是最常见的选择，也是Spring默认的事务传播行为。

**场景一：此场景外围方法没有开启事务。**

**a.验证方法**

两个实现类UserServiceImpl和UserRoleServiceImpl制定事物传播行为propagation=Propagation.REQUIRED，然后在测试方法中同时调用两个方法并在调用结束后抛出异常。



**b.主要代码**

TransactionalTest方法代码：

```java
    /**
     * 测试 PROPAGATION_REQUIRED
     */
    @Test
    public void test_PROPAGATION_REQUIRED() {
        // 增加用户表
        User user = new User();
        user.setName("用户A");
        user.setPassword("123456");
        userService.add(user);
        // 增加用户角色表
        UserRole userRole = new UserRole();
        userRole.setUserId(user.getId());
        userRole.setRoleId(200);
        userRoleService.add(userRole);
        //抛异常
        throw new RuntimeException();
    }
```

 UserServiceImpl代码： 

```java
    @Override
    @Transactional(propagation = Propagation.REQUIRED)
    public int add(User user) {
        return userMapper.insert(user);
    }
```

 UserRoleServiceImpl代码： 

```java
    /**
     * 增加用户角色
     */
    @Transactional(propagation = Propagation.REQUIRED)
    @Override
    public int add(UserRole userRole) {
        return userRoleMapper.insert(userRole);
    }
```

后续都以当前方法为标准。



**c.代码执行后数据库截图**

两张表数据都新增成功，截图如下：

![](http://img.bcoder.top/2019.11.11/1.png)



**场景二：**

此场景外围方法开启事务。

**a.主要代码**

TransactionalTest代码如下：

```java
    @Test
    @Transactional
    public void test_PROPAGATION_REQUIRED() {
        // 增加用户表
        User user = new User();
        user.setName("用户A");
        user.setPassword("123456");
        userService.add(user);
        // 增加用户角色表
        UserRole userRole = new UserRole();
        userRole.setUserId(user.getId());
        userRole.setRoleId(200);
        userRoleService.add(userRole);
        //抛异常
        throw new RuntimeException();
    }
```

**b.代码执行后数据库截图**

两张表数据都为空，截图如下:

![](http://img.bcoder.top/2019.11.11/2.png)



**c.结果分析**

外围方法开启事务，内部方法加入外围方法事务，外围方法回滚，内部方法也要回滚，所以两个记录都插入失败。

结论：以上结果证明在外围方法开启事务的情况下Propagation.REQUIRED修饰的内部方法会加入到外围方法的事务中，所以Propagation.REQUIRED修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。







### **2.SUPPORTS测试**

支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就以非事务执行。

**场景一：**

此场景外围方法没有开启事务。

**a.验证方法**

两个实现类UserServiceImpl和UserRoleServiceImpl制定事物传播行为propagation=Propagation.SUPPORTS，然后在测试方法中同时调用两个方法并在调用结束后抛出异常。

**b.主要代码**

UserServiceImpl代码：

```java
    @Override
    @Transactional(propagation = Propagation.SUPPORTS)
    public int add(User user) {
        return userMapper.insert(user);
    }
```

UserRoleServiceImpl代码：

```java
    /**
     * 增加用户角色
     */
    @Transactional(propagation = Propagation.SUPPORTS)
    @Override
    public int add(UserRole userRole) {
        return userRoleMapper.insert(userRole);
    }
```



**c.代码执行后数据库截图**

两张表数据都新增成功，截图如下:

![](http://img.bcoder.top/2019.11.11/3.png)



**d.结果分析**

外围方法未开启事务，插入用户表和用户角色表的方法以非事务的方式独立运行，外围方法异常不影响内部插入，所以两条记录都新增成功。



**场景二：**

此场景外围方法开启事务。

**a.主要代码**

test_PROPAGATION_SUPPORTS方法添加注解@Transactional即可。



**b.代码执行后数据库截图**

两张表数据都为空，截图如下：

![](http://img.bcoder.top/2019.11.11/4.png)



数据为空截图



**c.结果分析**

外围方法开启事务，内部方法加入外围方法事务，外围方法回滚，内部方法也要回滚，所以两个记录都插入失败。

结论：以上结果证明在外围方法开启事务的情况下Propagation.SUPPORTS修饰的内部方法会加入到外围方法的事务中，所以Propagation.SUPPORTS修饰的内部方法和外围方法均属于同一事务，只要一个方法回滚，整个事务均回滚。





### **3.MANDATORY测试**

支持当前事务，如果当前存在事务，就加入该事务，如果当前不存在事务，就抛出异常。

通过上面的测试，“支持当前事务，如果当前存在事务，就加入该事务”，这句话已经验证了，外层添加@Transactional注解后两条记录都新增失败，所以这个传播行为只测试下外层没有开始事务的场景。

**场景一：**

此场景外围方法没有开启事务。



**a.验证方法**

两个实现类UserServiceImpl和UserRoleServiceImpl制定事物传播行为propagation = Propagation.MANDATORY



**b.代码执行后数据库截图**

两张表数据都为空，截图如下：

![](http://img.bcoder.top/2019.11.11/5.png)



数据为空截图



**c.结果分析**



运行日志如下，可以发现在调用userService.add()时候已经报错了，所以两个表都没有新增数据，验证了“如果当前不存在事务，就抛出异常”。 

![](http://img.bcoder.top/2019.11.11/6.png)



### **4.NEW测试**

创建新事务，无论当前存不存在事务，都创建新事务。

这种情况每次都创建事务，所以我们验证一种情况即可。

**场景一：**

此场景外围方法开启事务。



**a.验证方法**

两个实现类UserServiceImpl和UserRoleServiceImpl制定事物传播行为propagation = Propagation.REQUIRES_NEW



**b. 代码执行后数据库截图**

两张表数据都新增成功，截图如下

![](http://img.bcoder.top/2019.11.11/7.png)

新增成功截图



**c.结果分析**

无论当前存不存在事务，都创建新事务，所以两个数据新增成功。





### **5.NOT_SUPPORTED测试**

以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。

**场景一：**

此场景外围方法不开启事务。

**a.验证方法**

两个实现类UserServiceImpl和UserRoleServiceImpl制定事物传播行为propagation = Propagation.NOT_SUPPORTED



**b.代码执行后数据库截图**

两张表数据都新增成功，截图如下：

![](http://img.bcoder.top/2019.11.11/7.png)

新增成功截图



**c.结果分析**

以非事务方式执行，所以两个数据新增成功。



**场景二：**

此场景外围方法开启事务。

**a,主要代码**

test_PROPAGATION_NOT_SUPPORTED方法添加注解@Transactional即可。



**b.代码执行后数据库截图**

两张表数据都新增成功，截图如下：

![](http://img.bcoder.top/2019.11.11/7.png)

 新增成功截图 



**c.结果分析**

如果当前存在事务，就把当前事务挂起，相当于以非事务方式执行，所以两个数据新增成功。



### **6.NEVER测试**

以非事务方式执行，如果当前存在事务，则抛出异常。

上面已经有类似情况，外层没有事务会以非事务的方式运行，两个表新增成功；有事务则抛出异常，两个表都都没有新增数据。



### **7.NESTED测试**

如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则按REQUIRED属性执行。

上面已经有类似情况，外层没有事务会以REQUIRED属性的方式运行，两个表新增成功；有事务但是用的是一个事务，方法最后抛出了异常导致回滚，两个表都都没有新增数据。