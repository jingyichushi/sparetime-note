# SpringBoot高级特性



> 引言

在我们进行开发应用时，除了要整合像MyBatis、Durid、logback等基础组件，还会去整合诸如缓存、消息队列、各种任务（异步任务、定时任务、邮件任务）等等。





## Spring Boot之缓存



### JSR107

JSR107定义了缓存的标准，主要有以下内容：

Java Caching定义了5个核心接口，分别是**CachingProvider**, **CacheManager**, **Cache**, **Entry** 和 **Expiry**。

+ CachingProvider定义了创建、配置、获取、管理和控制多个CacheManager。一个应用可以在运行期访问多个CachingProvider。
+ CacheManager定义了创建、配置、获取、管理和控制多个唯一命名的Cache，这些Cache存在于CacheManager的上下文中。一个CacheManager仅被一个CachingProvider所拥有。
+ Cache是一个类似Map的数据结构并临时存储以Key为索引的值。一个Cache仅被一个CacheManager所拥有。
+ Entry是一个存储在Cache中的key-value对。
+ Expiry 每一个存储在Cache中的条目有一个定义的有效期。一旦超过这个时间，条目为过期的状态。一旦过期，条目将不可访问、更新和删除。缓存有效期可以通过ExpiryPolicy设置。

我们可以用一个图来表示：

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/1.png)



###  Spring缓存抽象

Spring从3.1开始定义了org.springframework.cache.Cache和org.springframework.cache.CacheManager接口来统一不同的缓存技术；

并支持使用JCache（JSR-107）注解简化我们开发；



+ Cache接口为缓存的组件规范定义，包含缓存的各种操作集合；
+ Cache接口下Spring提供了各种xxxCache的实现；如RedisCache，EhCacheCache , ConcurrentMapCache等；
+ 每次调用需要缓存功能的方法时，Spring会检查检查指定参数的指定的目标方法是否已经被调用过；如果有就直接从缓存中获取方法调用后的结果，如果没有就调用方法并缓存结果后返回给用户。下次调用直接从缓存中获取。
+ 使用Spring缓存抽象时我们需要关注以下两点：
  + 确定方法需要被缓存以及他们的缓存策略
  + 从缓存中读取之前缓存存储的数据



我们可以用一个图来表示：

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/2.png)





### Spring缓存概念

| Cache          | 缓存接口，定义缓存操作。实现有：RedisCache、EhCacheCache、ConcurrentMapCache等 |
| :------------- | ------------------------------------------------------------ |
| CacheManager   | 缓存管理器，管理各种缓存（Cache）组件                        |
| @Cacheable     | 主要针对方法配置，能够根据方法的请求参数对其结果进行缓存     |
| @CacheEvict    | 清空缓存                                                     |
| @CachePut      | 保证方法被调用，又希望结果被缓存。                           |
| @EnableCaching | 开启基于注解的缓存                                           |
| keyGenerator   | 缓存数据时key生成策略                                        |
| serialize      | 缓存数据时value序列化策略                                    |



**@Cacheable/@CachePut/@CacheEvict主要的参数**

