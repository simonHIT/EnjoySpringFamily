# JDBC必知必会

## 如何配置数据源 

1. Spring Boot 的配置演示 
   - 引⼊对应数据库驱动——H2 
   - 引⼊ JDBC 依赖——spring-boot-starter-jdbc 
   - 获取 DataSource Bean，打印信息 
   - 也可通过 /acturator/beans 查看 Bean 
  
    ![spring boot 配置演示](images/spring-jdbc-01.png)

    - 运行代码

    ```java
    @SpringBootApplication
    @Slf4j
    public class DatasourceDemoApplication implements CommandLineRunner {

	    @Autowired
	    private DataSource dataSource;

	    @Autowired
	    private JdbcTemplate jdbcTemplate;

	    public static void main(String[] args) {
		    SpringApplication.run(DatasourceDemoApplication.class, args);
	    }

	    @Override
	    public void run(String... args) throws Exception {

		    log.info(dataSource.toString());
		    Connection connection=dataSource.getConnection();
		    log.info(connection.toString());
		    connection.close();

	    }
    }

    ```

   - 启动项目我们可以看到spring boot默认为我们配置好了数据源
   - 运行结果
    ![运行结果](images/spring-jdbc-03.png)

2. 直接配置所需的Bean 

   - 数据源相关 
      - DataSource（根据选择的连接池实现决定） 
   - 事务相关（可选） 
      - PlatformTransactionManager（DataSourceTransactionManager） 
      - TransactionTemplate 
   - 操作相关（可选） 
     - JdbcTemplate 

- 我们通过一个没有springboot的项目来演示jdbc操作
- 

3. Spring Boot 帮我们做了哪些配置 
   - DataSourceAutoConﬁguration 
     - 配置 DataSource 
   - DataSourceTransactionManagerAutoConﬁguration 
     - 配置 DataSourceTransactionManager 
   - JdbcTemplateAutoConﬁguration 
     - 配置 JdbcTemplate 
   - 注意：符合条件时才进⾏配置，如果项目中配置了数据源等信息，springboot不会去再配置这些信息；如果没有相应的配置，sringboot则会去自动配置

4. 数据源相关配置属性 

   - 通⽤ 
     - spring.datasource.url=jdbc:mysql://localhost/test 
     - spring.datasource.username=dbuser 
     - spring.datasource.password=dbpass 
     - spring.datasource.driver-class-name=com.mysql.jdbc.Driver（可选） 
   - 初始化内嵌数据库 
     - spring.datasource.initialization-mode=embedded|always|never 
     - spring.datasource.schema与spring.datasource.data确定初始化SQL⽂件 
     - spring.datasource.platform=hsqldb | h2 | oracle | mysql | postgresql（与前者对应）

- 下面我们通过一个简单的配置实例来演示
- application.properties的配置
  
  ```yml
  management.endpoints.web.exposure.include=*
  spring.output.ansi.enabled=ALWAYS

  spring.datasource.url=jdbc:h2:mem:testdb
  spring.datasource.username=sa
  spring.datasource.password=
  spring.datasource.hikari.maximumPoolSize=5
  spring.datasource.hikari.minimumIdle=5
  spring.datasource.hikari.idleTimeout=600000
  spring.datasource.hikari.connectionTimeout=30000
  spring.datasource.hikari.maxLifetime=1800000
  ```

- 实例代码
  ```java
    @SpringBootApplication
    @Slf4j
    public class DatasourceDemoApplication implements CommandLineRunner {

	    @Autowired
	    private DataSource dataSource;

	    @Autowired
	    private JdbcTemplate jdbcTemplate;

	    public static void main(String[] args) {
		    SpringApplication.run(DatasourceDemoApplication.class, args);
	    }

	    @Override
	    public void run(String... args) throws Exception {

		    showConnection();
		    showData();

	    }

	    public void showConnection() throws SQLException {
		    log.info(dataSource.toString());
		    Connection connection=dataSource.getConnection();
		    log.info(connection.toString());
		    connection.close();
	    }


	    public void showData(){

		    jdbcTemplate.queryForList("SELECT * FROM FOO")
				    .forEach(row->log.info(row.toString()));
	    }
    }
  ```
- 运行结果
  ![运行结果](images/spring-jdbc-03.png)
  
## 多数据源的配置注意事项

- 注意事项
  - 不同数据源的配置要分开 
  
  - 关注每次使⽤的数据源 
  
    - 有多个DataSource时系统如何判断，需要及时判断当前数据库操作使用的是哪个数据源
    - 对应的设施（事务、ORM等）如何选择DataSource ，当前事务、设施（mybatis、h2)等使用哪个数据源，需要格外小心

- 解决措施

- ⼿⼯配置两组 DataSource 及相关内容，与Spring Boot协同⼯作（两组datasource只能两者选其一）
   
  - 配置@Primary类型的Bean 
  - 排除Spring Boot的⾃动配置 
    - DataSourceAutoConﬁguration 
    - DataSourceTransactionManagerAutoConﬁguration 
    - JdbcTemplateAutoConﬁguration 

---
- 参考示例
  - application.properties

  ```yml
  management.endpoints.web.exposure.include=*
  spring.output.ansi.enabled=ALWAYS

  foo.datasource.url=jdbc:h2:mem:foo
  foo.datasource.username=sa
  foo.datasource.password=

  bar.datasource.url=jdbc:h2:mem:bar
  bar.datasource.username=sa
  bar.datasource.password=
  ```
  - 主启动类
  
  ```java
  @SpringBootApplication(exclude = {
        DataSourceAutoConfiguration.class,
        DataSourceTransactionManagerAutoConfiguration.class,
        JdbcTemplateAutoConfiguration.class
  })
  @Slf4j
  public class MultiDatasourceDemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(MultiDatasourceDemoApplication.class, args);
    }

    @Bean
    @ConfigurationProperties("foo.datasource")
    public DataSourceProperties fooDataSourceProperties(){
        return new DataSourceProperties();
    }

    @Bean
    public DataSource fooDataSource(){
        DataSourceProperties dataSourceProperties= fooDataSourceProperties();
        log.info("foo datasource:{}",dataSourceProperties.getUrl());
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

    @Bean
    public PlatformTransactionManager fooTxManager(DataSource fooDataSource){

        return new DataSourceTransactionManager(fooDataSource);
    }

    @Bean
    @ConfigurationProperties("bar.datasource")
    public DataSourceProperties barDataSourceProperties(){
        return new DataSourceProperties();
    }

    @Bean
    public DataSource barDataSource(){
        DataSourceProperties dataSourceProperties=barDataSourceProperties();
        log.info("bar datasource:{}",dataSourceProperties.getUrl());
        return dataSourceProperties.initializeDataSourceBuilder().build();
    }

    @Bean
    public PlatformTransactionManager barTxManager(DataSource barDataSource){
        return new DataSourceTransactionManager(barDataSource);
    }

  }

  ```

- 运行结果
  
  - 可以看到springboot为我们生成了两组datasource，调用哪一组取决于我们的业务
  
---

