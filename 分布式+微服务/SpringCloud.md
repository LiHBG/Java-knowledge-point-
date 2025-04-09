# SpringCloud 基础

## MybatisPlus

### 使用方式

- 引入依赖

  ```xml
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus</artifactId>
      <version>3.5.11</version>
  </dependency>
  <!-- 
  该依赖是mybatis的升级，
  所以mybatis内的所有东西它都能进行使用 
  -->
  <dependency>
      <groupId>com.baomidou</groupId>
      <artifactId>mybatis-plus-boot-starter</artifactId>
      <version>3.5.3.1</version>
  </dependency>
  ```

- 继承BaseMapper<T>

  ```java
  @Mapper
  public interface ClazzMapper extends BaseMapper<Clazz> {}
  ```

- 直接注入IOC容器中管理的原对象就能使用父类中的方法

  ```Java
  @Override
  public void updateClazzById(Clazz clazz) {
      clazz.setUpdateTime(LocalDateTime.now());
      clazzMapper.updateById(clazz);//updateById 就是baseMapper内  的方法
  }
  ```

### 常见注解

MybatisPlus通过扫描实体类，并基于反射获取实体类信息作为数据库表信息。

- **@TableName** ：用来指定表名

  - 如果数据库中表名的下划线转驼峰命名不一致就需要使用该注解

- **@TableId** :           用来指定表中的主键字段信息

  - **value** ：如果数据库中表的主键字段不是id，就需要去指定
  - **type**   ：可以指定主键的属性（例如：IdType.AUTO-主键自增）

- **@TableField** ：   用来指定表中普通字段信息

  - 如果字段是布尔类型的，并且在数据库表中的字段是以 **is_** 开头的，在MP中通过反射会将is去掉，所以要使用

      - ```Java
        @TableField("is_delete")
        private Boolean isDelete;
        ```

  - 成员变量名与数据库中的关键字有冲突
  
      - ```Java
          @TableField("`order`")
          private Integer order;
          ```
  
  - 成员变量不是数据库字段
  
      - ```Java
          @TableField(exist = false)
          private String address;
          ```

### 常用配置

```properties
#myabtis-plus
#开启驼峰命名
mybatis-plus.configuration.map-underscore-to-camel-case=true
#实体类包名
mybatis-plus.type-aliases-package=com.msl.pojo
#扫描mapper接口包名
mybatis-plus.mapper-locations="classpath*:/mapper/**/*.xml"
#是否开启二级缓存
mybatis-plus.configuration.cache-enabled=false
#指定全局id属性的生成策略，如果实体类中注解进行了配置，那么就用注解的配置
#mybatis-plus.global-config.db-config.id-type=auto
#更新策略：只更新非空字段
mybatis-plus.global-config.db-config.update-strategy=not_null
```

### 核心功能

#### 条件构造器

MP支持各种复杂的 **Where** 条件，可以满足日常开发的所有需求。

- QueryWrapper 设置的是where的条件

  - ```Java
    // 查询表中，username包含‘o’，balance大于1000的全部数据。
    QueryWrapper<User> wrapper = new QueryWrapper<User>()
        						 // 指定需要查询的字段
        							.select("id","username","info"."balance")
        						 // 模糊匹配
        							.like("username","o")
        						 // 大于等于
        							.ge("balance",1000);
    List<User> users = userMapper.selectList(wrapper);
    // 更新用户名为Lus的余额为3000
    User user = new User(null,"","",3000);
    QueryWrapper<User> wrapper = new QueryWrapper<User>()
        					// 指定用户名
        						.eq("username","Lus");
    userMapper.update(user,wrapper);
    ```

- UpdateWrapper 特殊的更新

  - ```Java
    //更新id为1,2,3,4用户的余额扣除500
    List<Integer> ids = List.of(1,2,3,4);
    UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
        						// 手写SQL
        						.setSql("balance = balance - 500")
        						// id列表
        						.in("id",ids);
    userMapper.update(null,wrapper);
    ```

