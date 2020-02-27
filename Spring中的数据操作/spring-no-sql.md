# NoSQL 实践

[TOC]

------



## 通过 Docker 辅助开发

- 什么是容器

    ![什么是容器](images/spring-nosql-01.png)

- 认识 Docker

    ![认识Docker](images/spring-nosql-02.png)

- 不同人眼中的 Docker
  - 开发眼中的 Docker
    - 简化了了重复搭建开发环境的⼯工作
    - 运维眼中的 Docker
  - 交付系统更更为流畅
    - 伸缩性更更好

- Docker 常⽤用命令
  - 镜像相关
    - docker pull <image>
    - docker search <image>
  - 容器器相关
    - docker run
    - docker start/stop <容器器名>
    - docker ps <容器器名>
    - docker logs <容器器名>

- docker run 的常⽤用选项
  - docker run [OPTIONS] IMAGE [COMMAND] [ARG…]
  - 选项说明
    - -d，后台运⾏行行容器器
    - -e，设置环境变量量
    - --expose / -p 宿主端⼝口:容器器端⼝口
    - --name，指定容器器名称
    - --link，链接不不同容器器
    - -v 宿主⽬目录:容器器⽬目录，挂载磁盘卷

- 国内 Docker 镜像配置
  - 官⽅ Docker Hub
    - https://hub.docker.com
  - 官⽅镜像
    - 镜像 https://www.docker-cn.com/registry-mirror
    - 下载 https://www.docker-cn.com/get-docker
  - 阿⾥里里云镜像
    - https://dev.aliyun.com

- 通过 Docker 启动 MongoDB
  - 官⽅方指引 
    - https://hub.docker.com/_/mongo
  - 获取镜像
    - docker pull mongo
  - 运⾏ MongoDB 镜像
    - docker run --name mongo -p 27017:27017 -v ~/docker-data/mongo:/data/db -e MONGO_INITDB_ROOT_USERNAME=admin -e MONGO_INITDB_ROOT_PASSWORD=admin -d mongo

  - 登录到 MongoDB 容器器中
    - docker exec -it mongo bash
  - 通过 Shell 连接 MongoDB
    - mongo -u admin -p admin

## 在 Spring 中访问 MongoDB

- Spring 对 MongoDB 的支持

  - MongoDB 是一款开源的文档型数据库
    - https://www.mongodb.com
  - Spring 对 MongoDB 的⽀支持
    - Spring Data MongoDB
      - MongoTemplate
      - Repository 支持

- Spring Data MongoDB 的基本⽤用法

  - 注解
    - @Document
    - @Id
  - MongoTemplate
    - save / remove
    - Criteria / Query / Update
  
- 初始化 MongoDB 的库及权限
  - 创建库
    
    - use springbucks;
- 创建⽤用户
  
    ```sql
    db.createUser(
    {
        user: "springbucks",
        pwd: "springbucks",
        roles: [
            { role: "readWrite", db: "springbucks" }
        ]
    }
    )
    ```

---
- mongo-demo
- MoneyReadConvertor.java

  ```java
  public class MoneyReadCovertor implements Converter<Document, Money> {
      @Override
      public Money convert(Document document) {
          //return null;
          Document money = (Document) document.get("money");

          double amount = Double.parseDouble(money.getString("amount"));

          String currency = ((Document) money.get("currency")).getString("code");

          return Money.of(CurrencyUnit.of(currency), amount);
      }
  }
  ```

- Money.java

  ```java
  @Document
  @Builder
  @NoArgsConstructor
  @AllArgsConstructor
  @Data
  public class Coffee {

      @Id
      private String id;

      private String name;

      private Money price;

      private Date createTime;

      private Date updateTime;

  }
  ```

- MongoDemoApplication.java

  ```java
  @SpringBootApplication
  @Slf4j
  public class MongoDemoApplication implements ApplicationRunner {

      @Autowired
      private MongoTemplate mongoTemplate;

      public static void main(String[] args) {
          SpringApplication.run(MongoDemoApplication.class, args);
      }

      @Bean
      public MongoCustomConversions mongoCustomConversions() {
          return new MongoCustomConversions(Arrays.asList(new MoneyReadCovertor()));
      }

      @Override
      public void run(ApplicationArguments args) throws Exception {

          Coffee espresso = Coffee.builder()
                  .name("espresso")
                  .price(Money.of(CurrencyUnit.of("CNY"), 20.0))
                  .createTime(new Date())
                  .updateTime(new Date()).build();

          Coffee saved = mongoTemplate.save(espresso);

          log.info("Coffee {}", saved);

          List<Coffee> coffees = mongoTemplate.find(
                  Query.query(Criteria.where("name").is("espresso")), Coffee.class
          );

          log.info("find {} coffee", coffees.size());

          coffees.forEach(c -> log.info("Coffee {}", c));

          Thread.sleep(1000);

          UpdateResult updateResult = mongoTemplate.updateFirst(Query.query(Criteria.where("name").is("espresso")),
                  new Update().set("price", Money.ofMajor(CurrencyUnit.of("CNY"), 30)).currentDate("updateTime"),
                  Coffee.class);

          log.info("Update result: {}", updateResult.getModifiedCount());

          Coffee updateOne = mongoTemplate.findById(saved.getId(), Coffee.class);

          log.info("Update result:{}", updateOne);

          mongoTemplate.remove(updateOne);
      }
  }
  ```
