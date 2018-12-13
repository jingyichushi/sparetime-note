





## 其他



1.对测试用例类进行静态导入

```java
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.delete;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.fileUpload;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.get;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.post;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.put;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.jsonPath;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;
```

2.使用json path进行测试：

```java 
@Test
public void whenGetInfoSuccess() throws Exception {
   String result = mockMvc.perform(get("/user/1")
         .contentType(MediaType.APPLICATION_JSON_UTF8))
         .andExpect(status().isOk())
         .andExpect(jsonPath("$.username").value("tom"))
         .andReturn().getResponse().getContentAsString();
   
    String result = mockMvc.perform(
        get("/user").param("username", "jojo").param("age", "18").param("ageTo", "60").param("xxx", "yyy")
        // .param("size", "15")
        // .param("page", "3")
        // .param("sort", "age,desc")
        .contentType(MediaType.APPLICATION_JSON_UTF8))
        .andExpect(status().isOk()).andExpect(jsonPath("$.length()").value(3))
        .andReturn().getResponse().getContentAsString();    

}
```



3.返回对象视图：@JsonView

https://www.baidu.com/link?url=3WdntvL2ni2uCODUEE_gzcReiXdU9SKe1f_mUWJJB51aJVLrueVhG2ZcyFgMnJ2j&wd=&eqid=98ebf47200000341000000055c124181



4.前后端分离：时间戳





5.@Vaild 参数校验

校验值

如何返回错误信息

如何进行错误信息定制

自定义注解来实现参数校验：

``` java
@Target({ElementType.METHOD, ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Constraint(validatedBy = MyConstraintValidator.class)
public @interface MyConstraint {
   
   String message();

   Class<?>[] groups() default { };

   Class<? extends Payload>[] payload() default { };

}
```

```java
public class MyConstraintValidator implements ConstraintValidator<MyConstraint, Object> {

   @Autowired
   private HelloService helloService;
   
   @Override
   public void initialize(MyConstraint constraintAnnotation) {
      System.out.println("my validator init");
   }

   @Override
   public boolean isValid(Object value, ConstraintValidatorContext context) {
      helloService.greeting("tom");
      System.out.println(value);
      return true;
   }

}
```



6.Restful API 拦截



1）.Filter（可以拿到原始的Http请求和响应信息）

```java
@Component
public class TimeFilter implements Filter {


   @Override
   public void destroy() {
      System.out.println("time filter destroy");
   }


   @Override
   public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
         throws IOException, ServletException {
      System.out.println("time filter start");
      long start = new Date().getTime();
      chain.doFilter(request, response);
      System.out.println("time filter 耗时:"+ (new Date().getTime() - start));
      System.out.println("time filter finish");
   }


   @Override
   public void init(FilterConfig arg0) throws ServletException {
      System.out.println("time filter init");
   }

}
```



加入第三方filter加入到Springboot中（去除@Component）：

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
     
  @Bean
   public FilterRegistrationBean timeFilter() {
      
      FilterRegistrationBean registrationBean = new FilterRegistrationBean();
      
      TimeFilter timeFilter = new TimeFilter();
      registrationBean.setFilter(timeFilter);
      
      List<String> urls = new ArrayList<>();
      urls.add("/*");
      registrationBean.setUrlPatterns(urls);
      return registrationBean;
   }

}
```

 

2）.拦截器（http原始信息+真正调用的方法）：

``` java
@Component
public class TimeInterceptor implements HandlerInterceptor {

   @Override
   public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
         throws Exception {
      System.out.println("preHandle");
 System.out.println(((HandlerMethod)handler).getBean().getClass().getName());
      System.out.println(((HandlerMethod)handler).getMethod().getName());
      
      request.setAttribute("startTime", new Date().getTime());
      return true;
   }

   @Override
   public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
         ModelAndView modelAndView) throws Exception {
      System.out.println("postHandle");
      Long start = (Long) request.getAttribute("startTime");
      System.out.println("time interceptor 耗时:"+ (new Date().getTime() - start));

   }

   @Override
   public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
         throws Exception {
      System.out.println("afterCompletion");
      Long start = (Long) request.getAttribute("startTime");
      System.out.println("time interceptor 耗时:"+ (new Date().getTime() - start));
      System.out.println("ex is "+ex);

   }

}
```

```java
@Configuration
public class WebConfig extends WebMvcConfigurerAdapter {
   
   @Autowired
   private TimeInterceptor timeInterceptor;
   
   @Override
   public void addInterceptors(InterceptorRegistry registry) {
	    registry.addInterceptor(timeInterceptor);
   }
}
```





3）.切片(可以获取请求参数)

```java
@Aspect
@Component
public class TimeAspect {
   
