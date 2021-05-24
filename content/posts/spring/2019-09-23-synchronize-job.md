---
author: ymkNK
categories: Spring
date: "2019-09-23T17:12:49Z"
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/5.jpg
subtitle: 从零开始搭建一个同步信息的项目
tag: Spring
title: 从零开始搭建AD信息同步的项目
---
## 一 前言
最近独自完成了一个同步信息的项目，主要工作内容是从第三方的服务端将信息同步到公司的二方AD域当中。本文主要记录本项目的主要实现过程。

## 二 项目设计
这是一个并不复杂的项目，因此项目主要涉及到两大部分：
1. 从第三方服务器端获取到信息数据
2. 将信息数据进行处理，能够存入到公司的二方AD域当中

## 三 第三方数据获取
### 1.主要技术
本项目主要获取信息的方式，是通过FeignClient来获取的

### 2.主要实现
#### a.最单纯的feign方式实现
##### 添加pom依赖
```
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-okhttp</artifactId>
			<version>10.1.0</version>
		</dependency>
		<dependency>
			<groupId>io.github.openfeign</groupId>
			<artifactId>feign-jackson</artifactId>
			<version>10.1.0</version>
		</dependency>
```
##### 定义接收返回值所用到的model对象
这个地方的model定义，需要仔细阅读第三方服务端的api文档，然后根据上面的具体信息来进行定义  
eg:
```
@Data
public class XxxApiResponse {
    private String code;
    private String message;
    private String timestamp;
    private String errcode;
    private String errmsg;
    private String detailMsg;
}

```
这里的变量名没有按照驼峰的方式进行定义，是因为官方文档返回值当中的字段key值就是这样非驼峰的方式，我们需要保证名字的一致性。

##### 定义Feign
```
public interface XxxClient {

    @RequestLine("GET /v3/xxx/details?param1={param1}")
    XxxDetailsResponse getXxxDetails(@Param("param1") String param1);

}
```
注意这种RequestLine的方式和GetMapping和PostMapping的方式有所区别
##### 定义Feign的Factory
```
public class XxxClientFactory {
    private final static String URL = "https://api.xxx.com";

    public XxxClient createXxxClient() {
        ObjectMapper mapper = new ObjectMapper()
                .setSerializationInclusion(JsonInclude.Include.NON_NULL)
                .setPropertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE)
                .configure(SerializationFeature.INDENT_OUTPUT, true)
                .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        return Feign.builder()
                .logger(new Slf4jLogger())
                .client(new OkHttpClient())
                .encoder(new FormEncoder(new JacksonEncoder(mapper)))
                .decoder(new JacksonDecoder(mapper))
                .logLevel(Logger.Level.FULL)
                .target(XxxClient.class, URL);
    }

    private XxxClientFactory() {

    }

    public static XxxClientFactory newInstance() {
        return new XxxClientFactory();
    }
}

```
#### b.常用FeignClient实现
##### 添加依赖

##### Application中增加注解,允许指定包的feignClient有效
```
@EnableFeignClients(basePackages = {"com.shuidihuzhu.devops.xrxs.client", "com.shuidihuzhu.devops.xrxs.feign"})
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

### 3.创建service
创建获取第三方数据的service，所有三方数据的获取，按照内容分模块的进行封装实现，将通过上面的client获取到的数据进行处理然后分别进行返回。

## 四 将数据同步到AD域

### 1.添加依赖和配置
#### a.依赖
```
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-data-ldap</artifactId>
		</dependency>
```
#### b.配置
LdapConfiguration.Java
```
@Configuration
@EnableLdapRepositories
public class LdapConfiguration {
    private LdapTemplate ldapTemplate;

    @Value("${spring.ldap.urls}")
    private String LDAP_URL;
    @Value("${spring.ldap.username}")
    private String USERNAME;
    @Value("${spring.ldap.password}")
    private String PASSWORD;
    @Value("${spring.ldap.base}")
    private String BASE;

    @Bean
    public LdapContextSource contextSource() {
        LdapContextSource contextSource = new LdapContextSource();
        Map<String, Object> config = new HashMap<>();
        contextSource.setUrl(LDAP_URL);
        contextSource.setBase(BASE);
        contextSource.setUserDn(USERNAME);
        contextSource.setPassword(PASSWORD);
        config.put("java.naming.ldap.attributes.binary", "objectGUID");
        contextSource.setPooled(true);
        contextSource.setBaseEnvironmentProperties(config);
        return contextSource;
    }