- LambdaQueryWrapper 不再是硬编码

  - ```Java
    LamdbaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
        						 // 指定需要查询的字段
        				.select(User::getId,User::getInfo,User::getBalance)
        						 // 模糊匹配
        							.like(User::getUsername,"o")
        						 // 大于等于
        							.ge(User::getBalance,1000);
    List<User> users = userMapper.selectList(wrapper);
    ```

#### 自定义SQL

利用Wrapper来构建复杂的Where条件，然后自定义SQL语句中剩下的部分。

1. 基于Wrapper构建where条件

   - ```Java
     List<Integer> ids = List.of(1,2,3,4);
     int amount = 500;
     LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
         						// id列表
         						.in(User::getId,ids);
     //调用自定义方法
     userMapper.updateBalanceById(wrapper,amount);
     ```

2. 在mapper方法参数中用Param注解声明wrapper变量名称,必须是ew

   - ```java 
     void updateBalanceById(@Param("ew") LambdaQueryWrapper<User> wrapper,@Param("amount") int amount);
     ```

3. 自定义SQL。并使用Wrapper条件

   ```xml
   <update id="updateBalanceById">
       UPDATE user SET balance = balance - #{amount} ${ew.customSqlSegment}
   </update>
   ```



#### Service接口

MP提供了很多基础类型的增删改查操作的方法。但是如果要进行使用的话，首先接口要进行继承，其次接口的实现类也需要继承。

```Java
// 接口
public interface BlogPostService  extends IService<BlogPost> {}
// 实现类
public class BlogPostServiceImpl extends ServiceImpl<BlogPostMapper, BlogPost> implements BlogPostService {}
```

- 使用IService的LambdaQueryWrapper条件查询

  - ```Java
    @Override
    public List<BlogPost> queryPosts(BlogQueryParam queryParam) {
        return lambdaQuery()
                .like(queryParam.getTitle() != null, BlogPost::getTitle, queryParam.getTitle())
                .list();
    }
    ```

- 使用IService的LambdaUpdateWrapper条件更新

  - ```Java
    return lambdaUpdate()
        	.eq(User::getId,id)
        	.set(User::getBalance,remainBalance)
        	.set(remainBalance == 0,User::getStuts,2)
        	.update();
    ```

- 使用Iservice的批量插入数据

  - ```properties
    #MySQL中开启批量SQL重写
    spring.datasource.url=jdbc:mysql://localhost:3306/blog?useUnicode=true&characterEncoding=utf-8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
    ```

### 扩展功能

#### 代码生成器

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250407162911624.png" alt="image-20250407162911624" style="zoom: 67%;" />

#### DB静态工具

解决循环依赖的情况，再进行当前类的时候，如果查询当前表之后还需要查询其他表的数据。就可以直接使用这个静态方法。

#### 逻辑删除

提供了逻辑删除的功能，无需改变方然调用的方式，而是底层帮助我们修改CRUD语句。

```properties
#逻辑删除字段名
mybatis-plus.global-config.db-config.logic-delete-field=isDelete
#逻辑删除值
mybatis-plus.global-config.db-config.logic-delete-value=1
#逻辑未删除值
mybatis-plus.global-config.db-config.logic-not-delete-value=0
```

#### 枚举处理器

可以帮我们**把枚举类型与数据库类型自动转换**。

```properties
#开启枚举处理器
mybatis-plus.configuration.default-enum-type-handler=com.baomidou.mybatisplus.core.handlers.MybatisEnumTypeHandler
```

```Java
@Getter
public enum IsDeletStatus {
    DELETED(1, "已删除"),
    NOT_DELETED(0, "未删除");

    @EnumValue//告诉MP怎么去进行取值
    @JsonValue//告诉SpringMVC将什么值返回给前端
    private final Integer code;
    private final String message;

    IsDeletStatus(Integer code, String message) {
        this.code = code;
        this.message = message;
    }
}
```