| value                               | 缓存的名称，在 spring   配置文件中定义，必须指定至少一个     | 例如：@Cacheable(value=”mycache”) 或者       @Cacheable(value={”cache1”,”cache2”} |
| ----------------------------------- | :----------------------------------------------------------- | ------------------------------------------------------------ |
| key                                 | 缓存的   key，可以为空，如果指定要按照   SpEL 表达式编写，如果不指定，则缺省按照方法的所有参数进行组合 | 例如：` @Cacheable(value=”testcache”,key=”#userName”)   `    |
| condition                           | 缓存的条件，可以为空，使用   SpEL 编写，返回   true 或者 false，只有为   true 才进行缓存/清除缓存，在调用方法之前之后都能判断 | 例如：`      @Cacheable(value=”testcache”,condition=”#userName.length()>2”)   ` |
| allEntries   (@CacheEvict   )       | 是否清空所有缓存内容，缺省为   false，如果指定为 true，则方法调用后将立即清空所有缓存 | 例如：`@CachEvict(value=”testcache”,allEntries=true)`        |
| beforeInvocation   (@CacheEvict)    | 是否在方法执行前就清空，缺省为   false，如果指定为 true，则在方法还没有执行的时候就清空缓存，缺省情况下，如果方法执行抛出异常，则不会清空缓存 | 例如：`@CachEvict(value=”testcache”，beforeInvocation=true)` |
| unless   (@CachePut)   (@Cacheable) | 用于否决缓存的，不像condition，该表达式只在方法执行之后判断，此时可以拿到返回值result进行判断。条件为true不会缓存，fasle才缓存 | 例如：`@Cacheable(value=”testcache”,unless=”#result== null”) ` |





**Cache SpEL available** 

| 名字          | 位置               | 描述                                                         | 示例                   |
| ------------- | ------------------ | ------------------------------------------------------------ | ---------------------- |
| methodName    | root object        | 当前被调用的方法名                                           | `#root.methodName`     |
| method        | root object        | 当前被调用的方法                                             | `#root.method.name`    |
| target        | root object        | 当前被调用的目标对象                                         | `#root.target`         |
| targetClass   | root object        | 当前被调用的目标对象类                                       | `#root.targetClass`    |
| args          | root object        | 当前被调用的方法的参数列表                                   | `#root.args[0]`        |
| caches        | root object        | 当前方法调用使用的缓存列表（如@Cacheable(value={"cache1",   "cache2"})），则有两个cache | `#root.caches[0].name` |
| argument name | evaluation context | 方法参数的名字. 可以直接 #参数名 ，也可以使用 #p0或#a0 的形式，0代表参数的索引； | `#iban 、 #a0 、  #p0` |
| result        | evaluation context | 方法执行后的返回值（仅当方法执行之后的判断有效，如‘unless’，’cache put’的表达式 ’cache evict’的表达式beforeInvocation=false） | `#result`              |



### 缓存实战

1.引入pom

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-cache</artifactId>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
		<dependency>
			<groupId>org.mybatis.spring.boot</groupId>
			<artifactId>mybatis-spring-boot-starter</artifactId>
			<version>1.3.2</version>
		</dependency>
		<dependency>
			<groupId>mysql</groupId>
			<artifactId>mysql-connector-java</artifactId>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
	</dependencies>
```

2.开启缓存

在XXXApplication.java中加上注解：@EnableCaching



3.创建数据表及其对应的实体bean：

```sql
CREATE TABLE `department` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `departmentName` varchar(255) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

-- ----------------------------
-- Table structure for employee
-- ----------------------------
DROP TABLE IF EXISTS `employee`;
CREATE TABLE `employee` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `lastName` varchar(255) DEFAULT NULL,
  `email` varchar(255) DEFAULT NULL,
  `gender` int(2) DEFAULT NULL,
  `d_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```



4.创建Mapper

```java
Component
@Mapper
public interface EmployeeMapper {
    @Select("select * from employee where id=#{id}")
    public Employee getEmpById(Integer id);

    @Update("update employee set lastName=#{lastName},email=#{email},gender=#{gender},d_id=#{d_id} where id=#{id}")
    public void updateEmp(Employee employee);

    @Delete("Delete from employee where id=#{id}")
    public void deleteEmpById(Integer id);

    @Insert("insert employee(lastName,email,gender,d_id) values(#{lastName},#{email},#{gender},#{dId}")
    public void insertEmployee(Employee employee);
}
```



5.创建Service(主要内容在注释)

```java

import com.zln.cache.bean.Employee;
import com.zln.cache.mapper.EmployeeMapper;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.cache.annotation.*;
import org.springframework.stereotype.Service;

@CacheConfig(cacheNames="emp"/*,cacheManager = "employeeCacheManager"*/) //抽取缓存的公共配置
@Service
public class EmployeeService {

    @Autowired
    EmployeeMapper employeeMapper;

    /**
     * 将方法的运行结果进行缓存；以后再要相同的数据，直接从缓存中获取，不用调用方法；
     * CacheManager管理多个Cache组件的，对缓存的真正CRUD操作在Cache组件中，每一个缓存组件有自己唯一一个名字；
     *

     *
     * 原理：
     *   1、自动配置类；CacheAutoConfiguration
     *   2、缓存的配置类
     *   org.springframework.boot.autoconfigure.cache.GenericCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.JCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.EhCacheCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.HazelcastCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.InfinispanCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CouchbaseCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.RedisCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.CaffeineCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.GuavaCacheConfiguration
     *   org.springframework.boot.autoconfigure.cache.SimpleCacheConfiguration【默认】
     *   org.springframework.boot.autoconfigure.cache.NoOpCacheConfiguration
     *   3、哪个配置类默认生效：SimpleCacheConfiguration；
     *
     *   4、给容器中注册了一个CacheManager：ConcurrentMapCacheManager
     *   5、可以获取和创建ConcurrentMapCache类型的缓存组件；他的作用将数据保存在ConcurrentMap中；
     *
     *   运行流程：
     *   @Cacheable：
     *   1、方法运行之前，先去查询Cache（缓存组件），按照cacheNames指定的名字获取；
     *      （CacheManager先获取相应的缓存），第一次获取缓存如果没有Cache组件会自动创建。
     *   2、去Cache中查找缓存的内容，使用一个key，默认就是方法的参数；
     *      key是按照某种策略生成的；默认是使用keyGenerator生成的，默认使用SimpleKeyGenerator生成key；
     *          SimpleKeyGenerator生成key的默认策略；
     *                  如果没有参数；key=new SimpleKey()；
     *                  如果有一个参数：key=参数的值
     *                  如果有多个参数：key=new SimpleKey(params)；
     *   3、没有查到缓存就调用目标方法；
     *   4、将目标方法返回的结果，放进缓存中
     *
     *   @Cacheable标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，
     *   如果没有就运行方法并将结果放入缓存；以后再来调用就可以直接使用缓存中的数据；
     *
     *   核心：
     *      1）、使用CacheManager【ConcurrentMapCacheManager】按照名字得到Cache【ConcurrentMapCache】组件
     *      2）、key使用keyGenerator生成的，默认是SimpleKeyGenerator
     *
     *
     *   几个属性：
     *      cacheNames/value：指定缓存组件的名字;将方法的返回结果放在哪个缓存中，是数组的方式，可以指定多个缓存；
     *
     *      key：缓存数据使用的key；可以用它来指定。默认是使用方法参数的值  1-方法的返回值
     *              编写SpEL； #i d;参数id的值   #a0  #p0  #root.args[0]
     *              getEmp[2]
     *
     *      keyGenerator：key的生成器；可以自己指定key的生成器的组件id
     *              key/keyGenerator：二选一使用;
     *
     *
     *      cacheManager：指定缓存管理器；或者cacheResolver指定获取解析器
     *
     *      condition：指定符合条件的情况下才缓存；
     *              ,condition = "#id>0"
     *          condition = "#a0>1"：第一个参数的值》1的时候才进行缓存
     *
     *      unless:否定缓存；当unless指定的条件为true，方法的返回值就不会被缓存；可以获取到结果进行判断
     *              unless = "#result == null"
     *              unless = "#a0==2":如果第一个参数的值是2，结果不缓存；
     *      sync：是否使用异步模式
     * @param id
     * @return
     *
     */
    @Cacheable(value = {"emp"}/*,keyGenerator = "myKeyGenerator",condition = "#a0>1",unless = "#a0==2"*/)
    public Employee getEmp(Integer id){
        System.out.println("查询"+id+"号员工");
        Employee emp = employeeMapper.getEmpById(id);
        return emp;
    }

    /**
     * @CachePut：既调用方法，又更新缓存数据；同步更新缓存
     * 修改了数据库的某个数据，同时更新缓存；
     * 运行时机：
     *  1、先调用目标方法
     *  2、将目标方法的结果缓存起来
     *
     * 测试步骤：
     *  1、查询1号员工；查到的结果会放在缓存中；
     *          key：1  value：lastName：张三
     *  2、以后查询还是之前的结果
     *  3、更新1号员工；【lastName:zhangsan；gender:0】
     *          将方法的返回值也放进缓存了；
     *          key：传入的employee对象  值：返回的employee对象；
     *  4、查询1号员工？
     *      应该是更新后的员工；
     *          key = "#employee.id":使用传入的参数的员工id；
     *          key = "#result.id"：使用返回后的id
     *             @Cacheable的key是不能用#result
     *      为什么是没更新前的？【1号员工没有在缓存中更新】
     *
     */
    @CachePut(/*value = "emp",*/key = "#result.id")
    public Employee updateEmp(Employee employee){
        System.out.println("updateEmp:"+employee);
        employeeMapper.updateEmp(employee);
        return employee;
    }

    /**
     * @CacheEvict：缓存清除
     *  key：指定要清除的数据
     *  allEntries = true：指定清除这个缓存中所有的数据
     *  beforeInvocation = false：缓存的清除是否在方法之前执行
     *      默认代表缓存清除操作是在方法执行之后执行;如果出现异常缓存就不会清除
     *
     *  beforeInvocation = true：
     *      代表清除缓存操作是在方法运行之前执行，无论方法是否出现异常，缓存都清除
     *
     *
     */
    @CacheEvict(value="emp",beforeInvocation = true/*key = "#id",*/)
    public void deleteEmp(Integer id){
        System.out.println("deleteEmp:"+id);
        //employeeMapper.deleteEmpById(id);
        int i = 10/0;
    }

    // @Caching 定义复杂的缓存规则
    @Caching(
         cacheable = {
             @Cacheable(/*value="emp",*/key = "#lastName")
         },
         put = {
             @CachePut(/*value="emp",*/key = "#result.id"),
             @CachePut(/*value="emp",*/key = "#result.email")
         }
    )
    public Employee getEmpByLastName(String lastName){
        return employeeMapper.getEmpByLastName(lastName);
    }
}

```



### 整合Redis作为缓存

1.引入pom

```xml
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-redis</artifactId>
		</dependency>
```

2.yml配置Redis相关配置



3.创建RedisCacheManager的实现（已上面的例子为基础）

```java
@Configuration
public class MyRedisConfig {
    @Bean
    public RedisTemplate<Object, Employee> empredisTemplate(
            RedisConnectionFactory redisConnectionFactory) throws Exception{
        RedisTemplate<Object,Employee> template=new RedisTemplate<Object, Employee>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Employee> ser=new Jackson2JsonRedisSerializer<Employee>(Employee.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    @Bean
    public RedisTemplate<Object, Department> deptredisTemplate(
            RedisConnectionFactory redisConnectionFactory) throws Exception{
        RedisTemplate<Object,Department> template=new RedisTemplate<Object, Department>();
        template.setConnectionFactory(redisConnectionFactory);
        Jackson2JsonRedisSerializer<Department> ser=new Jackson2JsonRedisSerializer<Department>(Department.class);
        template.setDefaultSerializer(ser);
        return template;
    }
    @Primary//必须设置一个默认的
    @Bean
    public RedisCacheManager empoyeeCacheManager(RedisTemplate<Object,Employee> employeeRedisTemplate){
        RedisCacheManager redisCacheManager=new RedisCacheManager(employeeRedisTemplate);
        redisCacheManager.setUsePrefix(true);
        return redisCacheManager;
    }
    @Bean
    public RedisCacheManager deptCacheManager(RedisTemplate<Object,Department> deptloyeeRedisTemplate){
        RedisCacheManager redisCacheManager=new RedisCacheManager(deptloyeeRedisTemplate);
        redisCacheManager.setUsePrefix(true);
        return redisCacheManager;
    }
}

```



4.在使用缓存注解时，指定cacheManager即可，当然也可以直接代码调用cacheManager，示例如下：

```java
@RestController
public class DeptController {

    @Autowired
    @Qualifier("deptCacheManager")
    private RedisCacheManager deptCacheManager;
    
    @Autowired
    private DeptService deptService;

    @GetMapping("/dept/{id}")
    public Department getDeptById(@PathVariable("id") Integer id){
        return deptService.getDeptById(id);
    }

    @GetMapping("/depts/{id}")
    public Department getDeptByIds(@PathVariable("id") Integer id){
        System.out.println("查询部门");
        Department department=deptService.getDeptById(1);
        Cache dept = deptCacheManager.getCache("dept");
        dept.put("dept:1",department);
        return department;
    }

}
```



### 缓存原理

**1.自动配置类：CacheAutoConfiguration.java**

```java
@Configuration
@ConditionalOnClass({CacheManager.class})
@ConditionalOnBean({CacheAspectSupport.class})
@ConditionalOnMissingBean(
    value = {CacheManager.class},
    name = {"cacheResolver"}
)
@EnableConfigurationProperties({CacheProperties.class})
@AutoConfigureAfter({CouchbaseAutoConfiguration.class, HazelcastAutoConfiguration.class, HibernateJpaAutoConfiguration.class, RedisAutoConfiguration.class})
//给容器中导入缓存要用的一些组件
@Import({CacheAutoConfiguration.CacheConfigurationImportSelector.class})
public class CacheAutoConfiguration {
```



```java
static class CacheConfigurationImportSelector implements ImportSelector {
    CacheConfigurationImportSelector() {
    }
 
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        CacheType[] types = CacheType.values();
        //给容器中导入缓存要用的一些组件
        String[] imports = new String[types.length];
 
        for(int i = 0; i < types.length; ++i) {
            imports[i] = CacheConfigurations.getConfigurationClass(types[i]);
        }
 
        return imports;
    }
}
```


**2.导入缓存自动配置类组件**

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/3.png)