    @Bean
    public LdapTemplate ldapTemplate() {
        if (null == ldapTemplate){
            ldapTemplate = new LdapTemplate(contextSource());
        }

        ldapTemplate.setIgnorePartialResultException(true);
        return ldapTemplate;
    }
}

```
Application.Java
```
@SpringBootApplication
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
@EnableScheduling
@EnableFeignClients(basePackages = {"com.shuidihuzhu.devops.xrxs.client", "com.shuidihuzhu.devops.xrxs.feign"})
@EnableAspectJAutoProxy(proxyTargetClass = true)
@EnableCircuitBreaker
@EnableLdapRepositories
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

    @Bean
    LdapTemplate getLdapTemplate(ContextSource contextSource) {
        LdapTemplate ldapTemplate = new LdapTemplate(contextSource);
        ldapTemplate.setIgnorePartialResultException(true);
        return ldapTemplate;
    }
}

```
注意`setIgnorePartialResultException(true)`这个方法是必须要设置好的，不然对接AD的时候，会报出莫名其妙的错误
### 2.定义model和DAO层
以人员为例,Ad中的user对应的objectClasses有四个类，base设置就是对应的AD域下的专门存放这些user的域
```
@Data
@Entry(objectClasses = {"top", "person", "organizationalPerson", "user"}, base = "ou=XxxUser")
public final class XxxUser {

    @Id
    @JsonIgnore
    private Name id;


    @Attribute(name = "mail")
    private String mail;

    @Attribute(name = "mobile")
    private String mobile;

    @Attribute(name = "sn")
    private String sn;

    @Attribute(name = "cn")
    @DnAttribute(value = "cn", index = 1)
    private String cn;


    @Attribute(name = "displayName")
    private String displayName;

    @Attribute(name = "ou")
    @DnAttribute(value = "ou", index = 1)
    private String ou;
}
```
定义对应的DAO,在这个DAO接口中，可以使用JPA的函数命名来进行条件查询，但是还是做不了一些复杂的查询，如果需要复杂的查询依然还是需要在自定义的接口XxxUserExtension中进行定义，然后进行实现。
```
@Repository
public interface XxxUserRepo extends LdapRepository<XxxUser>, XxxUserExtension {
 
    List<XxxUser> findByOu(String ou);

}
```

## 3.定义Service层
根据业务的需求，将获取到的第三方的信息，经过处理之后，将Response的有效内容解析到对应的AD的model中，通过对应的DAO层类，存入到AD中

## 五 创建定时任务
根据cron或者fixedRate(每隔一段时间执行)来设置定时任务
```
@Slf4j
@Component
@EnableScheduling
public class XxxToAdJob {
    /**
     * 每20分钟同步一次
     */
//  @Scheduled(cron = "0 */20 * * * ?") 
    @Scheduled(fixedRate = 1000 * 60 * 20)
    public void doExecuteJob() {
    	// 同步逻辑
    }
```
## 六 注意事项
1. 本文只是将本项目最简单的框架思路展现了出来，在具体实施的过程中，会有很多很多坑
2. 对接第三方数据源的时候，需要注意请求的一些频率，比如某些数据源要求，每分钟的请求调用不能超过200次
3. 对接AD的时候，需要it部门的人将对应功能模块所需要的端口都打开，不然会有问题
4. 如果需要ldaps访问，需要导出AD的证书然后使用java的keytool制作证书，然后在启动的时候增加参数（但是目前都没有成功）
5. ldap常见错误码 [官方链接](https://docs.servicenow.com/bundle/jakarta-platform-administration/page/administer/reference-pages/reference/r_LDAPErrorCodes.html?title=LDAP_Error_Codes)
6. mac电脑可以使用Parallels Client这个软件远程登录AD的服务器进行查看（AD是搭建在一个windows的集群上的） 
7. AD自定义属性的创建 [相关文章](https://www.cnblogs.com/llcs/archive/2018/04/26/8948315.html) 注意x500的中国区对象id必须是2.16.156开头的