- application.properties

  ```yml
  spring.data.mongodb.uri=mongodb://springbucks:springbucks@localhost:27017/springbucks
  ```

- 运行结果

```yml
2020-01-12 16:52:42.431  INFO 1952 --- [           main] c.simon.mongodemo.MongoDemoApplication   : Started MongoDemoApplication in 4.304 seconds (JVM running for 5.834)
2020-01-12 16:52:42.799  INFO 1952 --- [           main] org.mongodb.driver.connection            : Opened connection [connectionId{localValue:2, serverValue:4}] to localhost:27017
2020-01-12 16:52:42.848  INFO 1952 --- [           main] c.simon.mongodemo.MongoDemoApplication   : Coffee Coffee(id=5e1ade5a6de7a849c20e45c5, name=espresso, price=CNY 20.00, createTime=Sun Jan 12 16:52:42 CST 2020, updateTime=Sun Jan 12 16:52:42 CST 2020)
2020-01-12 16:52:42.995  INFO 1952 --- [           main] c.simon.mongodemo.MongoDemoApplication   : find 1 coffee
2020-01-12 16:52:42.995  INFO 1952 --- [           main] c.simon.mongodemo.MongoDemoApplication   : Coffee Coffee(id=5e1ade5a6de7a849c20e45c5, name=espresso, price=CNY 20.00, createTime=Sun Jan 12 16:52:42 CST 2020, updateTime=Sun Jan 12 16:52:42 CST 2020)
2020-01-12 16:52:44.044  INFO 1952 --- [           main] c.simon.mongodemo.MongoDemoApplication   : Update result: 1
2020-01-12 16:52:44.049  INFO 1952 --- [           main] c.simon.mongodemo.MongoDemoApplication   : Update result:Coffee(id=5e1ade5a6de7a849c20e45c5, name=espresso, price=CNY 30.00, createTime=Sun Jan 12 16:52:42 CST 2020, updateTime=Sun Jan 12 16:52:44 CST 2020)
2020-01-12 16:52:44.067  INFO 1952 --- [extShutdownHook] org.mongodb.driver.connection            : Closed connection [connectionId{localValue:2, serverValue:4}] to localhost:27017 because the pool has been closed.
```

---

- Spring Data MongoDB 的 Repository

  - 启用注解@EnableMongoRepositories
  - 对应接口
    - MongoRepository<T, ID>
    - PagingAndSortingRepository<T, ID>
    - CrudRepository<T, ID>

---

- mongo-repository-demo

- application.properties
  
  ```yml
    spring.data.mongodb.uri=mongodb://springbucks:springbucks@localhost:27017/springbucks
  ```

- money.java

  ```java
  同mongo-demo
  ```

- MoneyReadConvertor.java

  ```java
  同mongo-demo
  ```

- CoffeeRepository.java

  ```java
  public interface CoffeeRepository extends MongoRepository<Coffee,String> {

      List<Coffee> findByName(String name);
  }
  ```

- MongoReositoryDemoApplication.java

  ```java
  @SpringBootApplication
  @Slf4j
  @EnableMongoRepositories
  public class MongoRepositoryDemoApplication implements ApplicationRunner {

      @Autowired
      private CoffeeRepository coffeeRepository;

      public static void main(String[] args) {
          SpringApplication.run(MongoRepositoryDemoApplication.class, args);
      }

      @Bean
      public MongoCustomConversions mongoCustomConversions() {

          return new MongoCustomConversions(Arrays.asList(new MoneyReadCovertor()));
      }

      @Override
      public void run(ApplicationArguments args) throws Exception {

          Coffee espresso = Coffee.builder()
                  .name("espresso")
                  .price(Money.of(CurrencyUnit.of("CNY"), 20.0))
                  .createTime(new Date())
                  .updateTime(new Date())
                  .build();

          Coffee latte = Coffee.builder()
                  .name("latte")
                  .price(Money.of(CurrencyUnit.of("CNY"), 30.0))
                  .updateTime(new Date())
                  .createTime(new Date())
                  .build();

          coffeeRepository.insert(Arrays.asList(espresso, latte));

          coffeeRepository.findAll(Sort.by("name"))
                  .forEach(c -> log.info("Coffee saved as {}", c));

          Thread.sleep(1000);

          latte.setPrice(Money.of(CurrencyUnit.of("CNY"), 35.00));
          latte.setUpdateTime(new Date());
          coffeeRepository.save(latte);
          coffeeRepository.findByName("latte")
                  .forEach(coffee -> log.info("Latte saved as {}", coffee));

          //coffeeRepository.deleteAll();
      }
  }
  ```