```Java
@Data
public class Comment {
    @TableId(type = IdType.AUTO)
    private Integer id;
    private String content;
    private Integer postId;
    private Integer authorId;
    private LocalDateTime createTime;
    @TableField("`is_delete`")
    private IsDeletStatus isDelete;
}
```

#### JSON处理器

如果数据库表中有一个JSON格式的字段，后端使用的是String类型进行接收的，但是如果需要拿取其中的属性进行业务处理就会很麻烦，所以就需要将JSON格式转换为Java中的实体类，MP提供了这样的处理器，但是没有全局配置，具体操作如下：

1. 将需要进行JSON处理的字段加上注解

   ```Java
   @TableField(typeHandler = JacjsonTypeHandler.class)
   private UserInfo userInfo;//需要定义UserInfo的实体
   ```

2. 因为加上了这样的注解，就会出现实体类嵌套的情况，字段映射就会变得麻烦就需要开启MP的自动映射

   ```Java
   @TableName(value="user",autoResultMap = true)
   public class User{}
   ```

### 插件功能

#### 分页插件

插件功能并不是直接就能使用的，需要先进行配置才能够进行使用。

```Java
@Bean
public MybatisPlusInterceptor mybatisPlusInterceptor() {
    MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
    interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));
    return interceptor;
}
```

```Java
@Test
void testPageQuery() {
    // 1.分页查询，new Page()的两个参数分别是：页码、每页大小
    Page<User> p = userService.page(new Page<>(2, 2));
    // 2.总条数
    System.out.println("total = " + p.getTotal());
    // 3.总页数
    System.out.println("pages = " + p.getPages());
    // 4.数据
    List<User> records = p.getRecords();  
    records.forEach(System.out::printl0n);
}
// 也可以增加排序的条件
page.addOrder(new OrderItem("balance",true));//前者是排序的字段，后者是排序的顺序，true是顺序，false是倒序
```

## 微服务

### 准备工作

- 使用Docker部署MySQL容器

  - ```bash
    docker run -d  \
    	--name mysql  \
    	-p 3306:3306  \
    	-e TZ=Asia/Shanghai \
    	-e MYSQL_ROOT_PASSWORD=root \
    	-v /root/mysql/data:/var/lib/mysql \
    	-v /root/mysql/conf:/etc/mysql/conf.d \
    	-v /root/mysql/init:/docker-entrypoint-initdb.d \
    	--network hm-net \
    mysql
    ```

### 认识微服务

- 单体架构：将业务的所有功能集中在一个项目中开发，打成一个包进行部署。	
  - 优点：架构简单，部署成本低
  - 缺点：团队协作成本高、系统发布效率低、系统可用性差
  - 适用于开发功能相对于比较简单的，规模比较小的项目。
- 微服务架构：就是把单体架构中的功能模块拆分为多个独立项目。
  - 拆分的要求：粒度小、团队自治、服务自治。

### 微服务拆分

- 服务拆分的原则
  - 什么时候进行拆分？
    - 创业型项目：先采用单体架构，快速开发，快速试错。随着规模扩大，逐渐进行拆分。
    - 确定的大型项目：资金充足，目标明确，可以直接选择微服务架构，避免后续拆分的麻烦。
  - 怎么拆？
    - 高内聚：每个微服务的职责要尽量的单一，包含的业务相互关联度高、完整度高。
    - 低耦合：每个微服务的功能要相对的独立，尽量减少对其他微服务的依赖。
  - 拆分的方式？
    - 横向拆分：抽取公共服务，提高复用性
    - 纵向拆分：按照业务模块来拆分
  - 拆分后的工程架构？
    - 独立Project：适用于大型的项目，不方便管理
    - Maven聚合：适用于中小型项目，方便管理
- 拆分案例
- <img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250409010050100.png" alt="image-20250409010050100" style="zoom: 50%;" />



将一个单体的项目按照业务逻辑层面进行拆分，也就是纵向的进行拆分。

### 远程调用

Spring为我们提供了一个RestTemplate工具，可以方便的实现HTTP请求的发送。

