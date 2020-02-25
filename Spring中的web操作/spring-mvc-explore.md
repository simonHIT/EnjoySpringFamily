# 谈谈 Web 那些事-Spring MVC 实践

## 编写第一个Spring MVC Controller

- 认识 Spring MVC

  - DispatcherServlet

  - Controller

  - xxxResolver

  - ViewResolver

  - HandlerExceptionResolver

  - MultipartResolver

  - HandlerMapping

- Spring MVC 中的常⽤用注解

  - @Controller

  - @RestController

  - @RequestMapping

  - @GetMapping / @PostMapping

  - @PutMapping / @DeleteMapping

  - @RequestBody / @ResponseBody / @ResponseStatus

---
---

## 理解 Spring 的应⽤上下⽂

- Spring 的应⽤程序上下⽂
  
  ![Spring 的应⽤程序上下⽂](images/spring-mvc-explore-01.png)

  ![Spring 的应⽤程序上下⽂](images/spring-mvc-explore-02.png)

- 关于上下文常用的接⼝

  - BeanFactory

  - DefaultListableBeanFactory

  - ApplicationContext

  - ClassPathXmlApplicationContext

  - FileSystemXmlApplicationContext

  - AnnotationConfigApplicationContext

  - WebApplicationContext

- Web 上下文层次

  ![Web 上下文层次](images/spring-mvc-explore-03.png)

  ![Web 上下文层次](images/spring-mvc-explore-04.png)

  ![Web 上下文层次](images/spring-mvc-explore-04.png)

---
---

## Spring MVC 中的各种机制-请求处理

- Spring MVC 的请求处理流程

  ![请求处理流程](images/spring-mvc-explore-06.png)

- 一个请求的大致处理流程
  - 绑定一些 Attribute

    - WebApplicationContext / LocaleResolver / ThemeResolver
  - 处理 Multipart

    - 如果是，则将请求转为 MultipartHttpServletRequest
  - Handler 处理

    - 如果找到对应 Handler，执⾏ Controller 及前后置处理器逻辑
  - 处理返回的 Model ，呈现视图

- 如何定义处理方法

  - 定义映射关系
    - @Controller
    - @RequestMapping

      - path / method 指定映射路路径与⽅方法

      - params / headers 限定映射范围

      - consumes / produces 限定请求与响应格式
    - 一些快捷⽅方式

      - @RestController

      - @GetMapping / @PostMapping / @PutMapping / @DeleteMapping / @PatchMapping

  - 定义处理方法

    - @RequestBody / @ResponseBody / @ResponseStatus

    - @PathVariable / @RequestParam / @RequestHeader

    - HttpEntity / ResponseEntity

    - 详细参数

      - <https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/web.html#mvc-ann-arguments>

    - 详细返回

    - <https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/web.html#mvc-ann-return-types>

---
  
  - 方法示例
  
    ![Controller](images/spring-mvc-explore-07.png)
    ![Controller](images/spring-mvc-explore-08.png)

---

  - 定义类型转换
    - 自⼰实现 WebMvcConfigurer

      - Spring Boot 在 WebMvcAutoConfiguration 中实现了一个

      - 添加⾃自定义的 Converter

      - 添加⾃自定义的 Formatter

  - 定义校验
    - 通过 Validator 对绑定结果进行校验
      - Hibernate Validator

    - @Valid 注解

    - BindingResult

  - Multipart 上传
    - 配置 MultipartResolver

      - Spring Boot 自动配置 MultipartAutoConfiguration

    - 支持类型 multipart/form-data

    - MultipartFile 类型

---
---

## Spring MVC 中的各种机制-视图处理

- 视图解析的实现基础
  
  - ViewResolver 与 View 接⼝

    - AbstractCachingViewResolver

    - UrlBasedViewResolver

    - FreeMarkerViewResolver

    - ContentNegotiatingViewResolver

    - InternalResourceViewResolver

- DispatcherServlet 中的视图解析逻辑


  - initStrategies()

    - initViewResolvers() 初始化了对应 ViewResolver

  - doDispatch()

    - processDispatchResult()

      - 没有返回视图的话，尝试 RequestToViewNameTranslator

      - resolveViewName() 解析 View 对象

  - 使用 @ResponseBody 的情况

  - 在 HandlerAdapter.handle() 的中完成了Response 输出

    - RequestMappingHandlerAdapter.invokeHandlerMethod()

      - HandlerMethodReturnValueHandlerComposite.handleReturnValue()

        - RequestResponseBodyMethodProcessor.handleReturnValue()