- 运行结果

  ```yml
  2020-01-12 20:20:10.239  INFO 3722 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data MongoDB repositories in DEFAULT mode.
  2020-01-12 20:20:10.323  INFO 3722 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 78ms. Found 1 MongoDB repository interfaces.
  2020-01-12 20:20:11.095  INFO 3722 --- [           main] org.mongodb.driver.cluster               : Cluster created with settings {hosts=[localhost:27017], mode=SINGLE, requiredClusterType=UNKNOWN, serverSelectionTimeout='30000 ms', maxWaitQueueSize=500}
  2020-01-12 20:20:11.182  INFO 3722 --- [localhost:27017] org.mongodb.driver.connection            : Opened connection [connectionId{localValue:1, serverValue:6}] to localhost:27017
  2020-01-12 20:20:11.194  INFO 3722 --- [localhost:27017] org.mongodb.driver.cluster               : Monitor thread successfully connected to server with description ServerDescription{address=localhost:27017, type=STANDALONE, state=CONNECTED, ok=true, version=ServerVersion{versionList=[4, 2, 2]}, minWireVersion=0, maxWireVersion=8, maxDocumentSize=16777216, logicalSessionTimeoutMinutes=30, roundTripTimeNanos=9115465}
  2020-01-12 20:20:11.933  INFO 3722 --- [           main] c.s.m.MongoRepositoryDemoApplication     : Started MongoRepositoryDemoApplication in 2.856 seconds (JVM running for 4.238)
  2020-01-12 20:20:12.301  INFO 3722 --- [           main] org.mongodb.driver.connection            : Opened connection [connectionId{localValue:2, serverValue:7}] to localhost:27017
  2020-01-12 20:20:12.478  INFO 3722 --- [           main] c.s.m.MongoRepositoryDemoApplication     : Coffee saved as Coffee(id=5e1b0efcffd5436180e4f02b, name=espresso, price=CNY 20.00, createTime=Sun Jan 12 20:20:11 CST 2020, updateTime=Sun Jan 12 20:20:11 CST 2020)
  2020-01-12 20:20:12.479  INFO 3722 --- [           main] c.s.m.MongoRepositoryDemoApplication     : Coffee saved as Coffee(id=5e1b0efcffd5436180e4f02c, name=latte, price=CNY 30.00, createTime=Sun Jan 12 20:20:11 CST 2020, updateTime=Sun Jan 12 20:20:11 CST 2020)
  2020-01-12 20:20:13.671  INFO 3722 --- [           main] c.s.m.MongoRepositoryDemoApplication     : Latte saved as Coffee(id=5e1b0efcffd5436180e4f02c, name=latte, price=CNY 35.00, createTime=Sun Jan 12 20:20:11 CST 2020, updateTime=Sun Jan 12 20:20:13 CST 2020)
  2020-01-12 20:20:13.679  INFO 3722 --- [extShutdownHook] org.mongodb.driver.connection            : Closed connection [connectionId{localValue:2, serverValue:7}] to localhost:27017 because the pool has been closed.
  ```

---

## 在 Spring 中访问 Redis

- Spring 对 Redis 的支持
  - Redis 是⼀一款开源的内存 KV 存储，⽀支持多种数据结构
    - https://redis.io
  - Spring 对 Redis 的支持
    - Spring Data Redis
      - 支持的客户端 Jedis / Lettuce
      - RedisTemplate
      - Repository 支持

- Jedis 客户端的简单使⽤用
  - Jedis 不不是线程安全的
  - 通过 JedisPool 获得 Jedis 实例
  - 直接使用 Jedis 中的⽅方法

- 通过 Docker 启动 Redis

  - 官方指引
    - https://hub.docker.com/_/redis
  - 获取镜像
    - docker pull redis
  - 启动 Redis
    - docker run --name redis -d -p 6379:6379 redis

---

- cache-demo & cache-with-redis-demo
- cache-demo application.properties

```yml
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.format_sql=true
```

- cache-with-redis-demo application.properties

```yml
spring.jpa.hibernate.ddl-auto=none
spring.jpa.properties.hibernate.show_sql=true
spring.jpa.properties.hibernate.format_sql=true

management.endpoints.web.exposure.include=*

spring.cache.type=redis
spring.cache.cache-names=coffee
spring.cache.redis.time-to-live=5000
spring.cache.redis.cache-null-values=false

spring.redis.host=localhost
```

- CoffeeService.java


  ```java
  @Slf4j
  @Service
  @CacheConfig(cacheNames = "coffee")
  public class CoffeeService {
      @Autowired
      private CoffeeRepository coffeeRepository;

      @Cacheable
      public List<Coffee> findAllCoffee() {
          return coffeeRepository.findAll();
      }

      @CacheEvict
      public void reloadCoffee() {
      }

      public Optional<Coffee> findOneCoffee(String name) {
          ExampleMatcher matcher = ExampleMatcher.matching()
                  .withMatcher("name", exact().ignoreCase());
          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher));
          log.info("Coffee Found: {}", coffee);
          return coffee;
      }
  }
  ```