**3.默认生效配置类SimpleCacheConfiguration**

yml中 debug: true进行查看

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/4.png)



**4.SimpleCacheConfiguration给容器中注册一个CacheManager：ConcurrentMapCacheManager**

ConcurrentMapCacheManager.java

```java

    @Nullable
    public Cache getCache(String name) {
        Cache cache = (Cache)this.cacheMap.get(name);
        if (cache == null && this.dynamic) {
            ConcurrentMap var3 = this.cacheMap;
            synchronized(this.cacheMap) {
                cache = (Cache)this.cacheMap.get(name);
                if (cache == null) {
                    cache = this.createConcurrentMapCache(name);
                    this.cacheMap.put(name, cache);
                }
            }
        }
 
        return cache;
    }
```

ConcurrentMapCache.java    

```java
private final ConcurrentMap<Object, Object> store;
@Nullable
protected Object lookup(Object key) {
    return this.store.get(key);
}
public void put(Object key, @Nullable Object value) {
    this.store.put(key, this.toStoreValue(value));
}
```



### 缓存运行流程

标注的方法执行之前先来检查缓存中有没有这个数据，默认按照参数的值作为key去查询缓存，如果没有就运行方法并将结果放入缓存，以后再来调用就可以直接使用缓存中的数据。

**一.核心**

