# Spring Boot笔记（一）

## Spring Boot简介
Spring Boot来简化Spring应用开发，约定大于配置，去繁从简，just run就能创建一个独立的，产品级别的应用。

背景：J2EE笨重的开发、繁多的配置、低下的开发效率、复杂的部署流程、第三方技术集成难度大。

Spring全家桶时代
Spring Boot-J2EE一站式解决方案
Spring Cloud-分布式整体解决方案


优点：

+ 快速创建独立运行的Spring项目以及与主流框架集成
+ 使用嵌入式的Servlet容器，应用无需打成WAR包
+ starters自动依赖与版本控制
+ 大量的自动配置，简化开发，也可修改默认值
+ 无需配置XML，无代码生成，开箱即用
+ 准生产环境的运行时应用监控
+ 与云计算的天然集成

###微服务

2014，martin fowler

微服务：架构风格（服务微化）

一个应用应该是一组小型服务；可以通过HTTP的方式进行互通；

单体应用：ALL IN ONE

微服务：每一个功能元素最终都是一个可独立替换和独立升级的软件单元；

[详细参照微服务文档](httpsmartinfowler.comarticlesmicroservices.html#MicroservicesAndSoa)



## Spring Boot Hello World

### 环境需求

+ jdk1.8：Spring Boot 推荐jdk1.7及以上；java version 1.8.0_112
+ maven3.x：maven 3.3以上版本；Apache Maven 3.3.9
+ IntelliJIDEA2017：IntelliJ IDEA 2017.2.2 x64、STS
+ SpringBoot 1.5.9.RELEASE：1.5.9


### maven额外配置
给maven 的settings.xml配置文件的profiles标签添加：

```xml
profile
  idjdk-1.8id
  activation
    activeByDefaulttrueactiveByDefault
    jdk1.8jdk
  activation
  properties
    maven.compiler.source1.8maven.compiler.source
    maven.compiler.target1.8maven.compiler.target
    maven.compiler.compilerVersion1.8maven.compiler.compilerVersion
  properties
profile
```

### hello world实现

功能：浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串；


1.创建一个maven工程；（jar）

2.导入spring boot相关的依赖

```xml
    parent
        groupIdorg.springframework.bootgroupId
        artifactIdspring-boot-starter-parentartifactId
        version2.0.6.RELEASEversion
    parent
    dependencies
        dependency
            groupIdorg.springframework.bootgroupId
            artifactIdspring-boot-starter-webartifactId
        dependency
    dependencies
```



我们可以利用IDEA：使用 Spring Initializer快速创建项目

IDE都支持使用Spring的项目创建向导快速创建一个Spring Boot项目；

选择我们需要的模块；向导会联网创建Spring Boot项目；

默认生成的Spring Boot项目，主程序已经生成好了，我们只需要我们自己的逻辑

+ java
+ resources文件夹中目录结构
    + static：保存所有的静态资源； jscssimages
    + templates：保存所有的模板页面；（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页面）；可以使用模板引擎（freemarker、thymeleaf）；
    + application.properties：Spring Boot应用的配置文件；可以修改一些默认设置；


3.编写一个主程序；启动Spring Boot应用

```java


   @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

         Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

4.编写相关的Controller、Service

```java
@Controller
public class HelloController {

    @ResponseBody
    @RequestMapping(hello)
    public String hello(){
        return Hello World!;
    }
}
```

5.运行主程序测试
http127.0.0.18080hello


6.简化部署

```xml
 !-- 这个插件，可以将应用打包成一个可执行的jar包；--
    build
        plugins
            plugin
                groupIdorg.springframework.bootgroupId
                artifactIdspring-boot-maven-pluginartifactId
            plugin
        plugins
    build
```

将这个应用打成jar包，直接使用java -jar的命令进行执行；

## Hello World分析

### 1.POM文件

(1)父项目
```xml
parent
    groupIdorg.springframework.bootgroupId
    artifactIdspring-boot-starter-parentartifactId
    version2.0.6.RELEASEversion
parent
```


父项目的父项目：
```java
parent
    groupIdorg.springframework.bootgroupId
    artifactIdspring-boot-dependenciesartifactId
    version2.0.6.RELEASEversion
    relativePath....spring-boot-dependenciesrelativePath
parent
```
他来真正管理Spring Boot应用里面的所有依赖版本
Spring Boot的版本仲裁中心；
以后我们导入依赖默认是不需要写版本；（没有在dependencies里面管理的依赖自然需要声明版本号）


(2)启动器
```xml
dependencies
    dependency
        groupIdorg.springframework.bootgroupId
        artifactIdspring-boot-starter-webartifactId
    dependency
dependencies
```
spring-boot-starter：spring-boot场景启动器；帮我们导入了web模块正常运行所依赖的组件。

Spring Boot将所有的功能场景都抽取出来，做成一个个的starters（启动器），只需要在项目里面引入这些starter相关场景的所有依赖都会导入进来。要用什么功能就导入什么场景的启动器

其它启动器请看：httpsdocs.spring.iospring-bootdocs2.1.0.BUILD-SNAPSHOTreferencehtmlsingle#using-boot-starter


### 主程序类，主入口类

```java

   @SpringBootApplication 来标注一个主程序类，说明这是一个Spring Boot应用
 
@SpringBootApplication
public class HelloWorldMainApplication {

    public static void main(String[] args) {

         Spring应用启动起来
        SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}

```

@SpringBootApplicationSpring Boot应用标注在某个类上说明这个类是SpringBoot的主配置类，SpringBoot就应该运行这个类的main方法来启动SpringBoot应用；


@SpringBootApplication的实现
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {
    @Filter(type = FilterType.CUSTOM,classes = {TypeExcludeFilter.class}), 
    @Filter(type = FilterType.CUSTOM,classes={AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```
1.@SpringBootConfigurationSpring Boot的配置类,标注在某个类上，表示这是一个Spring Boot的配置类；

@Configuration配置类上来标注这个注解，配置类（可以叫做配置文件）；配置类也是容器中的一个组件（@Component）

2.@EnableAutoConfiguration开启自动配置功能；以前我们需要配置的东西，Spring Boot帮我们自动配置；@EnableAutoConfiguration告诉SpringBoot开启自动配置功能；这样自动配置才能生效；
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```


2.1 @AutoConfigurationPackage：给容器中导入组件,作用：将主配置类（@SpringBootApplication标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器；
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
```
Import(Registrar.class)：Spring的底层注解@Import，给容器中导入一个组件；导入的组件由Registrar.class；

2.2 @Import({AutoConfigurationImportSelector.class})

给容器中导入组件？
EnableAutoConfigurationImportSelector：导入哪些组件的选择器；
将所有需要导入的组件以全类名的方式返回；这些组件就会被添加到容器中；他会给容器中导入非常多的自动配置类（xxxAutoConfiguration）；就是给容器中导入这个场景需要的所有组件，并配置好这些组件；有了自动配置类，免去了我们手动编写配置注入功能组件等的工作；

![Spring Boot学习----入门&&配置文件](httpimg.bcoder.top2018.10.201.png)

Boot在启动的时候从类路径下的META-INFspring.factories中获取EnableAutoConfiguration指定的值，将这些值作为自动配置类导入到容器中，自动配置类就生效，帮我们进行自动配置工作；以前我们需要自己配置的东西，自动配置类都自动完成了


## Spring Boot配置文件

SpringBoot使用一个全局的配置文件，配置文件名是固定的：

+ application.properties
+ application.yml


### YAML介绍

YAML（YAML Ain't Markup Language）：

+ YAML  A Markup Language：是一个标记语言
+ YAML   isn't Markup Language：不是一个标记语言；



标记语言：

+ 以前的配置文件；大多都使用的是  xxxx.xml文件；
+ YAML：以数据为中心，比json、xml等更适合做配置文件；


YAML：配置例子

```yaml
server
  port 8081
```

XML：
```xml
server
	port8081port
server
```


properties：
```xml
server.port=8888
```




​    
### YAML语法

YAML基本语法
1.使用缩进表示层级关系
2.缩进时不允许使用Tab键，只允许使用空格。
3.缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
4.大小写敏感
```yaml
server
    port 8081
    path hello
```


值的写法
1.字面量：普通的值（数字，字符串，布尔）
k v：字面直接来写；
字符串默认不用加上单引号或者双引号；
：双引号；不会转义字符串里面的特殊字符；特殊字符会作为本身想表示的意思
name   zhangsan n lisi：输出；zhangsan 换行  lisi

''：单引号；会转义特殊字符，特殊字符最终只是一个普通的字符串数据
name   'zhangsan n lisi'：输出；zhangsan n  lisi

2.对象、Map（属性和值）（键值对）：
k v：在下一行来写对象的属性和值的关系；注意缩进
对象还是k v的方式
```yaml
friends
		lastName zhangsan
		age 20
```

行内写法：
```yaml
friends {lastName zhangsan,age 18}
```

3.数组（List、Set）：

用- 值表示数组中的一个元素
```yaml
pets
 - cat
 - dog
 - pig
```

行内写法
```yaml
pets [cat,dog,pig]
```




### YAML实践

application.yml
```yaml
person
  lastName 张三
  age 18
  boss false
  birth 20181012
  maps {k1 v1,k2 v2}
  lists
    - 李四
    - 赵六
  dog
    name 哈士奇
    age 18
```

java bean
Person.java
```java

  将配置文件中配置的每一个属性的值，映射到这个组件中
  @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
       prefix = person：配置文件中哪个下面的所有属性进行一一映射
 
  只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
 
 
@Data
@Component
@ConfigurationProperties(prefix = person)
public class Person {

    private String lastName;
    private Integer age;
    private Boolean boss;
    private Date birth;

    private MapString,Object maps;
    private ListObject lists;
    private Dog dog;
}

```
Dog
```java
import lombok.Data;

@Data
public class Dog {
    private String name;

    private Integer age;
}
```


测试Springboot1ApplicationTests：
```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot1ApplicationTests {

    @Autowired
    Person person;

    @Test
	public void contextLoads() {
        System.out.println(person);
    }

}
```

运行结果：

![Spring Boot学习----入门&&配置文件](httpimg.bcoder.top2018.10.202.png)

需要注意的是，properties配置文件在idea中默认utf-8可能会乱码

调整：

![Spring Boot学习----入门&&配置文件](httpimg.bcoder.top2018.10.203.png)





### 配置文件基础

除了@ConfigurationProperties(prefix =person)可以注入值，也可以用@value进行注入值,

 例如：

```java
@Component
@ConfigurationProperties(prefix = person)
注入值数据校验
@Validated
public class Person {

    
      bean class=Person
           property name=lastName value=字面量${key}从环境变量、配置文件中获取值#{SpEL}property
      bean
     

   lastName必须是邮箱格式
    @Email
    @Value(${person.last-name})
    private String lastName;
    @Value(#{112})
    private Integer age;
    @Value(true)
    private Boolean boss;

    private Date birth;
    private MapString,Object maps;
    private ListObject lists;
    private Dog dog;
```


 @Value获取值和@ConfigurationProperties获取值比较

![Spring Boot学习----入门&&配置文件](httpimg.bcoder.top2018.10.204.png)

配置文件yml还是properties他们都能获取到值；

如果说，我们只是在某个业务逻辑中需要获取一下配置文件中的某项值，使用@Value；

如果说，我们专门编写了一个javaBean来和配置文件进行映射，我们就直接使用@ConfigurationProperties；




@PropertySource&@ImportResource&@Bean

@PropertySource：加载指定的配置文件
```java

  将配置文件中配置的每一个属性的值，映射到这个组件中
  @ConfigurationProperties：告诉SpringBoot将本类中的所有属性和配置文件中相关的配置进行绑定；
       prefix = person：配置文件中哪个下面的所有属性进行一一映射
 
  只有这个组件是容器中的组件，才能容器提供的@ConfigurationProperties功能；
   @ConfigurationProperties(prefix = person)默认从全局配置文件中获取值；
 
 
@PropertySource(value = {classpathperson.properties})
@Component
@ConfigurationProperties(prefix = person)

public class Person {
    private String lastName;
    private Integer age;
    private Boolean boss;
```

@ImportResource：导入Spring的配置文件，让配置文件里面的内容生效；

Spring Boot里面没有Spring的配置文件，我们自己编写的配置文件，也不能自动识别；

想让Spring的配置文件生效，加载进来；@ImportResource标注在一个配置类(Springboot1Application.java)上

```java
@ImportResource(locations = {classpathbeans.xml})
导入Spring的配置文件让其生效
```


不要编写Spring的配置文件

```xml
xml version=1.0 encoding=UTF-8
beans xmlns=httpwww.springframework.orgschemabeans
       xmlnsxsi=httpwww.w3.org2001XMLSchema-instance
       xsischemaLocation=httpwww.springframework.orgschemabeans httpwww.springframework.orgschemabeansspring-beans.xsd


    bean id=helloService class=com.atguigu.springboot.service.HelloServicebean
beans
```

SpringBoot推荐给容器中添加组件的方式；推荐使用全注解的方式
1、配置类@Configuration------Spring配置文件
2、使用@Bean给容器中添加组件

```java

  @Configuration：指明当前类是一个配置类；就是来替代之前的Spring配置文件
 
  在配置文件中用beanbean标签添加组件
 
 
@Configuration
public class MyAppConfig {

    将方法的返回值添加到容器中；容器中这个组件默认的id就是方法名
    @Bean
    public HelloService helloService02(){
        System.out.println(配置类@Bean给容器中添加组件了...);
        return new HelloService();
    }
}
```


### 配置文件占位符

1、随机数

```java
${random.value}、${random.int}、${random.long}
${random.int(10)}、${random.int[1024,65536]}

```



2、占位符获取之前配置的值，如果没有可以是用指定默认值

```properties
person.last-name=张三${random.uuid}
person.age=${random.int}
person.birth=20171215
person.boss=false
person.maps.k1=v1
person.maps.k2=14
person.lists=a,b,c
person.dog.name=${person.hellohello}_dog
person.dog.age=15
```

### Profile
1、多Profile文件

我们在主配置文件编写的时候，文件名可以是application-{profile}.propertiesyml

默认使用application.properties的配置；



2、yml支持多文档块方式

```yml
server
  port 8081
spring
  profiles
    active prod
---
server
  port 8083
spring
  profiles dev
--- 
server
  port 8084
spring
  profiles prod  #指定属于哪个环境
```

3、激活指定profile
（1）在配置文件中指定  spring.profiles.active=dev

（2）命令行：
java -jar spring-boot-02-config-0.0.1-SNAPSHOT.jar --spring.profiles.active=dev；
可以直接在测试的时候，配置传入命令行参数

（3）虚拟机参数；-Dspring.profiles.active=dev

### 配置文件加载位置

springboot 启动会扫描以下位置的application.properties或者application.yml文件作为Spring boot的默认配置文件

- file.config
- file.
- classpathconfig
- classpath

优先级由高到底，高优先级的配置会覆盖低优先级的配置；
SpringBoot会从这四个位置全部加载主配置文件；互补配置；


我们还可以通过spring.config.location来改变默认的配置文件位置

项目打包好以后，我们可以使用命令行参数的形式，启动项目的时候来指定配置文件的新位置；指定配置文件和默认加载的这些配置文件共同起作用形成互补配置；

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --spring.config.location=Gapplication.properties


### 外部配置加载顺序

SpringBoot也可以从以下位置加载配置； 优先级从高到低；高优先级的配置覆盖低优先级的配置，所有的配置会形成互补配置

1.命令行参数

所有的配置都可以在命令行上进行指定

java -jar spring-boot-02-config-02-0.0.1-SNAPSHOT.jar --server.port=8087  --server.context-path=abc

多个配置用空格分开； --配置项=值


2.来自javacompenv的JNDI属性

3.Java系统属性（System.getProperties()）

4.操作系统环境变量

5.RandomValuePropertySource配置的random.属性值

由jar包外向jar包内进行寻找，优先加载带profile

6.jar包外部的application-{profile}.properties或application.yml(带spring.profile)配置文件
7.jar包内部的application-{profile}.properties或application.yml(带spring.profile)配置文件
8.jar包外部的application.properties或application.yml(不带spring.profile)配置文件
9.jar包内部的application.properties或application.yml(不带spring.profile)配置文件

10.@Configuration注解类上的@PropertySource

11.通过SpringApplication.setDefaultProperties指定的默认属性

所有支持的配置加载来源；

[参考官方文档](httpsdocs.spring.iospring-bootdocs2.0.6.RELEASEreferencehtmlsingle#boot-features-external-config)



### 自动配置原理

配置文件到底能写什么？怎么写？自动配置原理；

[配置文件能配置的属性参照](httpsdocs.spring.iospring-bootdocs2.0.6.RELEASEreferencehtmlsingle#common-application-properties)

1）、SpringBoot启动的时候加载主配置类，开启了自动配置功能 @EnableAutoConfiguration

2）、@EnableAutoConfiguration 作用：

-  利用EnableAutoConfigurationImportSelector给容器中导入一些组件？
-  可以查看selectImports()方法的内容；
-  ListString configurations = getCandidateConfigurations(annotationMetadata,      attributes);获取候选的配置

`SpringFactoriesLoader.loadFactoryNames()`
扫描所有jar包类路径下  META-INFspring.factories
把扫描到的这些文件的内容包装成properties对象
从properties中获取到EnableAutoConfiguration.class类（类名）对应的值，然后把他们添加在容器中


将（spring-boot-autoconfigure-2.0.6.RELEASE.jar）类路径下  META-INFspring.factories 里面配置的所有EnableAutoConfiguration的值加入到了容器中==

```properties
# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,
org.springframework.boot.autoconfigure.cache.CacheAutoConfiguration,
org.springframework.boot.autoconfigure.cassandra.CassandraAutoConfiguration,
org.springframework.boot.autoconfigure.cloud.CloudAutoConfiguration,
org.springframework.boot.autoconfigure.context.ConfigurationPropertiesAutoConfiguration,
org.springframework.boot.autoconfigure.context.MessageSourceAutoConfiguration,
org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration,
org.springframework.boot.autoconfigure.couchbase.CouchbaseAutoConfiguration,
org.springframework.boot.autoconfigure.dao.PersistenceExceptionTranslationAutoConfiguration,
org.springframework.boot.autoconfigure.data.cassandra.CassandraDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.cassandra.CassandraReactiveRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.cassandra.CassandraRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseReactiveRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.couchbase.CouchbaseRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchAutoConfiguration,
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.elasticsearch.ElasticsearchRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.jpa.JpaRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.ldap.LdapDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.ldap.LdapRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.mongo.MongoDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.mongo.MongoReactiveRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.mongo.MongoRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.neo4j.Neo4jDataAutoConfiguration,
org.springframework.boot.autoconfigure.data.neo4j.Neo4jRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.solr.SolrRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.redis.RedisAutoConfiguration,
org.springframework.boot.autoconfigure.data.redis.RedisReactiveAutoConfiguration,
org.springframework.boot.autoconfigure.data.redis.RedisRepositoriesAutoConfiguration,
org.springframework.boot.autoconfigure.data.rest.RepositoryRestMvcAutoConfiguration,
org.springframework.boot.autoconfigure.data.web.SpringDataWebAutoConfiguration,
org.springframework.boot.autoconfigure.elasticsearch.jest.JestAutoConfiguration,
org.springframework.boot.autoconfigure.flyway.FlywayAutoConfiguration,
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration,
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration,
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration,
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration,
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration,
org.springframework.boot.autoconfigure.hazelcast.HazelcastJpaDependencyAutoConfiguration,
org.springframework.boot.autoconfigure.http.HttpMessageConvertersAutoConfiguration,
org.springframework.boot.autoconfigure.http.codec.CodecsAutoConfiguration,
org.springframework.boot.autoconfigure.influx.InfluxDbAutoConfiguration,
org.springframework.boot.autoconfigure.info.ProjectInfoAutoConfiguration,
org.springframework.boot.autoconfigure.integration.IntegrationAutoConfiguration,
org.springframework.boot.autoconfigure.jackson.JacksonAutoConfiguration,
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,
org.springframework.boot.autoconfigure.jdbc.JdbcTemplateAutoConfiguration,
org.springframework.boot.autoconfigure.jdbc.JndiDataSourceAutoConfiguration,
org.springframework.boot.autoconfigure.jdbc.XADataSourceAutoConfiguration,
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,
org.springframework.boot.autoconfigure.jms.JmsAutoConfiguration,
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,
org.springframework.boot.autoconfigure.jms.JndiConnectionFactoryAutoConfiguration,
org.springframework.boot.autoconfigure.jms.activemq.ActiveMQAutoConfiguration,
org.springframework.boot.autoconfigure.jms.artemis.ArtemisAutoConfiguration,
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration,
org.springframework.boot.autoconfigure.jersey.JerseyAutoConfiguration,
org.springframework.boot.autoconfigure.jooq.JooqAutoConfiguration,
org.springframework.boot.autoconfigure.jsonb.JsonbAutoConfiguration,
org.springframework.boot.autoconfigure.kafka.KafkaAutoConfiguration,
org.springframework.boot.autoconfigure.ldap.embedded.EmbeddedLdapAutoConfiguration,
org.springframework.boot.autoconfigure.ldap.LdapAutoConfiguration,
org.springframework.boot.autoconfigure.liquibase.LiquibaseAutoConfiguration,
org.springframework.boot.autoconfigure.mail.MailSenderAutoConfiguration,
org.springframework.boot.autoconfigure.mail.MailSenderValidatorAutoConfiguration,
org.springframework.boot.autoconfigure.mongo.embedded.EmbeddedMongoAutoConfiguration,
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,
org.springframework.boot.autoconfigure.mongo.MongoReactiveAutoConfiguration,
org.springframework.boot.autoconfigure.mustache.MustacheAutoConfiguration,
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,
org.springframework.boot.autoconfigure.quartz.QuartzAutoConfiguration,
org.springframework.boot.autoconfigure.reactor.core.ReactorCoreAutoConfiguration,
org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration,
org.springframework.boot.autoconfigure.security.servlet.SecurityRequestMatcherProviderAutoConfiguration,
org.springframework.boot.autoconfigure.security.servlet.UserDetailsServiceAutoConfiguration,
org.springframework.boot.autoconfigure.security.servlet.SecurityFilterAutoConfiguration,
org.springframework.boot.autoconfigure.security.reactive.ReactiveSecurityAutoConfiguration,
org.springframework.boot.autoconfigure.security.reactive.ReactiveUserDetailsServiceAutoConfiguration,
org.springframework.boot.autoconfigure.sendgrid.SendGridAutoConfiguration,
org.springframework.boot.autoconfigure.session.SessionAutoConfiguration,
org.springframework.boot.autoconfigure.security.oauth2.client.OAuth2ClientAutoConfiguration,
org.springframework.boot.autoconfigure.solr.SolrAutoConfiguration,
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,
org.springframework.boot.autoconfigure.transaction.TransactionAutoConfiguration,
org.springframework.boot.autoconfigure.transaction.jta.JtaAutoConfiguration,
org.springframework.boot.autoconfigure.validation.ValidationAutoConfiguration,
org.springframework.boot.autoconfigure.web.client.RestTemplateAutoConfiguration,
org.springframework.boot.autoconfigure.web.embedded.EmbeddedWebServerFactoryCustomizerAutoConfiguration,
org.springframework.boot.autoconfigure.web.reactive.HttpHandlerAutoConfiguration,
org.springframework.boot.autoconfigure.web.reactive.ReactiveWebServerFactoryAutoConfiguration,
org.springframework.boot.autoconfigure.web.reactive.WebFluxAutoConfiguration,
org.springframework.boot.autoconfigure.web.reactive.error.ErrorWebFluxAutoConfiguration,
org.springframework.boot.autoconfigure.web.reactive.function.client.WebClientAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.DispatcherServletAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.ServletWebServerFactoryAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.error.ErrorMvcAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.HttpEncodingAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.MultipartAutoConfiguration,
org.springframework.boot.autoconfigure.web.servlet.WebMvcAutoConfiguration,
org.springframework.boot.autoconfigure.websocket.reactive.WebSocketReactiveAutoConfiguration,
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketServletAutoConfiguration,
org.springframework.boot.autoconfigure.websocket.servlet.WebSocketMessagingAutoConfiguration,
org.springframework.boot.autoconfigure.webservices.WebServicesAutoConfiguration

# Failure analyzers
org.springframework.boot.diagnostics.FailureAnalyzer=
org.springframework.boot.autoconfigure.diagnostics.analyzer.NoSuchBeanDefinitionFailureAnalyzer,
org.springframework.boot.autoconfigure.jdbc.DataSourceBeanCreationFailureAnalyzer,
org.springframework.boot.autoconfigure.jdbc.HikariDriverConfigurationFailureAnalyzer,
org.springframework.boot.autoconfigure.session.NonUniqueSessionRepositoryFailureAnalyzer

# Template availability providers
org.springframework.boot.autoconfigure.template.TemplateAvailabilityProvider=
org.springframework.boot.autoconfigure.freemarker.FreeMarkerTemplateAvailabilityProvider,
org.springframework.boot.autoconfigure.mustache.MustacheTemplateAvailabilityProvider,
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAvailabilityProvider,
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafTemplateAvailabilityProvider,
org.springframework.boot.autoconfigure.web.servlet.JspTemplateAvailabilityProvider

```
每一个这样的  xxxAutoConfiguration类都是容器中的一个组件，都加入到容器中；用他们来做自动配置；

3）、每一个自动配置类进行自动配置功能；

4）、以HttpEncodingAutoConfiguration（Http编码自动配置）为例解释自动配置原理；

```java
@Configuration 
表示这是一个配置类，以前编写的配置文件一样，也可以给容器中添加组件
@EnableConfigurationProperties(HttpEncodingProperties.class) 
启动指定类的ConfigurationProperties功能；将配置文件中对应的值和HttpEncodingProperties绑定起来；并把HttpEncodingProperties加入到ioc容器中

@ConditionalOnWebApplication
Spring底层@Conditional注解，根据不同的条件，如果满足指定的条件，整个配置类里面的配置就会生效；
判断当前应用是否是web应用，如果是，当前配置类生效

@ConditionalOnClass(CharacterEncodingFilter.class)  判断当前项目有没有这个类CharacterEncodingFilter；SpringMVC中进行乱码解决的过滤器；

@ConditionalOnProperty(prefix = spring.http.encoding, value = enabled, matchIfMissing = true)  
判断配置文件中是否存在某个配置  spring.http.encoding.enabled；如果不存在，判断也是成立的
即使我们配置文件中不配置spring.http.encoding.enabled=true，也是默认生效的；
public class HttpEncodingAutoConfiguration {
  
  	他已经和SpringBoot的配置文件映射了
  	private final HttpEncodingProperties properties;
  
   只有一个有参构造器的情况下，参数的值就会从容器中拿
  	public HttpEncodingAutoConfiguration(HttpEncodingProperties properties) {
		this.properties = properties;
	}
  
    @Bean   给容器中添加一个组件，这个组件的某些值需要从properties中获取
	@ConditionalOnMissingBean(CharacterEncodingFilter.class) 判断容器没有这个组件？
	public CharacterEncodingFilter characterEncodingFilter() {
		CharacterEncodingFilter filter = new OrderedCharacterEncodingFilter();
		filter.setEncoding(this.properties.getCharset().name());
		filter.setForceRequestEncoding(this.properties.shouldForce(Type.REQUEST));
		filter.setForceResponseEncoding(this.properties.shouldForce(Type.RESPONSE));
		return filter;
	}
```
根据当前不同的条件判断，决定这个配置类是否生效？

一但这个配置类生效；这个配置类就会给容器中添加各种组件；这些组件的属性是从对应的properties类中获取的，这些类里面的每一个属性又是和配置文件绑定的；

5）、所有在配置文件中能配置的属性都是在xxxxProperties类中封装者；配置文件能配置什么就可以参照某个功能对应的这个属性类

```java
@ConfigurationProperties(prefix = spring.http.encoding)  从配置文件中获取指定的值和bean的属性进行绑定
public class HttpEncodingProperties {

   public static final Charset DEFAULT_CHARSET = Charset.forName(UTF-8);
```

精髓：
1）、SpringBoot启动会加载大量的自动配置类
2）、我们看我们需要的功能有没有SpringBoot默认写好的自动配置类；
3）、我们再来看这个自动配置类中到底配置了哪些组件；（只要我们要用的组件有，我们就不需要再来配置了）
4）、给容器中自动配置类添加组件的时候，会从properties类中获取某些属性。我们就可以在配置文件中指定这些属性的值；

xxxxAutoConfigurartion：自动配置类；给容器中添加组件
xxxxProperties封装配置文件中相关属性；




1、@Conditional派生注解（Spring原生的@Conditional作用）

作用：必须是@Conditional指定的条件成立，才给容器中添加组件，配置配里面的所有内容才生效；

 @Conditional扩展注解                 作用（判断是否满足当前指定条件）               
-------------------------------  ------------------------------
 @ConditionalOnJava               系统的java版本是否符合要求                
 @ConditionalOnBean               容器中存在指定Bean；                   
 @ConditionalOnMissingBean        容器中不存在指定Bean；                  
 @ConditionalOnExpression         满足SpEL表达式指定                    
 @ConditionalOnClass              系统中有指定的类                       
 @ConditionalOnMissingClass       系统中没有指定的类                      
 @ConditionalOnSingleCandidate    容器中只有一个指定的Bean，或者这个Bean是首选Bean 
 @ConditionalOnProperty           系统中指定的属性是否有指定的值                
 @ConditionalOnResource           类路径下是否存在指定资源文件                 
 @ConditionalOnWebApplication     当前是web环境                       
 @ConditionalOnNotWebApplication  当前不是web环境                      
 @ConditionalOnJndi               JNDI存在指定项                      

自动配置类必须在一定的条件下才能生效；

我们怎么知道哪些自动配置类生效；

我们可以通过启用debug=true属性；来让控制台打印自动配置报告，这样我们就可以很方便的知道哪些自动配置类生效；

```java
=========================
AUTO-CONFIGURATION REPORT
=========================


Positive matches（自动配置类启用的）
-----------------

   DispatcherServletAutoConfiguration matched
      - @ConditionalOnClass found required class 'org.springframework.web.servlet.DispatcherServlet'; @ConditionalOnMissingClass did not find unwanted class (OnClassCondition)
      - @ConditionalOnWebApplication (required) found StandardServletEnvironment (OnWebApplicationCondition)
        
    
Negative matches（没有启动，没有匹配成功的自动配置类）
-----------------

   ActiveMQAutoConfiguration
      Did not match
         - @ConditionalOnClass did not find required classes 'javax.jms.ConnectionFactory', 'org.apache.activemq.ActiveMQConnectionFactory' (OnClassCondition)

   AopAutoConfiguration
      Did not match
         - @ConditionalOnClass did not find required classes 'org.aspectj.lang.annotation.Aspect', 'org.aspectj.lang.reflect.Advice' (OnClassCondition)
        
```