- 注入RestTemplate到Spring容器

  - ```Java
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
    ```

- 发起远程调用（这种方式了解即可，不需要掌握）

  - ```Java
    ResponseEntity<List<ItemDTO>> entity = restTemplate.exchange(
            "http://localhost:8081/items?ids={ids}",
            HttpMethod.GET,
            null,
            new ParameterizedTypeReference<List<ItemDTO>>() {
            },
            Map.of("ids", CollUtil.join(itemIds, ","))
    );
    if (!entity.getStatusCode().is2xxSuccessful()) return;
    List<ItemDTO> items = entity.getBody();
    
    if (CollUtils.isEmpty(items)) {
        return;
    }
    ```

### 服务治理

#### 注册中心原理

注册中心会详细的记录部署起来的服务所需要被记录的所有内容。服务的调用者会通过注册中心获取服务的调用者的访问路径。服务的调用者会时刻的向注册中心发送心跳回应，如果服务宕机了那么注册中心会将该宕机的服务提供者从注册中心剔除，并向服务的调用者推送最新的信息。

#### Nacos注册中心

- 配置文件

  - ```properties
    PREFER_HOST_MODE=hostname
    MODE=standalone
    SPRING_DATASOURCE_PLATFORM=mysql
    MYSQL_SERVICE_HOST=192.168.220.132
    MYSQL_SERVICE_DB_NAME=nacos
    MYSQL_SERVICE_PORT=3306
    MYSQL_SERVICE_USER=root
    MYSQL_SERVICE_PASSWORD=root
    MYSQL_SERVICE_DB_PARAM=characterEncoding=utf8&connectTimeout=1000&socketTimeout=3000&autoReconnect=true&useSSL=false&allowPublicKeyRetrieval=true&serverTimezone=Asia/Shanghai
    ```

    

- Docker进行部署nacos(需要和配置文件夹在同一目录)

  - ```bash
    docker run -d \
    --env-file ./nacos/custom.env \
    -p 8848:8848 \
    -p 9848:9848 \
    -p 9849:9849 \
    --restart=always \
    nacos/nacos-server:v2.1.0-slim
    ```

- 访问nacos http://xxx.xxx.xxx.xxx:8848/nacos 在首次访问的时候需要输入用户和密码 默认都是 nacos

##### 服务的注册