+ 使用CacheManager（ConcurrentMapCacheManager）按照名字得到Cache（ConcurrentMapCache）组件
+ key使用KeyGenerator生成的，默认是使用SimpleKeyGenerator生成key



**二.流程**

方法运行之前，先去查询Cache（缓存组件），按照cacheName指定的名字获取CacheManager先获取相应的缓存，第一次获取缓存如果没有cache组件会自动创建

**ConcurrentMapCacheManager.java**

```java
@Nullable
public Cache getCache(String name) {
    Cache cache = (Cache)this.cacheMap.get(name);
    if (cache == null && this.dynamic) {
        ConcurrentMap var3 = this.cacheMap;
        synchronized(this.cacheMap) {
            cache = (Cache)this.cacheMap.get(name);
            if (cache == null) {
                cache = this.createConcurrentMapCache(name);
                this.cacheMap.put(name, cache);
            }
        }
    }
    return cache;
}
```
2.使用一个key去cache中查找缓存的内容,key默认就是方法的参数key是按照某种策略生成的，使用KeyGenerator生成的，默认是使用SimpleKeyGenerator生成key

SimpleKeyGenerator生成key的默认策略：

+ 如果没有参数 key = new SimpleKey();

+ 如果有一个参数 key = 参数的值;

+ 如果有多个参数 key = new SimpleKey(params);

ConcurrentMapCache.java 

```java
@Nullable
protected Object lookup(Object key) {
    return this.store.get(key);
}
```



CacheAspectSupport.java

```java
@Nullable
private ValueWrapper findCachedItem(Collection<CacheAspectSupport.CacheOperationContext> contexts) {
    Object result = CacheOperationExpressionEvaluator.NO_RESULT;
    Iterator var3 = contexts.iterator();

    while(var3.hasNext()) {
        CacheAspectSupport.CacheOperationContext context = (CacheAspectSupport.CacheOperationContext)var3.next();
        if (this.isConditionPassing(context, result)) {
            Object key = this.generateKey(context, result);
            ValueWrapper cached = this.findInCaches(context, key);
            if (cached != null) {
                return cached;
            }

            if (this.logger.isTraceEnabled()) {
                this.logger.trace("No cache entry for key '" + key + "' in cache(s) " + context.getCacheNames());
            }
        }
    }

    return null;
}
```



SimpleKeyGenerator.java

```java
public static Object generateKey(Object... params) {
    if (params.length == 0) {
        return SimpleKey.EMPTY;
    } else {
        if (params.length == 1) {
            Object param = params[0];
            if (param != null && !param.getClass().isArray()) {
                return param;
            }
        }

        return new SimpleKey(params);
    }
}
```

3.没有查到缓存就调用目标方法

```java
@Cacheable(cacheNames = "emp")
public Employee getEmp(Integer id){
    System.out.println("查询"+id+"号员工");
    Employee emp = employeeMapper.getEmpById(id);
    return emp;
}

```



4.将目标方法返回到结果放进缓存中

ConcurrentMapCache.java   

```java
public void put(Object key, @Nullable Object value) {
	this.store.put(key, this.toStoreValue(value));
}
```



## Spring Boot之消息队列

### 消息队列概述



**1.消息队列简介**

+ 大多应用中，可通过消息服务中间件来提升系统异步通信、扩展解耦能力

+ 消息服务中两个重要概念：消息代理（message broker）和目的地（destination）
  + 当消息发送者发送消息以后，将由消息代理接管，消息代理保证消息传递到指定目的地。

+ 消息队列主要有两种形式的目的地
  + **队列**（queue）：点对点消息通信（point-to-point）
  + **主题**（topic）：发布（publish）/订阅（subscribe）消息通信

+ **点对点式**：
  + 消息发送者发送消息，消息代理将其放入一个队列中，消息接收者从队列中获取消息内容，消息读取后被移出队列
  + 消息只有唯一的发送者和接受者，但并不是说只能有一个接收者

+ **发布订阅式**：发送者（发布者）发送消息到主题，多个接收者（订阅者）监听（订阅）这个主题，那么就会在消息到达时同时收到消息
+ **JMS**（Java Message Service）JAVA消息服务：基于JVM消息代理的规范。
  + ActiveMQ、HornetMQ是JMS实现
+ **AMQP**（Advanced Message Queuing Protocol）：高级消息队列协议，也是一个消息代理的规范，兼容JMS
  + RabbitMQ是AMQP的实现