- Application.java

  ```java
  @SpringBootApplication
  @Slf4j
  @EnableJpaRepositories
  @EnableCaching(proxyTargetClass = true)
  @EnableTransactionManagement
  public class CacheWithRedisDemoApplication implements ApplicationRunner {

      @Autowired
      private CoffeeService coffeeService;

      public static void main(String[] args) {
          SpringApplication.run(CacheWithRedisDemoApplication.class, args);
      }

      @Override
      public void run(ApplicationArguments args) throws Exception {

          log.info("Count: {}", coffeeService.findAllCoffee().size());
          for (int i = 0; i < 5; i++) {
              log.info("Reading from cache.");
              coffeeService.findAllCoffee();
          }
          Thread.sleep(5_000);
          log.info("Reading after refresh.");
          coffeeService.findAllCoffee().forEach(c -> log.info("Coffee {}", c.getName()));
      }
  }
  ```
- 运行结果

  ```yml
  2020-01-12 22:44:30.169  INFO 6243 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 97ms. Found 2 JPA repository interfaces.
  2020-01-12 22:44:30.659  INFO 6243 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Multiple Spring Data modules found, entering strict repository configuration mode!
  2020-01-12 22:44:30.660  INFO 6243 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Bootstrapping Spring Data Redis repositories in DEFAULT mode.
  2020-01-12 22:44:30.749  INFO 6243 --- [           main] .RepositoryConfigurationExtensionSupport : Spring Data Redis - Could not safely identify store assignment for repository candidate interface com.simon.cachewithredisdemo.repository.CoffeeOrderRepository. If you want this repository to be a Redis repository, consider annotating your entities with one of these annotations: org.springframework.data.redis.core.RedisHash (preferred), or consider extending one of the following types with your repository: org.springframework.data.keyvalue.repository.KeyValueRepository.
  2020-01-12 22:44:30.751  INFO 6243 --- [           main] .RepositoryConfigurationExtensionSupport : Spring Data Redis - Could not safely identify store assignment for repository candidate interface com.simon.cachewithredisdemo.repository.CoffeeRepository. If you want this repository to be a Redis repository, consider annotating your entities with one of these annotations: org.springframework.data.redis.core.RedisHash (preferred), or consider extending one of the following types with your repository: org.springframework.data.keyvalue.repository.KeyValueRepository.
  2020-01-12 22:44:30.751  INFO 6243 --- [           main] .s.d.r.c.RepositoryConfigurationDelegate : Finished Spring Data repository scanning in 18ms. Found 0 Redis repository interfaces.
  2020-01-12 22:44:31.775  INFO 6243 --- [           main] trationDelegate$BeanPostProcessorChecker : Bean 'org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration' of type [org.springframework.transaction.annotation.ProxyTransactionManagementConfiguration] is not eligible for getting processed by all BeanPostProcessors (for example: not eligible for auto-proxying)
  2020-01-12 22:44:32.343  INFO 6243 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
  2020-01-12 22:44:32.704  INFO 6243 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
  2020-01-12 22:44:33.386  INFO 6243 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
  2020-01-12 22:44:33.746  INFO 6243 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.4.9.Final}
  2020-01-12 22:44:34.265  INFO 6243 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.0.Final}
  2020-01-12 22:44:34.645  INFO 6243 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
  2020-01-12 22:44:36.406  INFO 6243 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
  2020-01-12 22:44:36.453  INFO 6243 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
  2020-01-12 22:44:38.237  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Started CacheWithRedisDemoApplication in 10.701 seconds (JVM running for 12.158)
  2020-01-12 22:44:38.516  INFO 6243 --- [           main] io.lettuce.core.EpollProvider            : Starting without optional epoll library
  2020-01-12 22:44:38.518  INFO 6243 --- [           main] io.lettuce.core.KqueueProvider           : Starting without optional kqueue library
  Hibernate: 
      select
          coffee0_.id as id1_0_,
          coffee0_.create_time as create_t2_0_,
          coffee0_.update_time as update_t3_0_,
          coffee0_.name as name4_0_,
          coffee0_.price as price5_0_ 
      from
          t_coffee coffee0_
  2020-01-12 22:44:39.287  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Count: 5
  2020-01-12 22:44:39.287  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Reading from cache.
  2020-01-12 22:44:39.293  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Reading from cache.
  2020-01-12 22:44:39.297  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Reading from cache.
  2020-01-12 22:44:39.300  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Reading from cache.
  2020-01-12 22:44:39.305  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Reading from cache.
  2020-01-12 22:44:44.312  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Reading after refresh.
  Hibernate: 
      select
          coffee0_.id as id1_0_,
          coffee0_.create_time as create_t2_0_,
          coffee0_.update_time as update_t3_0_,
          coffee0_.name as name4_0_,
          coffee0_.price as price5_0_ 
      from
          t_coffee coffee0_
  2020-01-12 22:44:44.323  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Coffee espresso
  2020-01-12 22:44:44.323  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Coffee latte
  2020-01-12 22:44:44.323  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Coffee capuccino
  2020-01-12 22:44:44.324  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Coffee mocha
  2020-01-12 22:44:44.324  INFO 6243 --- [           main] c.s.c.CacheWithRedisDemoApplication      : Coffee macchiato
  2020-01-12 22:44:44.331  INFO 6243 --- [extShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
  2020-01-12 22:44:44.334  INFO 6243 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
  2020-01-12 22:44:44.338  INFO 6243 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
  ```

