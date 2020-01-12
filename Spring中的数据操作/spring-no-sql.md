# NoSQL 实践

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

---

```java

```

---

- 通过 Docker 启动 Redis

  - 官方指引
    - https://hub.docker.com/_/redis
  - 获取镜像
    - docker pull redis
  - 启动 Redis
    - docker run --name redis -d -p 6379:6379 redis

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

  - @EnableCaching
    - @Cacheable
    - @CacheEvict
    - @CachePut
    - @Caching
    - @CacheConfig

---

```java
```

---

- 通过 Spring Boot 配置 Redis 缓存

- pom依赖

```xml
```

- application.properties

```yml
```

- Application.java

```java
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
  