   @Around("execution(* com.imooc.web.controller.UserController.*(..))")
   public Object handleControllerMethod(ProceedingJoinPoint pjp) throws Throwable {
      
      System.out.println("time aspect start");
      
      Object[] args = pjp.getArgs();
      for (Object arg : args) {
         System.out.println("arg is "+arg);
      }
      
      long start = new Date().getTime();
      
      Object object = pjp.proceed();
      
      System.out.println("time aspect 耗时:"+ (new Date().getTime() - start));
      
      System.out.println("time aspect end");
      
      return object;
   }

}
```

![1544711564329](img/Untitled/1544711564329.png)

7.Spring Boot文件上传和下载

1）文件上传

测试用例：

``` java
@Test
public void whenUploadSuccess() throws Exception {
   String result = mockMvc.perform(fileUpload("/file")
         .file(new MockMultipartFile("file", "test.txt", "multipart/form-data", "hello upload".getBytes("UTF-8"))))
         .andExpect(status().isOk())
         .andReturn().getResponse().getContentAsString();
   System.out.println(result);
}
```

具体实现：

```java
@PostMapping
public FileInfo upload(MultipartFile file) throws Exception {

   System.out.println(file.getName());
   System.out.println(file.getOriginalFilename());
   System.out.println(file.getSize());
   String folder = "/Users/zhailiang/Documents/my/muke/inaction/java/workspace/github/imooc-security-demo/src/main/java/com/imooc/web/controller";

   //可能上传到HDFS或者阿里云服务器上
   File localFile = new File(folder, new Date().getTime() + ".txt");

   file.transferTo(localFile);

    //url入库
   return new FileInfo(localFile.getAbsolutePath());
}
```





2）文件下载

``` java
import org.apache.commons.io.IOUtils;

@GetMapping("/{id}")
public void download(@PathVariable String id, HttpServletRequest request, HttpServletResponse response) throws Exception {

    String folder = "/Users/zhailiang/Documents/my/muke/inaction/java/workspace/github/imooc-security-demo/src/main/java/com/imooc/web/controller";
    
   try (InputStream inputStream = new FileInputStream(new File(folder, id + ".txt"));
         OutputStream outputStream = response.getOutputStream();) {
      
      response.setContentType("application/x-download");
      response.addHeader("Content-Disposition", "attachment;filename=test.txt");
      
      IOUtils.copy(inputStream, outputStream);
      outputStream.flush();
   } 

}
```





8.异步处理Rest服务

![1544711501449](img/Untitled/1544711501449.png)



同步方式：

``` java
    @RequestMapping("/order")
   public String order() throws Exception {
      logger.info("主线程开始");
       Thread.sleep(1000);
       logger.info("主线程返回");
       return "success";
   }
```



Callabe异步处理：

```java
@RequestMapping("/order")
public Callable<String> order() throws Exception {
   logger.info("主线程开始");
  
   Callable<String> result = new Callable<String>() {
      @Override
      public String call() throws Exception {
         logger.info("副线程开始");
         Thread.sleep(1000);
         logger.info("副线程返回");
         return "success";
      }
   };
   logger.info("主线程返回");
   return result;
}
```





DeferredResult处理

![1544712193057](img/Untitled/1544712193057.png)



```java
    @Autowired
   private MockQueue mockQueue;
   
   @Autowired
   private DeferredResultHolder deferredResultHolder;
   
   private Logger logger = LoggerFactory.getLogger(getClass());
   
   @RequestMapping("/order")
   public DeferredResult<String> order() throws Exception {
      logger.info("主线程开始");
      
      String orderNumber = RandomStringUtils.randomNumeric(8);
      mockQueue.setPlaceOrder(orderNumber);
      
      DeferredResult<String> result = new DeferredResult<>();
      deferredResultHolder.getMap().put(orderNumber, result);
      
      return result;
     
   }
```



9.wiremock进行模拟http请求（伪造Rest服务）





10.依赖管理

```xml
<dependencyManagement>
   <dependencies>
      <dependency>
         <groupId>io.spring.platform</groupId>
         <artifactId>platform-bom</artifactId>
         <version>Brussels-SR4</version>
         <type>pom</type>
         <scope>import</scope>
      </dependency>
      <dependency>
         <groupId>org.springframework.cloud</groupId>
         <artifactId>spring-cloud-dependencies</artifactId>
         <version>Dalston.SR2</version>
         <type>pom</type>
         <scope>import</scope>
      </dependency>
   </dependencies>
</dependencyManagement>
```