---

- Redis 的哨兵与集群模式

  - Redis 的哨兵模式
    - Redis Sentinel 是 Redis 的一种⾼可用⽅案
      - 监控、通知、⾃自动故障转移、服务发现
    - JedisSentinelPool

  - Redis 的集群模式
    - Redis Cluster
      - 数据⾃自动分⽚片（分成16384个 Hash Slot ）
      - 在部分节点失效时有⼀一定可⽤用性
    - JedisCluster
      - Jedis 只从 Master 读数据，如果想要⾃自动读写分离，可以定制

## 了解 Spring 的缓存抽象

- Spring 的缓存抽象
  - 为不同的缓存提供一层抽象
  - 为 Java 方法增加缓存，缓存执行结果
  - 支持ConcurrentMap、EhCache、Caffeine、JCache（JSR-107）
  - 接⼝
    - org.springframework.cache.Cache
    - org.springframework.cache.CacheManager

- 基于注解的缓存

  - @EnableCaching(开启缓存)
    - @Cacheable（声明缓存,如果缓存无目标值则添加缓存，有则取缓存值）
    - @CacheEvict（清除缓存）
    - @CachePut（无论缓存中是否有目标值，均添加至缓存）
    - @Caching（缓存打包，配置多个缓存）
    - @CacheConfig（缓存配置，缓存名称等属性配置）

- 缓存的使用场景

- 对于经久不变（一天之内或更久）的数据，建议在页面内部做一个缓存，避免使用分布式缓存访问后端缓存的网络开销
- 对于不经常变化或者允许变化有一定的延迟（一天之内或更久）的数据，建议在页面内部做一个缓存并设置一个失效时间
- 对于经常变化且读写要求一致性的数据，建议使用redis等分布式高可用集群做一个缓存
- 对于读写比例高的数据，可以使用redis等缓存，对于读写比例不好的数据（如1：1）等不建议使用缓存

---

  ```java
  @Slf4j
  @Service
  @CacheConfig(cacheNames = "coffee")
  public class CoffeeService {
      @Autowired
      private CoffeeRepository coffeeRepository;

      @Cacheable
      public List<Coffee> findAllCoffee() {
          return coffeeRepository.findAll();
      }

      @CacheEvict
      public void reloadCoffee() {
      }

      public Optional<Coffee> findOneCoffee(String name) {
          ExampleMatcher matcher = ExampleMatcher.matching()
                  .withMatcher("name", exact().ignoreCase());
          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher));
          log.info("Coffee Found: {}", coffee);
          return coffee;
      }
  }
  ```

---

- 通过 Spring Boot 配置 Redis 缓存

- pom依赖

  ```xml
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-cache</artifactId>
          </dependency>
          <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-redis</artifactId>
          </dependency>
  ```

- application.properties

  ```yml
  spring.jpa.hibernate.ddl-auto=none
  spring.jpa.properties.hibernate.show_sql=true
  spring.jpa.properties.hibernate.format_sql=true

  management.endpoints.web.exposure.include=*

  spring.cache.type=redis
  spring.cache.cache-names=coffee
  spring.cache.redis.time-to-live=5000
  spring.cache.redis.cache-null-values=false

  spring.redis.host=localhost
  ```

- CoffeeService.java

  ```java
  @Slf4j
  @Service
  @CacheConfig(cacheNames = "coffee")
  public class CoffeeService {
      @Autowired
      private CoffeeRepository coffeeRepository;

      @Cacheable
      public List<Coffee> findAllCoffee() {
          return coffeeRepository.findAll();
      }

      @CacheEvict
      public void reloadCoffee() {
      }

      public Optional<Coffee> findOneCoffee(String name) {
          ExampleMatcher matcher = ExampleMatcher.matching()
                  .withMatcher("name", exact().ignoreCase());
          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher));
          log.info("Coffee Found: {}", coffee);
          return coffee;
      }
  }
  ```

- Redis 在 Spring 中的其他用法

- 与 Redis 建⽴连接
  - 配置连接⼯厂
  - LettuceConnectionFactory 与 JedisConnectionFactory
    - RedisStandaloneConfiguration
    - RedisSentinelConfiguration
    - RedisClusterConfiguration

- 读写分离

  - Lettuce 内置支持读写分离
    - 只读主、只读从
    - 优先读主、优先读从
  - LettuceClientConfiguration
  - LettucePoolingClientConfiguration
  - LettuceClientConfigurationBuilderCustomizer

- RedisTemplate
  - RedisTemplate<K, V>
    - opsForXxx()
  - StringRedisTemplate
  - 一定注意设置过期时间！！！

