# Spring Boot杂记

## Spring Boot异常处理

文档所在：https://docs.spring.io/spring-boot/docs/2.0.6.RELEASE/reference/htmlsingle/#boot-features-error-handling



关注两个类：

+ BasicErrorController
+ ErrorMvcAutoConfiguration





### 对于所有错误进行统一定义（可控）

前置条件：

1.错误种类

```java

/**
 * 错误类型
 */
public enum ErrorType {
    ID_IS_NULL("F001", "编号不可为空"),
    UNKNOWN_ERROR("999", "未知异常");

    private String code;
    private String message;

    ErrorType(String code, String message) {
        this.code = code;
        this.message = message;
    }

    /**
     * 根据名称获取错误类型
     *
     * @param name
     * @return
     */
    public static ErrorType getByName(String name) {
        for (ErrorType type : ErrorType.values()) {
            if (type.name().equals(name)) {
                return type;
            }
        }
        return ErrorType.UNKNOWN_ERROR;
    }

    public String getCode() {
        return code;
    }

    public String getMessage() {
        return message;
    }
}

```



2.自定义错误处理controller

```java
import org.springframework.boot.autoconfigure.web.ErrorProperties;
import org.springframework.boot.autoconfigure.web.servlet.error.BasicErrorController;
import org.springframework.boot.autoconfigure.web.servlet.error.ErrorViewResolver;
import org.springframework.boot.web.servlet.error.ErrorAttributes;

import javax.servlet.http.HttpServletRequest;
import java.util.List;
import java.util.Map;

/**
 * 自定义错误处理controller
 */
public class MyErrorController extends BasicErrorController{
    public MyErrorController(ErrorAttributes errorAttributes, ErrorProperties errorProperties, List<ErrorViewResolver> errorViewResolvers) {
        super(errorAttributes, errorProperties, errorViewResolvers);
    }
    /**
     * {
     x"timestamp": "2018-01-14 10:41:17",
     x"status": 500,
     x"error": "Internal Server Error",
     x"exception": "java.lang.IllegalArgumentException",
     "message": "编号不可为空",
     x"path": "/manager/products"
     + code
     + canRetry
     }
     */
    @Override
    protected Map<String, Object> getErrorAttributes(HttpServletRequest request, boolean includeStackTrace) {
        Map<String, Object> attrs = super.getErrorAttributes(request, includeStackTrace);
        attrs.remove("timestamp");
        attrs.remove("status");
        attrs.remove("error");
        attrs.remove("exception");
        attrs.remove("path");
        String errorCode = (String) attrs.get("message");
        ErrorEnum errorEnum = ErrorEnum.getByCode(errorCode);
        attrs.put("message",errorEnum.getMessage());
        attrs.put("code",errorEnum.getCode());
        attrs.put("canRetry",errorEnum.isCanRetry());
        return attrs;
    }
}


```

2.注册错误处理controller到spring boot:

```java
import org.springframework.beans.factory.ObjectProvider;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.SearchStrategy;
import org.springframework.boot.autoconfigure.web.*;
import org.springframework.boot.autoconfigure.web.servlet.error.ErrorViewResolver;
import org.springframework.boot.web.servlet.error.ErrorAttributes;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.util.List;

/**
 * 错误处理相关配置
 */
//@Configuration
public class ErrorConfiguration {
    @Bean
    public MyErrorController basicErrorController(ErrorAttributes errorAttributes, ServerProperties serverProperties,
                                                  ObjectProvider<List<ErrorViewResolver>> errorViewResolversProvider) {
        return new MyErrorController(errorAttributes, serverProperties.getError(),
                errorViewResolversProvider.getIfAvailable());
    }
}

```



### 抛出异常处理（不可控）

```java

import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.util.Assert;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.ResponseBody;

import java.util.HashMap;
import java.util.Map;

/**
 * 统一错误处理
 */
@ControllerAdvice(basePackages = {"com.imooc.manager.controller"})
public class ErrorControllerAdvice {

    //可以自定义自己的异常
    @ExceptionHandler(Exception.class)
    @ResponseBody
    public ResponseEntity handleException(Exception e){
        Map<String, Object> attrs = new HashMap();
        String errorCode = e.getMessage();
        ErrorEnum errorEnum = ErrorEnum.getByCode(errorCode);
        attrs.put("message",errorEnum.getMessage());
        attrs.put("code",errorEnum.getCode());
        attrs.put("canRetry",errorEnum.isCanRetry());
        attrs.put("type","advice");
        Assert.isNull(attrs,"advice");
        return new ResponseEntity(attrs, HttpStatus.INTERNAL_SERVER_ERROR);
    }
}
```



## Spring Boot测试

### Spring Boot冒烟测试

对暴露的API进行测试，使用postman即可



### Spring Boot自动化测试

####  功能测试

使用junit进行测试：

1.测试类添加：

**@RunWith(SpringRunner.class)**
**@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)**

webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT表示运行测试类时随机启动一个接口。



2.测试方法注解

@BeforeClass：针对所有测试，只执行一次，且必须为static void 

@Before

@Test====>Assert

@Ignore：忽略的测试方法 

@After

@AfterClass：针对所有测试，只执行一次，且必须为static void 



3.测试方法的执行顺序：

@FixMethodOrder(MethodSorters.NAME_ASCENDING)

按照字典殊勋



4.测试覆盖率：

正确数据、错误数据、边界检测、错误消息是否符合预期



## Spring Boot文档

接口文档场景：

+ 前后端分离
+ 第三方合作
+ 二次开发系统



文档工具：Swagger（官方文档：https://swagger.io/solutions/api-documentation/）



