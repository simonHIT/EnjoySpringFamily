# 访问 Web 资源

## 通过 RestTemplate 访问 Web 资源

- Spring Boot 中的 RestTemplate

  - Spring Boot 中没有自动配置 RestTemplate

  - Spring Boot 提供了 RestTemplateBuilder

  - RestTemplateBuilder.build()

- 常用方法

  - GET 请求

    - getForObject() / getForEntity()

  - POST 请求

    - postForObject() / postForEntity()

  - PUT 请求

    - put()

  - DELETE 请求

    - delete()

- 构造 URI
  
  - 构造 URI

    - UriComponentsBuilder
  
  - 构造相对于当前请求的 URI

    - ServletUriComponentsBuilder
  
  - 构造指向 Controller 的 URI

    - MvcUriComponentsBuilder

---
---

- RestTemplate 的高阶用法

  - 传递 HTTP Header

    - RestTemplate.exchange()

    - RequestEntity<T> / ResponseEntity<T>

  - HTTP Header
  
  ![](images/spring-rest-template-04.png)