- 重定向

  - 两种不同的重定向前缀

    - redirect:

    - forward:

## Spring MVC 中的常用视图

- Spring MVC 支持的视图
  - 支持的视图列表

    - <https://docs.spring.io/spring/docs/5.1.5.RELEASE/spring-framework-reference/web.html#mvc-view>

    - Jackson-based JSON / XML

    - Thymeleaf & FreeMarker

  ![支持的视图列表](images/spring-mvc-explore-09.png)

- 配置 MessageConverter
  - 通过 WebMvcConfigurer 的 configureMessageConverters()

    - Spring Boot 自动查找 HttpMessageConverters 进⾏注册
  ![Spring Boot 自动注册](images/spring-mvc-explore-10.png)

- Spring Boot 对 Jackson 的支持

  - JacksonAutoConfiguration

    - Spring Boot 通过 @JsonComponent 注册 JSON 序列列化组件

    - Jackson2ObjectMapperBuilderCustomizer

  - JacksonHttpMessageConvertersConfiguration

    - 增加 jackson-dataformat-xml 以支持 XML 序列化

---

---

## 使用 Thymeleaf

- 添加 Thymeleaf 依赖

  - org.springframework.boot:spring-boot-starter-thymeleaf
  
- Spring Boot 的自动配置

  - ThymeleafAutoConfiguration

  - ThymeleafViewResolver

- Thymeleaf 的一些默认配置

    ```yml
    spring.thymeleaf.cache=true

    spring.thymeleaf.check-template=true

    spring.thymeleaf.check-template-location=true

    spring.thymeleaf.enabled=true

    spring.thymeleaf.encoding=UTF-8

    spring.thymeleaf.mode=HTML

    spring.thymeleaf.servlet.content-type=text/html

    spring.thymeleaf.prefix=classpath:/templates/

    spring.thymeleaf.suffix=.html
    ```

---
---

## 静态资源与缓存

- Spring Boot 中的静态资源配置
  - 核心逻辑

    - WebMvcConfigurer.addResourceHandlers()
  - 常⽤配置

    - spring.mvc.static-path-pattern=/**

    - spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/

## Spring Boot 中的缓存配置

- 常⽤配置（默认时间单位都是秒）

  - ResourceProperties.Cache

  - spring.resources.cache.cachecontrol.max-age=时间

  - spring.resources.cache.cachecontrol.no-cache=true/false

  - spring.resources.cache.cachecontrol.s-max-age=时间

- Controller 中⼿工设置缓存
  
  ![Controller 中⼿工设置缓存](images/spring-mvc-explore-11.png)

  ---
  ---

- 建议的资源访问方式
  
  ![建议的资源访问方式](images/spring-mvc-explore-12.png)

## Spring MVC 中的各种机制-异常处理

- Spring MVC 的异常解析
  - 核心接⼝

    - HandlerExceptionResolver
  - 实现类

    - SimpleMappingExceptionResolver

    - DefaultHandlerExceptionResolver

    - ResponseStatusExceptionResolver

    - ExceptionHandlerExceptionResolver
  
  - 异常处理方法
  
    - 处理方法

      - @ExceptionHandler
    - 添加位置

      - @Controller / @RestController

      - @ControllerAdvice / @RestControllerAdvice

---
---

## 了解 Spring MVC 的切入点

- Spring MVC 的拦截器

  - 核心接⼝

    - HandlerInteceptor

    - boolean preHandle()

    - void postHandle()

    - void afterCompletion()
  
  - 针对 @ResponseBody 和 ResponseEntity 的情况

    - ResponseBodyAdvice
  
  - 针对异步请求的接⼝

    - AsyncHandlerInterceptor

    - void afterConcurrentHandlingStarted()

  - 拦截器的配置方式
  
    - 常规方法

      - WebMvcConfigurer.addInterceptors()
  
    - Spring Boot 中的配置

      - 创建⼀一个带 @Configuration 的 WebMvcConfigurer 配置类

      - 不能带 @EnableWebMvc（想彻底自己控制 MVC 配置除外）

---
---

---

## SpringBucks 进度小结

- 本章小结

  - 解释了什么是 Spring 的 ApplicationContext

  - 了解了 Spring MVC 的基本使⽤

  - 理解 Spring MVC 的多种机制

- SpringBucks 进度小结

  - 拆分了 waiter-service

  - 增加了更多 REST ⽅方法

  - 增加了缓存、性能日志与异常处理
  
---