---

- CoffeeService.java

  ```java
  @Slf4j
  @Service
  public class CoffeeService {
      private static final String CACHE = "springbucks-coffee";
      @Autowired
      private CoffeeRepository coffeeRepository;
      @Autowired
      private RedisTemplate<String, Coffee> redisTemplate;

      public List<Coffee> findAllCoffee() {
          return coffeeRepository.findAll();
      }
      public Optional<Coffee> findOneCoffeeByName(String name) {

          HashOperations<String, Object, Coffee> hashOperations = redisTemplate.opsForHash();

          if (redisTemplate.hasKey(CACHE) && hashOperations.hasKey(CACHE, name)) {

              log.info("Get Coffee {} from redis.", name);

              return (Optional<Coffee>) Optional.of(hashOperations.get(CACHE, name));
              //return Optional.of(hashOperations.get(CACHE, name));
          }

          ExampleMatcher matcher = ExampleMatcher.matching().withMatcher("name", exact().ignoreCase());

          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher)
          );

          log.info("Coffee found:{}", coffee);

          if (coffee.isPresent()) {

              log.info("Put coffee {} to Redis.", name);
              hashOperations.put(CACHE, name, coffee.get());
              redisTemplate.expire(CACHE, 1, TimeUnit.MINUTES);
          }

          return coffee;
      }
  }
  ```

- Application.java

  ```java
  @EnableTransactionManagement
  @Slf4j
  @EnableJpaRepositories
  @SpringBootApplication
  public class RedisDemoApplication implements ApplicationRunner {

      @Autowired
      private CoffeeService coffeeService;

      public static void main(String[] args) {
          SpringApplication.run(RedisDemoApplication.class, args);
      }

      @Bean
      public RedisTemplate<String, Coffee> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
          RedisTemplate<String, Coffee> redisTemplate = new RedisTemplate<>();
          redisTemplate.setConnectionFactory(redisConnectionFactory);
          return redisTemplate;
      }

      @Bean
      public LettuceClientConfigurationBuilderCustomizer customizer() {

          return builder -> builder.readFrom(ReadFrom.MASTER_PREFERRED);
      }

      @Override
      public void run(ApplicationArguments args) throws Exception {

          Optional<Coffee> mocha = coffeeService.findOneCoffeeByName("mocha");

          log.info("Coffee {}.", mocha);

          for (int i = 0; i < 5; i++) {
              mocha = coffeeService.findOneCoffeeByName("mocha");
          }

          log.info("Value form Redis:{}", mocha);
      }
  }

  ```

- 运行结果

  ```yml
  2020-02-16 21:45:09.235  INFO 4521 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
  2020-02-16 21:45:09.479  INFO 4521 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
  2020-02-16 21:45:09.848  INFO 4521 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
  2020-02-16 21:45:10.038  INFO 4521 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.4.10.Final}
  2020-02-16 21:45:10.442  INFO 4521 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.0.Final}
  2020-02-16 21:45:10.630  INFO 4521 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
  2020-02-16 21:45:12.087  INFO 4521 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
  2020-02-16 21:45:12.101  INFO 4521 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
  2020-02-16 21:45:14.004  INFO 4521 --- [           main] c.simon.redisdemo.RedisDemoApplication   : Started RedisDemoApplication in 8.444 seconds (JVM running for 9.99)
  2020-02-16 21:45:14.216  INFO 4521 --- [           main] io.lettuce.core.EpollProvider            : Starting without optional epoll library
  2020-02-16 21:45:14.218  INFO 4521 --- [           main] io.lettuce.core.KqueueProvider           : Starting without optional kqueue library
  Hibernate: 
      select
          coffee0_.id as id1_0_,
          coffee0_.create_time as create_t2_0_,
          coffee0_.update_time as update_t3_0_,
          coffee0_.name as name4_0_,
          coffee0_.price as price5_0_ 
      from
          t_coffee coffee0_ 
      where
          lower(coffee0_.name)=?
  2020-02-16 21:45:14.839  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Coffee found:Optional[Coffee(super=BaseEntity(id=4, createTime=2020-02-16 21:45:09.62, updateTime=2020-02-16 21:45:09.62), name=mocha, price=CNY 30.00)]
  2020-02-16 21:45:14.839  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Put coffee mocha to Redis.
  2020-02-16 21:45:14.857  INFO 4521 --- [           main] c.simon.redisdemo.RedisDemoApplication   : Coffee Optional[Coffee(super=BaseEntity(id=4, createTime=2020-02-16 21:45:09.62, updateTime=2020-02-16 21:45:09.62), name=mocha, price=CNY 30.00)].
  2020-02-16 21:45:14.862  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Get Coffee mocha from redis.
  2020-02-16 21:45:14.875  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Get Coffee mocha from redis.
  2020-02-16 21:45:14.884  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Get Coffee mocha from redis.
  2020-02-16 21:45:14.893  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Get Coffee mocha from redis.
  2020-02-16 21:45:14.899  INFO 4521 --- [           main] c.simon.redisdemo.service.CoffeeService  : Get Coffee mocha from redis.
  2020-02-16 21:45:14.903  INFO 4521 --- [           main] c.simon.redisdemo.RedisDemoApplication   : Value form Redis:Optional[Coffee(super=BaseEntity(id=4, createTime=2020-02-16 21:45:09.62, updateTime=2020-02-16 21:45:09.62), name=mocha, price=CNY 30.00)]
  2020-02-16 21:45:15.036  INFO 4521 --- [extShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
  2020-02-16 21:45:15.043  INFO 4521 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
  2020-02-16 21:45:15.052  INFO 4521 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.

  ```