+ **Spring支持**
  + spring-jms提供了对JMS的支持
  + spring-rabbit提供了对AMQP的支持
  + 需要ConnectionFactory的实现来连接消息代理
  + 提供JmsTemplate、RabbitTemplate来发送消息
  + @JmsListener（JMS）、@RabbitListener（AMQP）注解在方法上监听消息代理发布的消息
  + @EnableJms、@EnableRabbit开启支持
+ Spring Boot自动配置
  + JmsAutoConfiguration
  + RabbitAutoConfiguration

**JMS  VS  AMQP**   

|              | JMS                                                          | AMQP                                                         |
| ------------ | ------------------------------------------------------------ | :----------------------------------------------------------- |
| 定义         | Java   api                                                   | 网络线级协议                                                 |
| 跨语言       | 否                                                           | 是                                                           |
| 跨平台       | 否                                                           | 是                                                           |
| Model        | 提供两种消息模型：   （1）、Peer-2-Peer   （2）、Pub/sub     | 提供了五种消息模型：   （1）、direct   exchange   （2）、fanout   exchange   （3）、topic   change   （4）、headers   exchange   （5）、system   exchange   本质来讲，后四种和JMS的pub/sub模型没有太大差别，仅是在路由机制上做了更详细的划分； |
| 支持消息类型 | 多种消息类型：   TextMessage   MapMessage   BytesMessage   StreamMessage   ObjectMessage   Message   （只有消息头和属性） | byte[]   当实际应用时，有复杂的消息，可以将消息序列化后发送。 |
| 综合评价     | JMS   定义了JAVA   API层面的标准；在java体系中，多个client均可以通过JMS进行交互，不需要应用修改代码，但是其对跨平台的支持较差； | AMQP定义了wire-level层的协议标准；天然具有跨平台、跨语言特性。 |





**2.消息队列的好处**

①异步处理

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/5.png)

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/6.png)

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/7.png)

②应用解耦

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/8.png)

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/9.png)

③流量削峰

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/10.png)





### RabbitMQ简介

RabbitMQ是一个由erlang开发的AMQP(Advanved Message Queue Protocol)的开源实现。



核心概念如下：

+ **Message**

  消息，消息是不具名的，它由消息头和消息体组成。消息体是不透明的，而消息头则由一系列的可选属性组成，这些属性包括routing-key（路由键）、priority（相对于其他消息的优先权）、delivery-mode（指出该消息可能需要持久性存储）等。

+ **Publisher**

  消息的生产者，也是一个向交换器发布消息的客户端应用程序。

+ **Exchange**

  交换器，用来接收生产者发送的消息并将这些消息路由给服务器中的队列。

  Exchange有4种类型：direct(默认)，fanout, topic, 和headers，不同类型的Exchange转发消息的策略有所区别

+ **Queue**

  消息队列，用来保存消息直到发送给消费者。它是消息的容器，也是消息的终点。一个消息可投入一个或多个队列。消息一直在队列里面，等待消费者连接到这个队列将其取走。

+ **Binding**

  绑定，用于消息队列和交换器之间的关联。一个绑定就是基于路由键将交换器和消息队列连接起来的路由规则，所以可以将交换器理解成一个由绑定构成的路由表。

  Exchange 和Queue的绑定可以是多对多的关系。

+ **Connection**

  网络连接，比如一个TCP连接。

+ **Channel**

  信道，多路复用连接中的一条独立的双向数据流通道。信道是建立在真实的TCP连接内的虚拟连接，AMQP 命令都是通过信道发出去的，不管是发布消息、订阅队列还是接收消息，这些动作都是通过信道完成。因为对于操作系统来说建立和销毁 TCP 都是非常昂贵的开销，所以引入了信道的概念，以复用一条 TCP 连接。

+ **Consumer**

  消息的消费者，表示一个从消息队列中取得消息的客户端应用程序。

+ **Virtual Host**

  虚拟主机，表示一批交换器、消息队列和相关对象。虚拟主机是共享相同的身份认证和加密环境的独立服务器域。每个 vhost 本质上就是一个 mini 版的 RabbitMQ 服务器，拥有自己的队列、交换器、绑定和权限机制。vhost 是 AMQP 概念的基础，必须在连接时指定，RabbitMQ 默认的 vhost 是 / 。

+ **Broker**

  表示消息队列服务器实体

总结起来如下图所示：

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/11.png)



### RabbitMQ运行机制

**1.AMQP 中的消息路由**

AMQP 中消息的路由过程和 Java 开发者熟悉的 JMS 存在一些差别，AMQP 中增加了 **Exchange** 和 **Binding** 的角色。生产者把消息发布到 Exchange 上，消息最终到达队列并被消费者接收，而 Binding 决定交换器的消息应该发送到那个队列。

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/12.png)



**2.Exchange类型**

Exchange分发消息时根据类型的不同分发策略有区别，目前共四种类型：**direct**、**fanout**、**topic**、**headers** 。headers 匹配 AMQP 消息的 header 而不是路由键， **headers 交换器和 direct 交换器完全一致，但性能差很多，目前几乎用不到**了，所以直接看另外三种类型：

**①direct**

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/13.png)

消息中的路由键（routing key）如果和 Binding 中的 binding key 一致，交换器就将消息发到对应的队列中。路由键与队列名完全匹配，如果一个队列绑定到交换机要求路由键为“dog”，则只转发 routing key 标记为“dog”的消息，不会转发“dog.puppy”，也不会转发“dog.guard”等等。它是完全匹配、单播的模式。



**②fanout**

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/14.png)

每个发到 fanout 类型交换器的消息都会分到所有绑定的队列上去。fanout交换器不处理路由键，只是简单的将队列绑定到交换器上，每个发送到交换器的消息都会被转发到与该交换器绑定的所有队列上。很像子网广播，每台子网内的主机都获得了一份复制的消息。fanout类型转发消息是最快的。



**③topic**