1. 引入nacos discovery依赖

   ```xml
   <!--nacos注册中心-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

2. 配置nacos地址

   ```yaml
   spring:
     application:
       name: item-service #服务名称
     cloud:
       nacos:
         server-addr: ${hm.nacos.host}:${hm.nacos.port} #nacos地址
   #nacos配置
   hm:
     nacos:
       host: 192.168.220.132
       port: 8848
   ```

3. 配置完成之后，启动项目就会自动的完成服务的注册。

##### 服务的发现

消费者需要连接nacos以拉取和订阅服务，因此服务发现的前两步与服务注册一致，后面再加上服务调用即可。

1. 引入nacos discovery依赖

   ```xml
   <!--nacos注册中心-->
   <dependency>
       <groupId>com.alibaba.cloud</groupId>
       <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
   </dependency>
   ```

   

2. 配置nacos地址

   ```yaml
   spring:
     application:
       name: cart-service
     cloud:
       nacos:
         server-addr: ${hm.nacos.host}:${hm.nacos.port}
   ```

3. 服务发现

   ```Java
   //根据服务名称，获取服务实例
   List<ServiceInstance> instances = discoveryClient.getInstances("item-service");
   if (CollUtil.isEmpty(instances)) return;
   // 2.随机获取一个实例,负载均衡
   ServiceInstance serviceInstance = instances.get(RandomUtil.randomInt(instances.size()));
   // 3.获取实例的IP和端口
   URI uri = serviceInstance.getUri();
   ResponseEntity<List<ItemDTO>> entity = restTemplate.exchange(
           uri + "/items?ids={ids}",
           HttpMethod.GET,
           null,
           new ParameterizedTypeReference<List<ItemDTO>>() {
           },
           Map.of("ids", CollUtil.join(itemIds, ","))
   );
   ```

### OpenFegin

OpenFegin是一个声明式的http客户端，作用是基于SpringMvc的常见注解，帮我们优雅的实现请求的发送。

#### 快速入门

1. 引入依赖

   ```xml
   <!--OpenFeign-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-openfeign</artifactId>
   </dependency>
   <!--负载均衡-->
   <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-loadbalancer</artifactId>
   </dependency>
   ```

   

2. 通过@EnableFeginClients注解，启用OpenFegin功能

   ```Java
   @EnableFeignClients // 启用OpenFegin功能
   @SpringBootApplication
   public class CartServiceApplication {}
   ```

3. 编写FeginClient

   ```Java
   @FeignClient(value = "item-service")
   public interface ItemClient {
       @GetMapping("/items")
       List<ItemDTO> queryItemByIds(@RequestParam("ids") List<Long> ids);
   }
   ```

4. 使用FeginClient，实现远程调用

   ```Java
   // 使用feign 来进行远程调用
   List<ItemDTO> items = itemClient.queryItemByIds(itemIds);
   ```

#### 连接池

OpenFegin对Http请求做了伪装，但是底层发起请求依赖于其他的框架。可以对框架进行选择，如下：

- HttpURLConnection：默认实现，不支持连接池

- Apache HttpClient：支持连接池

- OKHttp：支持连接池

  - 引入依赖

    ```xml
    <!--okhttp-->
    <dependency>
        <groupId>io.github.openfeign</groupId>
        <artifactId>feign-okhttp</artifactId>
    </dependency>
    ```

    

  - 开启连接池功能

    ```yaml
    feign:
      okhttp:
        enabled: true # 启用okhttp,开启okhttp连接池支持
    ```

#### 最佳实践

1. 方案一：将原来的模块再次划分为小的模块。DTO：放置dto的实体类，client：方式需要对外暴露的接口，biz：放置原本需要执行的全部逻辑代码。
2. 方案二：使用的是maven聚合的，那么就可以直接在父工程下创建一个新的模块，里面的内容全部都是其他摸扩需要对外暴露的接口，不包含其他的内容。

#### 日志输出

只会在FeginClient所在包的日志级别为DEBUG时，才会输出日志。日志级别有四种：

- **NOME** ： 不记录任何日志信息，这是默认值。
- **BASIC** : 仅记录请求的方法，URL以及响应状态码和执行时间。
- **HEADERS** : 在BASIC的基础上，额外记录了请求和响应的头信息。
- **FULL** ： 记录所有请求和响应的明细，包括头信息、请求体、元数据。

要自定义日志级别需要声明一个类型为Logger.Level的bean，在其中定义日志级别。

```Java
public class DefaultFeignConfig{
    @Bean
    public Logger.Level feignLogLeevel(){
        return Logger.Level.FULL;
    }
}
```

此时bean并未生效，想要配置某个FeignClient的日志，可以在@FeignClient注解中说明。

```Java
@FeignClient(value = "item-service"，configuration = DefaultFeignConfig.clss)
```

如果需要进行全局的配置。

```Java
@EnableFeignClients(DefaultConfiguration = DefaultFeignConfig.class)
```

### 网关

就是网络的关口，负责请求的路由、转发、身份校验。

#### 网关路由

SpringCloudGateWay

1. 创建新的模块，引入依赖，编写启动类

   ```xml
   <dependencies>
       <!--common-->
       <dependency>
           <groupId>com.heima</groupId>
           <artifactId>hm-common</artifactId>
           <version>1.0.0</version>
       </dependency>
       <!--网关-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-gateway</artifactId>
       </dependency>
       <!--nacos discovery-->
       <dependency>
           <groupId>com.alibaba.cloud</groupId>
           <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
       </dependency>
       <!--负载均衡-->
       <dependency>
           <groupId>org.springframework.cloud</groupId>
           <artifactId>spring-cloud-starter-loadbalancer</artifactId>
       </dependency>
   </dependencies>
   <build>
       <finalName>${project.artifactId}</finalName>
       <plugins>
           <plugin>
               <groupId>org.springframework.boot</groupId>
               <artifactId>spring-boot-maven-plugin</artifactId>
           </plugin>
       </plugins>
   </build>
   ```

   

2. 编写配置类

   ```yaml
   spring:
     application:
       name: gateway
     cloud:
       nacos:
         server-addr: 192.168.220.132:8848
       gateway:
         routes:
           - id: item # 路由规则id，自定义，唯一
             uri: lb://item-service # 路由的目标服务，lb代表负载均衡，会从注册中心拉取服务列表
             predicates: # 路由断言，判断当前请求是否符合当前规则，符合则路由到目标服务
               - Path=/items/**,/search/** # 这里是以请求路径作为判断规则
           - id: trade
             uri: lb://trade-service
             predicates:
               - Path=/orders/**
           - id: pay
             uri: lb://pay-service
             predicates:
               - Path=/pay-orders/**
           - id: user
             uri: lb://user-service
             predicates:
               - Path=/users/**
           - id: cart
             uri: lb://cart-service
             predicates:
               - Path=/carts/**
   server:
     port: 8080
   ```

##### 路由属性

网关路由对应的Java类是RounteDefinitiom，其中常见的属性有：

- id：路由的唯一标识
- url：路由目标地址
- predicates：路由断言，判断请求是否符合当前路由
- filters：路由过滤器，对请求或响应做特殊处理

#### 网关登录校验

##### 网关请求处理流程

- **客户端** 发送请求到网关。
- 发送到网关之后被 **路由映射器** 进行处理。
  - **HandlerMapping** 的默认的实现是  **RoutePredicateHandlerMapping** 也就是 **路由断言** 。
  - **HandlerMapping** 根据请求找到匹配的路由并存入上下文，然后把请求交给 **WebHandler** 处理。
- **路由映射器** 处理之后会把请求交给 **WebHandler** 处理，也就是 **请求处理器**。
  - **WebHandler** 的默认实现是 **FilterWebHandler** 就是一个过滤器处理器，会加载网关中配置的多个过滤器，放入过滤器集合中并进行排序，形成**过滤器链** 并依次执行这些过滤器。
    - 过滤器的内部分别包含两个部分的逻辑：pre 和 post
      - **pre** :指的是请求经**过滤器**到**微服务**之前执行的逻辑。
      - **post**:指的是请求经**微服务**返回结果到**过滤器**之后执行的逻辑。
  - **NettyRoutingFilter** 是过滤器链的最后一个过滤器，它负责的就是将**过滤器链** 处理过后的请求发送到 **微服务** ，当**微服务**返回结果后存入上下文。

##### 实现网关登录校验

需要设置一个**过滤器**并且自己设置的过滤器需要在**NettyRouteingFilter**之前去执行，需要执行的逻辑部分就是请求发送到微服务之前需要执行的**pre**请求。

- **自定义过滤器** ： 网关过滤器有两种，分别是：

  - **GatewayFilter** : 路由过滤器，作用于任意指定的路由；默认不生效，要配置到路由后生效。

    - 自定义的**并不是直接实现GatewayFilter**，而是创建一个工厂。

      - 细节：

        - 类的名称：**UserPassGatewayFilterFactory** 其中 **GatewayFilterFactory** 是固定的类名称后缀，方便配置的使用

        - 类的名称：**UserPassGatewayFilterFactory** 其中 **UserPass** 前缀就被作为配置类中过滤器名称

          - ```yaml
                gateway:
                  routes:
                    - id: item
                      uri: lb://item-service
                      predicates:
                        - Path=/items/**,/search/**
                  default-filters: 
                    - UserPass # 添加自定义过滤器
            ```

    ```Java
    @Component 
    public class UserPassGatewayFilterFactory extends AbstractGatewayFilterFactory<Object> {
    
        @Override
        public GatewayFilter apply(Object config) {
            return new GatewayFilter() {
                @Override
                public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                    // TODO 过滤器逻辑
                    return chain.filter(exchange);
                }
            };
        }
        
        @Override
        public GatewayFilter apply(Object config) {
            return new OrderedGatewayFilter(//该类可以控制自定义过滤器的流程
                    new GatewayFilter() {
                        @Override
                        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
                            // TODO 过滤器逻辑
                            return chain.filter(exchange);
                        }
                    },1
            );
        }
    }
    
    
    ```

  - **GlobalFilter** : 全局过滤器，作用范围是所有路由；声明之后自动生效。

    ```Java
    @Component
    public class MyGlobalFilter implements GlobalFilter, Ordered {
        /**
         * 过滤器
         * @param exchange 里面包含的内容有请求头、响应头、请求体、响应体等
         * @param chain 过滤器链，可以继续执行下一个过滤器
         * @return
         */
        @Override
        public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
            //TODO 过滤器逻辑
            return chain.filter(exchange);
        }
    
        /**
         * 控制过滤器执行顺序，越小越先执行
         * @return
         */
        @Override
        public int getOrder() {
            return 0;
        }
    }
    ```

- 实现网关登录校验

  ```Java
  /**
   * @author: lihonbin
   * @date: 2025/04/10
   */
  @Component
  @RequiredArgsConstructor
  public class AuthGlobalFilter implements GlobalFilter, Ordered {
  
      private final AuthProperties authProperties;
  
      private final JwtTool jwtTool;
  
      private final AntPathMatcher antPathMatcher = new AntPathMatcher();
  
      @Override
      public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
          //1.获取请求request
          ServerHttpRequest request = exchange.getRequest();
          //2.判断是否要进行登录拦截
          if (isExcludePaths(request.getPath().toString())) {
              //2.1.不需要，直接放行
              return chain.filter(exchange);
          }
          //3.获取token
          String token = null;
          List<String> headers = request.getHeaders().get("authorization");
          if (headers != null && !headers.isEmpty()) {
              token = headers.get(0);
          }
          //4.检验并解析token
          Long userId = null;
          try {
              userId = jwtTool.parseToken(token);
          } catch (RuntimeException e) {
              // 5.校验失败，返回401
              ServerHttpResponse response = exchange.getResponse();
              response.setStatusCode(HttpStatus.UNAUTHORIZED);
              return response.setComplete();
          }
          //TODO 5.传递用户信息
          //6.放行
          return chain.filter(exchange);
      }
  
      private boolean isExcludePaths(String path) {
          List<String> excludePaths = authProperties.getExcludePaths();
          for (String excludePath : excludePaths) {
              if (antPathMatcher.match(excludePath, path)) {
                  return true;
              }
          }
          return false;
      }
  
      @Override
      public int getOrder() {
          return 0;
      }
  }
  ```







# SpringColud 总结

## MybatisPlus 总结

**1.是如何获取实现CRUD的数据库表信息的？**

回答：1.默认以类名的驼峰转下划线作为表名；2.默认名为id的字段作为表中的主键；3.默认把变量名驼峰转下划线作为表中的列名

**2.常用的注解有哪些？**

回答：1.@TableName 用来指定所映射的类在数据库中的表名(在类名驼峰转下划线与数据库中表名不一致时使用)；2.@TableId 用来指定某一个字段是表名的主键，如果不是字段名不是id，那么就需要通过value属性来指定表中主键的字段名。还需要指定在表中主键的约束和策略；@TableField用来指定普通的字段，如果表中的字段名是数据库中的关键字就需要使用该注解，如果实体类中有表中以外的字段也需要使用。

**3.IdType常见的类型有哪些？**

回答：1.AUTO自增；2.ASSIGN_ID由MP通过指定的算法生成；3.INPUT通过用户自己指定。

**4.使用@TableField的常见场景是？**

回答：1.如果实体类中有表中不存在的字段；2.表中的字段在数据库中是关键字；3.表中字段是 **is_** 开头。