---

- Redis Repository
  - 实体注解
    - @RedisHash
    - @Id
    - @Indexed

- 处理不同类型数据源的 Repository
  - 如何区分这些 Repository
    - 根据实体的注解
    - 根据继承的接⼝类型
    - 扫描不同的包

---

- BytesToMoneyConverter.java

  ```java
  @ReadingConverter
  public class BytesToMoneyConverter implements Converter<byte[], Money> {
      @Override
      public Money convert(byte[] source) {
          String value = new String(source, StandardCharsets.UTF_8);
          return Money.ofMinor(CurrencyUnit.of("CNY"), Long.parseLong(value));
      }
  }
  ```

- MoneyToBytesConverter.java
  
  ```java
  @WritingConverter
  public class MoneyToBytesConverter implements Converter<Money, byte[]> {
      @Override
      public byte[] convert(Money source) {
          String value = Long.toString(source.getAmountMinorLong());
          return value.getBytes(StandardCharsets.UTF_8);
      }
  }
  ```

- CoffeeCache.java

  ```java
  @RedisHash(value = "springbucks-coffee", timeToLive = 60)
  @Data
  @NoArgsConstructor
  @AllArgsConstructor
  @Builder
  public class CoffeeCache {
      @Id
      private Long id;
      @Indexed
      private String name;
      private Money price;
  }
  ```

- CoffeeCacheRepository.java

  ```java
  public interface CoffeeCacheRepository extends CrudRepository<CoffeeCache, Long> {
      Optional<CoffeeCache> findOneByName(String name);
  }
  ```

- CoffeeService.java

  ```java
  @Slf4j
  @Service
  public class CoffeeService {
      @Autowired
      private CoffeeRepository coffeeRepository;
      @Autowired
      private CoffeeCacheRepository cacheRepository;

      public List<Coffee> findAllCoffee() {
          return coffeeRepository.findAll();
      }

      public Optional<Coffee> findSimpleCoffeeFromCache(String name) {
          Optional<CoffeeCache> cached = cacheRepository.findOneByName(name);
          if (cached.isPresent()) {
              CoffeeCache coffeeCache = cached.get();
              Coffee coffee = Coffee.builder()
                      .name(coffeeCache.getName())
                      .price(coffeeCache.getPrice())
                      .build();
              log.info("Coffee {} found in cache.", coffeeCache);
              return Optional.of(coffee);
          } else {
              Optional<Coffee> raw = findOneCoffee(name);
              raw.ifPresent(c -> {
                  CoffeeCache coffeeCache = CoffeeCache.builder()
                          .id(c.getId())
                          .name(c.getName())
                          .price(c.getPrice())
                          .build();
                  log.info("Save Coffee {} to cache.", coffeeCache);
                  cacheRepository.save(coffeeCache);
              });
              return raw;
          }
      }

      public Optional<Coffee> findOneCoffee(String name) {
          ExampleMatcher matcher = ExampleMatcher.matching()
                  .withMatcher("name", exact().ignoreCase());
          Optional<Coffee> coffee = coffeeRepository.findOne(
                  Example.of(Coffee.builder().name(name).build(), matcher));
          log.info("Coffee Found: {}", coffee);
          return coffee;
      }
  }
  ```

- Application.java

  ```java
  @Slf4j
  @EnableTransactionManagement
  @EnableJpaRepositories
  @SpringBootApplication
  @EnableRedisRepositories
  public class RedisRepositoryDemoApplication implements ApplicationRunner {

      @Autowired
      private CoffeeService coffeeService;

      public static void main(String[] args) {
          SpringApplication.run(RedisRepositoryDemoApplication.class, args);
      }

      @Bean
      public LettuceClientConfigurationBuilderCustomizer customizer(){

          return builer->builer.readFrom(ReadFrom.MASTER_PREFERRED);
      }

      @Bean
      public RedisCustomConversions redisCustomConversions(){
          return new RedisCustomConversions(
                  Arrays.asList(new BytesToMoneyConverter(),new MoneyToBytesConverter())
          );
      }

      @Override
      public void run(ApplicationArguments args) throws Exception {
          Optional<Coffee> c = coffeeService.findSimpleCoffeeFromCache("mocha");
          log.info("Coffee {}", c);

          for (int i = 0; i < 5; i++) {
              c = coffeeService.findSimpleCoffeeFromCache("mocha");
          }

          log.info("Value from Redis: {}", c);
      }
  }
  ```

