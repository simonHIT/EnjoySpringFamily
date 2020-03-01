# 服务配置

[TOC]



## 基于 Git 的配置中⼼

### Spring Cloud Conﬁg Server 

- ⽬的 
  - 提供针对外置配置的 HTTP API 
- 依赖 
  - spring-cloud-conﬁg-server 
  - @EnableConfigServer 
  - ⽀持 Git / SVN / Vault / JDBC … 

### 使⽤ Git 作为后端存储 

- 配置 
  - MultipleJGitEnvironmentProperties 
    - spring.cloud.config.server.git.uri 
- 配置⽂件的要素
  - {application}，即客户端的  spring.application.name 
  - {profile}，即客户端的  spring.profiles.active 
  - {label}，配置⽂件的特定标签，默认 master 

- 访问配置内容 
  - HTTP 请求 
    - GET /{application}/{profile}[/{label}] 
    - GET /{application}-{profile}.yml 
    - GET /{label}/{application}-{profile}.yml 
    - GET /{application}-{profile}.properties 
    - GET /{label}/{application}-{profile}.properties 

------

------

### Spring Cloud Conﬁg Client 

- 依赖 
  - spring-cloud-starter-conﬁg 
- 发现配置中⼼ 
  - bootstrap.properties | yml 
  - spring.cloud.config.fail-fast=true 
  - 通过配置 
    - spring.cloud.config.uri=http://localhost:8888 

- 发现配置中⼼ 

  - bootstrap.properties | yml 

  - 通过服务发现 

    ```yaml
    spring.cloud.config.discovery.enabled=true 
    
    spring.cloud.config.discovery.service-id=configserver 
    ```

    

- 配置刷新

  - @RefreshScope 
  - Endpoint - /actuator/refresh 

------

------

## 基于 Zookeeper 的配置中⼼ 

### Spring Cloud Zookeeper Conﬁg 

- 依赖 

  - spring-cloud-starter-zookeeper-conﬁg 
  - 注意 Zookeeper 版本 

- 启⽤ 

  - bootstrap.properties | yml 

    ```yaml
    spring.cloud.zookeeper.connect-string=localhost:2181 
    
    spring.cloud.zookeeper.config.enabled=true 
    ```

### Zookeeper 中的数据怎么存 

- 配置项 

  - /config/应⽤名,profile/key=value 
  - /config/application,profile/key=value 

- 如何定制

  ```yaml
  spring.cloud.zookeeper.config.root=config 
  
  spring.cloud.zookeeper.config.default-context=application 
  
  spring.cloud.zookeeper.config.profile-separator=',' 
  ```

  

------

------

## 深⼊理解 Spring Cloud 的配置抽象 

### Spring Cloud Conﬁg 

- ⽬标 
  - 在分布式系统中，提供外置配置⽀持 
- 实现 
  - 类似于 Spring 应⽤中的  Environment与  PropertySource 
  - 在上下⽂中增加 Spring Cloud Conﬁg 的  PropertySource 

### Spring Cloud Conﬁg 的 PropertySource 

- PropertySource 
  - Spring Cloud Conﬁg Client - CompositePropertySource 
  - Zookeeper - ZookeeperPropertySource 
  - Consul - ConsulPropertySource / ConsulFilesPropertySource 
- PropertySourceLocator 
  - 通过  PropertySourceLocator提供  PropertySource 

### Spring Cloud Conﬁg Server 

- EnvironmentRepository
  - Git / SVN / Vault / JDBC … 
- 功能特性 
  - SSH、代理访问、配置加密 … 
- 配置刷新 
  - /actuator/refresh 
  - Spring Cloud Bus - RefreshRemoteApplicationEvent 

### Spring Cloud Conﬁg Zookeeper 

- ZookeeperConﬁgBootstrapConﬁguration 
  - 注册  ZookeeperPropertySourceLocator 
    - 提供  ZookeeperPropertySource 
- ZookeeperConﬁgAutoConﬁguration  
  - 注册  ConfigWatcher 

### 配置的组合顺序 

- 以 yml 为例 
  - 应⽤名-proﬁle.yml 
  - 应⽤名.yml 
  - application-proﬁle.yml 
  - application.yml 

## 基于 Consul 的配置中⼼ 

### Spring Cloud Consul Conﬁg 

- 依赖 

  - spring-cloud-starter-consul-conﬁg 

- 启⽤ 

  - bootstrap.properties | yml 

    ```yaml
    spring.cloud.consul.host=localhost 
    
    spring.cloud.consul.port=8500 
    
    spring.cloud.consul.config.enabled=true 
    ```

### Consul 中的数据怎么存 

- 配置项 
  - spring.cloud.consul.config.format= KEY_VALUE | YAML | PROPERTIES | FILES 
  - /config/应⽤名,profile/data 
  - /config/application,profile/data 

- 如何定制 

  ```yaml
  spring.cloud.consul.config.data-key=data 
  
  spring.cloud.consul.config.root=config 
  
  spring.cloud.consul.config.default-context=application 
  
  spring.cloud.consul.config.profile-separator=',' 
  ```

### 配置项变更 

- ⾃动刷新配置 

  ```yaml
  spring.cloud.consul.config.watch.enabled=true 
  
  spring.cloud.consul.config.watch.delay=1000 
  ```

  

- 实现原理

  - 单线程  ThreadPoolTaskScheduler 
  - ConsulConfigAutoConfiguration.CONFIG_WATCH_TASK_SCHEDULER_NAME 

------

------

## 基于 Nacos 的配置中⼼ 

### Spring Cloud Alibaba Nacos Conﬁg 

- 依赖 
  - spring-cloud-starter-alibaba-nacos-conﬁg 
  - spring-cloud-alibaba-dependencies:0.9.0 
    - 注意 Spring Cloud 与 Spring Boot 的对应版本 

- 启⽤ 

  - bootstrap.properties | yml 

  ```yaml
  spring.cloud.nacos.config.server-addr=127.0.0.1:8848 
  
  spring.cloud.nacos.config.enabled=true 
  ```

### Nacos 中的数据怎么存 

- 配置项 
  - dataId 
    - ${prefix}-${spring.profile.active}.${file-extension} 
    - spring.cloud.nacos.config.prefix 
    - spring.cloud.nacos.config.file-extension 
  - spring.cloud.nacos.config.group 

------

------

## SpringBucks 实战项⽬进度⼩结 

### 本章⼩结 

- ⼏种不同的配置中⼼ 
  - Spring Cloud Conﬁg Server 
    - Git / SVN / RDBMS / Vault 
  - Zookeeper 
  - Consul 
  - Nacos 

### SpringBucks 进度⼩结 

- waiter-service 
  - 增加了订单⾦额与折扣 
  - 增加了 Waiter 名称 
  - 使⽤了不同的配置中⼼ 
    - Spring Cloud Conﬁg Client 
    - 使⽤ Zookeeper 
    - 使⽤ Consul 
    - 使⽤ Nacos 

### 携程 Apollo 

- 官⽅地址 
  - https://github.com/ctripcorp/apollo 
- 特性 
  - 统⼀管理不同环境、不同集群的配置 
  - 配置修改实时⽣效（热发布） 
  - 版本发布管理 
  - 灰度发布 
  - 权限管理、发布审核、操作审计 
  - 客户端配置信息监控 
  - 提供开放平台API 