![SpringBoot高级特性](http://img.bcoder.top/2019.01.01/15.png)

topic 交换器通过模式匹配分配消息的路由键属性，将路由键和某个模式进行匹配，此时队列需要绑定到一个模式上。它将路由键和绑定键的字符串切分成单词，这些单词之间用点隔开。它同样也会识别两个通配符：符号“`#`”和符号“`*`”。`#`匹配0个或多个单词，`*`匹配一个单词。





### 整合RabbitMQ作为缓存

1. **引入 spring-boot-starter-amqp**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```
2. **application.yml配置**

```properties
#主机地址
spring.rabbitmq.host=192.168.3.224
spring.rabbitmq.username=guest
spring.rabbitmq.password=guest
spring.rabbitmq.port=5672
```



3.配置消息转换器（默认jdk实现）

```java
//自定义MessageConverter：将数据自动转为josn发送出去
@Configuration
public class MyAMQPConfig {
    @Bean
    public MessageConverter messageConverter(){
        return new Jackson2JsonMessageConverter();
    }
}
```



**4.测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot02AmqpApplicationTests {
    @Autowired
    RabbitTemplate rabbitTemplate;
    /**
     * 发送消息
     * 如何将数据自动转为josn发送出去:自定义messageConverter
     * */
    @Test
    public void contextLoads() {
        //Message需要自己构造一个；定义消息体内容和消息头
        //rabbitTemplate.send(exchange,routeKey,message);
        //1、单播（点对点）
        //object默认当成消息体，只需要传入要发送的对象，自动序列化发送给rabbitmq;
        //rabbitTemplate.convertAndSend(exchange,routeKey,object);
        Map<String,Object> map = new HashMap<>();
        map.put("msg","这是第一个消息");
        map.put("data",Arrays.asList("helloworld",123,true));
        //对象被默认序列化以后发送出去
        //rabbitTemplate.convertAndSend("exchange.direct","zln.news",map);
        rabbitTemplate.convertAndSend("exchange.direct","zln.news",new Book("西游记","吴承恩"));
        //2、广播
        rabbitTemplate.convertAndSend("exchange.fanout","",new Book("三国演义","罗贯中"));
    }
 
    /**
     * 接收消息
     * */
    @Test
    public void receive() {
        Object o = rabbitTemplate.receiveAndConvert("zln.news");
        System.out.println(o.getClass());
        System.out.println(o);
    }
}
```





**5.注解使用:@EnableRabbit+@RabbitListener**

```java
@EnableRabbit //开启基于注解的RabbitMQ
@SpringBootApplication
public class Springboot02AmqpApplication {
    public static void main(String[] args) {
        SpringApplication.run(Springboot02AmqpApplication.class, args);
    }
}
```

```java
@Service
public class BookService {
    //监听队列消息
    @RabbitListener(queues = "zln.news")
    public  void  receive(Book book){
        System.out.println("收到消息："+book);
    }
 
    //监听消息相关信息
    @RabbitListener(queues = "zln.news")
    public  void  receive02(Message message){
        System.out.println(message.getBody());
        System.out.println(message.getMessageProperties());
    }
}

```



**6.AmqpAdmin：创建和删除Queue、Exchange、Binding**

```java
    @Autowired
    AmqpAdmin amqpAdmin;
 
    @Test
    public void createExchange(){
        //创建exchange
        //amqpAdmin.declareExchange(new DirectExchange("amqpadmin.exchange"));
        //创建queue
        //amqpAdmin.declareQueue(new Queue("amqpadmin.queue",true));
        //创建Binding
        amqpAdmin.declareBinding(new Binding("amqpadmin.queue",Binding.DestinationType.QUEUE,"amqpadmin.exchange","amqpa.haha",null));
        System.out.println("创建完成");
    }

```



### 整合原理

1.自动配置类RabbitAutoConfiguration.java

**①自动配置类连接工厂CachingConnectionFactory**

```java
@Configuration
@ConditionalOnClass({ RabbitTemplate.class, Channel.class })
@EnableConfigurationProperties(RabbitProperties.class)
@Import(RabbitAnnotationDrivenConfiguration.class)
public class RabbitAutoConfiguration {
                //有自动配置类连接工厂
		@Bean
		public CachingConnectionFactory rabbitConnectionFactory(
				RabbitProperties properties,
				ObjectProvider<ConnectionNameStrategy> connectionNameStrategy)
				throws Exception {
			PropertyMapper map = PropertyMapper.get();
                        //配置连接工厂属性
			CachingConnectionFactory factory = new CachingConnectionFactory(
					getRabbitConnectionFactoryBean(properties).getObject());
			map.from(properties::determineAddresses).to(factory::setAddresses);
			map.from(properties::isPublisherConfirms).to(factory::setPublisherConfirms);
			map.from(properties::isPublisherReturns).to(factory::setPublisherReturns);
			RabbitProperties.Cache.Channel channel = properties.getCache().getChannel();
			map.from(channel::getSize).whenNonNull().to(factory::setChannelCacheSize);
			map.from(channel::getCheckoutTimeout).whenNonNull().as(Duration::toMillis)
					.to(factory::setChannelCheckoutTimeout);
			RabbitProperties.Cache.Connection connection = properties.getCache()
					.getConnection();
			map.from(connection::getMode).whenNonNull().to(factory::setCacheMode);
			map.from(connection::getSize).whenNonNull()
					.to(factory::setConnectionCacheSize);
			map.from(connectionNameStrategy::getIfUnique).whenNonNull()
					.to(factory::setConnectionNameStrategy);
			return factory;
		}
    
 		//配置自动配置的连接   配置RabbitProperties里的属性
		private RabbitConnectionFactoryBean getRabbitConnectionFactoryBean(
				RabbitProperties properties) throws Exception {
			PropertyMapper map = PropertyMapper.get();
			RabbitConnectionFactoryBean factory = new RabbitConnectionFactoryBean();
			map.from(properties::determineHost).whenNonNull().to(factory::setHost);
			map.from(properties::determinePort).to(factory::setPort);
			map.from(properties::determineUsername).whenNonNull()
					.to(factory::setUsername);
			map.from(properties::determinePassword).whenNonNull()
					.to(factory::setPassword);
			map.from(properties::determineVirtualHost).whenNonNull()
					.to(factory::setVirtualHost);
			map.from(properties::getRequestedHeartbeat).whenNonNull()
					.asInt(Duration::getSeconds).to(factory::setRequestedHeartbeat);
			RabbitProperties.Ssl ssl = properties.getSsl();
			if (ssl.isEnabled()) {
				factory.setUseSSL(true);
				map.from(ssl::getAlgorithm).whenNonNull().to(factory::setSslAlgorithm);
				map.from(ssl::getKeyStoreType).to(factory::setKeyStoreType);
				map.from(ssl::getKeyStore).to(factory::setKeyStore);
				map.from(ssl::getKeyStorePassword).to(factory::setKeyStorePassphrase);
				map.from(ssl::getTrustStoreType).to(factory::setTrustStoreType);
				map.from(ssl::getTrustStore).to(factory::setTrustStore);
				map.from(ssl::getTrustStorePassword).to(factory::setTrustStorePassphrase);
				map.from(ssl::isValidateServerCertificate).to((validate) -> factory
						.setSkipServerCertificateValidation(!validate));
				map.from(ssl::getVerifyHostname).when(Objects::nonNull)
						.to(factory::setEnableHostnameVerification);
				if (ssl.getVerifyHostname() == null && CAN_ENABLE_HOSTNAME_VERIFICATION) {
					factory.setEnableHostnameVerification(true);
				}
			}
			map.from(properties::getConnectionTimeout).whenNonNull()
					.asInt(Duration::toMillis).to(factory::setConnectionTimeout);
			factory.afterPropertiesSet();
			return factory;
		}
 
	}

```



**②RabbitProperties：封装了RabbitMQ的所有配置**

```java
//RabbitProperties封装了RabbitMQ的所有配置
@ConfigurationProperties(prefix = "spring.rabbitmq")
public class RabbitProperties {
```



**③rabbitTemplate：给RabbitMQ发送和接受消息的**

```java
@Bean
@ConditionalOnSingleCandidate(ConnectionFactory.class)
@ConditionalOnMissingBean
public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
    PropertyMapper map = PropertyMapper.get();
    RabbitTemplate template = new RabbitTemplate(connectionFactory);
    MessageConverter messageConverter = this.messageConverter.getIfUnique();
    if (messageConverter != null) {
        //如果有自己设置的messageConverter消息转换器，也会在这设置进来
        template.setMessageConverter(messageConverter);
    }
    template.setMandatory(determineMandatoryFlag());
    RabbitProperties.Template properties = this.properties.getTemplate();
    if (properties.getRetry().isEnabled()) {
        template.setRetryTemplate(createRetryTemplate(properties.getRetry()));
    }
    map.from(properties::getReceiveTimeout).whenNonNull().as(Duration::toMillis)
        .to(template::setReceiveTimeout);
    map.from(properties::getReplyTimeout).whenNonNull().as(Duration::toMillis)
        .to(template::setReplyTimeout);
    map.from(properties::getExchange).to(template::setExchange);
    map.from(properties::getRoutingKey).to(template::setRoutingKey);
    return template;
}
```


**④消息转换器**

RabbitTemplate.java

```java
//默认使用的是SimpleMessageConverter消息转换器
private volatile MessageConverter messageConverter = new SimpleMessageConverter();

```



SimpleMessageConverter.java

```java
 //默认序列化数据的时候，按照jdk序列化规则来序列化
    protected Message createMessage(Object object, MessageProperties messageProperties) throws MessageConversionException {
        byte[] bytes = null;
        if (object instanceof byte[]) {
            bytes = (byte[])((byte[])object);
            messageProperties.setContentType("application/octet-stream");
        } else if (object instanceof String) {
            try {
                bytes = ((String)object).getBytes(this.defaultCharset);
            } catch (UnsupportedEncodingException var6) {
                throw new MessageConversionException("failed to convert to Message content", var6);
            }
 
            messageProperties.setContentType("text/plain");
            messageProperties.setContentEncoding(this.defaultCharset);
        } else if (object instanceof Serializable) {
            try {
                bytes = SerializationUtils.serialize(object);
            } catch (IllegalArgumentException var5) {
                throw new MessageConversionException("failed to convert to serialized Message content", var5);
            }
 
            messageProperties.setContentType("application/x-java-serialized-object");
        }
 
        if (bytes != null) {
            messageProperties.setContentLength((long)bytes.length);
            return new Message(bytes, messageProperties);
        } else {
            throw new IllegalArgumentException(this.getClass().getSimpleName() + " only supports String, byte[] and Serializable payloads, received: " + object.getClass().getName());
        }
    }

```

## Spring Boot之任务



### 异步任务

在Java应用中，绝大多数情况下都是通过同步的方式来实现交互处理的;但是在处理与第三方系统交互的时候，容易造成响应迟缓的情况，之前大部分都是使用多线程来完成此类任务，其实，在Spring 3.x之后，就已经内置了@Async来完美解决这个问题。

两个注解:**@EnableAysnc、@Aysnc**

**1.Service**

```java
@Service
public class AsyncService {
    //告诉Spring这是一个异步方法
    @Async
    public void hello(){
        try {
            Thread.sleep(30000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("数据处理中。。。。");
    }
}
```



**2.Controller**

```java
@RestController
public class AsyncController {
    @Autowired
    AsyncService asyncService;

    @GetMapping("/hello")
    public String hello(){
        asyncService.hello();
        return "success";
    }

}

```



**3.Application**

```java
@EnableAsync  //开启异步注解功能
@SpringBootApplication
public class SpringBootTaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootTaskApplication.class, args);
    }
}
```



### 定时任务

项目开发中经常需要执行一些定时任务，比如需要在每天凌晨时候，分析一次前一天的日志信息。Spring为我们提供了异步执行任务调度的方式，提供 TaskExecutor 、TaskScheduler 接口。

**两个注解:@EnableScheduling、@Scheduled**



**cron表达式**

| 字段 | 允许值                  | 允许的特殊字符    |
| ---- | ----------------------- | ----------------- |
| 秒   | 0-59                    | , -   * /         |
| 分   | 0-59                    | , -   * /         |
| 小时 | 0-23                    | , -   * /         |
| 日期 | 1-31                    | , -   * ? / L W C |
| 月份 | 1-12                    | , -   * /         |
| 星期 | 0-7或SUN-SAT   0,7是SUN | , -   * ? / L C # |



| 字段 | 允许值                  | 允许的特殊字符    |
| ---- | ----------------------- | ----------------- |
| 秒   | 0-59                    | , -   * /         |
| 分   | 0-59                    | , -   * /         |
| 小时 | 0-23                    | , -   * /         |
| 日期 | 1-31                    | , -   * ? / L W C |
| 月份 | 1-12                    | , -   * /         |
| 星期 | 0-7或SUN-SAT   0,7是SUN | , -   * ? / L C # |



**1.service**

```java
@Service
public class ScheduledService {
    /**
     * 定时任务
     * cron 指定定时任务的cron表达式
     * second(秒),minute（分）,hour（时）,day of month（日）,month（月）,day of week（周几）
     * 0 * * * * MON-FRI (周一到周五每一分钟整分钟的时候运行一次)
     */
    //@Scheduled(cron = "0 * * * * MON-FRI")
    //@Scheduled(cron = "0,1,2,3,4 * * * * MON-FRI")  //0,1,2,3,4枚举
    //@Scheduled(cron = "0-4 * * * * MON-FRI")    //0-4秒区间
    //@Scheduled(cron = "0/4 * * * * MON-FRI")    //每4秒一次
    //@Scheduled(cron = "0 0/5 14,18 * * ?")    //每天14点整和18点整，每隔5分钟执行一次
    //@Scheduled(cron = "0 15 10 ？ * 1-6")  //每个月点周一至周六10：15分执行一次
    //@Scheduled(cron = "0 0 2 ？ * 6L")  //每个月的最后一个周六凌晨2点执行一次
    //@Scheduled(cron = "0 0 2 LW * ？")  //每个月的最后一个工作日凌晨两点执行一次
    @Scheduled(cron = "0 0 2-4 ？ * 1#1")  //每个月的第一个周一凌晨2点到4点期间每个整点都执行一次
    public void hello() {
        System.out.println("hello...");
    }
}
```

**2.Application**

```java
@EnableScheduling  //开启基于注解的定时任务
@SpringBootApplication
public class SpringBootTaskApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootTaskApplication.class, args);
    }
}

```



### 邮件任务

**1.引入pom**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-mail</artifactId>
</dependency>
```



**2.配置application.yml**

```properties
spring.mail.username=1269995677@qq.com
spring.mail.password=mpduxmuivewdigjf
spring.mail.host=smtp.qq.com
```



**3.测试**

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class SpringBootTaskApplicationTests {
    //注入邮件发送器
    @Autowired
    JavaMailSenderImpl javaMailSender;
 
    //测试简单邮件
    @Test
    public void contextLoads() {
        SimpleMailMessage message = new SimpleMailMessage();
        //邮件设置
        message.setSubject("通知：今晚开会");
        message.setText("今晚7：30开会");
        message.setTo("1269995677@qq.com");
        message.setFrom("1269995677@qq.com");
        javaMailSender.send(message);
    }
 
 
    //测试复杂邮件
    @Test
    public void test() throws MessagingException {
        //1、创建一个复杂消息邮件
        MimeMessage message = javaMailSender.createMimeMessage();
        MimeMessageHelper helper = new MimeMessageHelper(message, true);//multipart 是否上传文件
        //邮件设置
        helper.setSubject("通知：今晚开会");
        helper.setText("<b style='color:red'>今晚7：30开会</b>", true);
        helper.setTo("1269995677@qq.com");
        helper.setFrom("1269995677@qq.com");
 
        //上传附件
        helper.addAttachment("1.jpg", new File("/Users/Amy/Downloads/1.jpg"));
        helper.addAttachment("2.jpg", new File("/Users/Amy/Downloads/2.jpg"));
        javaMailSender.send(message);
    }
}
```



#### 邮件发送原理

**1.SpringBoot 自动配置MailSenderAutoConfiguration**

```java
//邮件自动配置类
@Configuration
@ConditionalOnClass({ MimeMessage.class, MimeType.class, MailSender.class })
@ConditionalOnMissingBean(MailSender.class)
@Conditional(MailSenderCondition.class)
@EnableConfigurationProperties(MailProperties.class)
@Import({ MailSenderJndiConfiguration.class, MailSenderPropertiesConfiguration.class })
public class MailSenderAutoConfiguration {
```

```java
//邮件发送属性
@ConfigurationProperties(prefix = "spring.mail")
public class MailProperties {
```



**2.自动装配JavaMailSender**

```java
@Configuration
@ConditionalOnClass(Session.class)
@ConditionalOnProperty(prefix = "spring.mail", name = "jndi-name")
@ConditionalOnJndi
class MailSenderJndiConfiguration {
 
	private final MailProperties properties;
 
	MailSenderJndiConfiguration(MailProperties properties) {
		this.properties = properties;
	}
 
	@Bean //发送邮件的组件
	public JavaMailSenderImpl mailSender(Session session) {
		JavaMailSenderImpl sender = new JavaMailSenderImpl();
		sender.setDefaultEncoding(this.properties.getDefaultEncoding().name());
		sender.setSession(session);
		return sender;
	}
```



暂时先介绍这三个我们常用的高级特性，后面根据实际应用会介绍其他的高级技术，例如：检索、安全等。