- 运行结果

  ```yml
  2020-02-16 22:13:12.804  INFO 4742 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Starting...
  2020-02-16 22:13:13.046  INFO 4742 --- [           main] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Start completed.
  2020-02-16 22:13:13.695  INFO 4742 --- [           main] o.hibernate.jpa.internal.util.LogHelper  : HHH000204: Processing PersistenceUnitInfo [name: default]
  2020-02-16 22:13:14.010  INFO 4742 --- [           main] org.hibernate.Version                    : HHH000412: Hibernate Core {5.4.10.Final}
  2020-02-16 22:13:14.696  INFO 4742 --- [           main] o.hibernate.annotations.common.Version   : HCANN000001: Hibernate Commons Annotations {5.1.0.Final}
  2020-02-16 22:13:15.107  INFO 4742 --- [           main] org.hibernate.dialect.Dialect            : HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
  2020-02-16 22:13:17.045  INFO 4742 --- [           main] o.h.e.t.j.p.i.JtaPlatformInitiator       : HHH000490: Using JtaPlatform implementation: [org.hibernate.engine.transaction.jta.platform.internal.NoJtaPlatform]
  2020-02-16 22:13:17.081  INFO 4742 --- [           main] j.LocalContainerEntityManagerFactoryBean : Initialized JPA EntityManagerFactory for persistence unit 'default'
  2020-02-16 22:13:19.753  INFO 4742 --- [           main] c.s.r.RedisRepositoryDemoApplication     : Started RedisRepositoryDemoApplication in 10.733 seconds (JVM running for 12.291)
  2020-02-16 22:13:20.101  INFO 4742 --- [           main] io.lettuce.core.EpollProvider            : Starting without optional epoll library
  2020-02-16 22:13:20.103  INFO 4742 --- [           main] io.lettuce.core.KqueueProvider           : Starting without optional kqueue library
  Hibernate: 
      select
          coffee0_.id as id1_0_,
          coffee0_.create_time as create_t2_0_,
          coffee0_.update_time as update_t3_0_,
          coffee0_.name as name4_0_,
          coffee0_.price as price5_0_ 
      from
          t_coffee coffee0_ 
      where
          lower(coffee0_.name)=?
  2020-02-16 22:13:21.069  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Coffee Found: Optional[Coffee(super=BaseEntity(id=4, createTime=2020-02-16 22:13:13.125, updateTime=2020-02-16 22:13:13.125), name=mocha, price=CNY 30.00)]
  2020-02-16 22:13:21.070  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Save Coffee CoffeeCache(id=4, name=mocha, price=CNY 30.00) to cache.
  2020-02-16 22:13:21.155  INFO 4742 --- [           main] c.s.r.RedisRepositoryDemoApplication     : Coffee Optional[Coffee(super=BaseEntity(id=4, createTime=2020-02-16 22:13:13.125, updateTime=2020-02-16 22:13:13.125), name=mocha, price=CNY 30.00)]
  2020-02-16 22:13:21.212  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Coffee CoffeeCache(id=4, name=mocha, price=CNY 30.00) found in cache.
  2020-02-16 22:13:21.227  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Coffee CoffeeCache(id=4, name=mocha, price=CNY 30.00) found in cache.
  2020-02-16 22:13:21.247  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Coffee CoffeeCache(id=4, name=mocha, price=CNY 30.00) found in cache.
  2020-02-16 22:13:21.260  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Coffee CoffeeCache(id=4, name=mocha, price=CNY 30.00) found in cache.
  2020-02-16 22:13:21.271  INFO 4742 --- [           main] c.s.r.service.CoffeeService              : Coffee CoffeeCache(id=4, name=mocha, price=CNY 30.00) found in cache.
  2020-02-16 22:13:21.271  INFO 4742 --- [           main] c.s.r.RedisRepositoryDemoApplication     : Value from Redis: Optional[Coffee(super=BaseEntity(id=null, createTime=null, updateTime=null), name=mocha, price=CNY 30.00)]
  2020-02-16 22:13:21.412  INFO 4742 --- [extShutdownHook] j.LocalContainerEntityManagerFactoryBean : Closing JPA EntityManagerFactory for persistence unit 'default'
  2020-02-16 22:13:21.415  INFO 4742 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown initiated...
  2020-02-16 22:13:21.420  INFO 4742 --- [extShutdownHook] com.zaxxer.hikari.HikariDataSource       : HikariPool-1 - Shutdown completed.
  ```

---

## SpringBucks 进度⼩结

- 本章小结
  - 了解了 Docker 在本地的基本⽤用法
  - 了解了 Spring Data MongoDB 的基本⽤用法
  - 了解了 Spring Data Redis 的基本⽤用法
  - 了解了 Redis 的⼏几种运⾏行行模式
  - 了解了 Spring 的缓存抽象

- SpringBucks 进度小结
  - 使⽤不同类型的数据库存储咖啡信息
  - 结合 JPA 与 Redis 来优化咖啡信息的存储
  