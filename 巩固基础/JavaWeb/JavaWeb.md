# JavaWeb

## Web基础



#### HTTP协议：

**规定了浏览器和服务器之间数据传输的规则。（都是纯文本）**

##### 请求协议：

请求标头：

```http
GET /hello?name=lhb HTTP/1.1 
第一行描述的信息有：发送请求的方式（get），发送请求的路径以及携带的参数，http协议的版本（请求行）
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7（表示浏览器能够接收的资源类型）
Accept-Encoding: gzip, deflate, br, zstd（浏览器可以支持的压缩类型）
Accept-Language: zh-CN,zh;q=0.9（浏览器偏好的语言，服务器可以根据此返回不同语言的网页）
Connection: keep-alive
Host: localhost:8080（代表的是请求的主机名）
Sec-Fetch-Dest: document
Sec-Fetch-Mode: navigate
Sec-Fetch-Site: none
Sec-Fetch-User: ?1
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.0.0 Safari/537.36 QuarkPC/2.3.2.266（浏览器的版本）
sec-ch-ua: "Not?A_Brand";v="99", "Chromium";v="130"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
其余的行被称为请求头，get方式的请求是没有请求体的（get请求在浏览器中是有限制的；POST请求，请求参数在请求体中，所以POST请求大小是没有限制的）。
```

请求数据的获取：

Web服务器对http协议的请求数据进行了解析，并进行了封装（HttpServletRequest），在调用Controller方法的时候传递给了该方法。所以我们开发的时候不需要直接对协议进行处理。

```Java
@RestController
public class RequstController {
    @RequestMapping("/request")
    public String getRquestInfo(HttpServletRequest request) {
        String methodName = request.getMethod();// 获取请求方法名(请求的方式)
        System.out.println("请求的方式:" + methodName);
        String url = request.getRequestURL().toString();// 获取请求的url(请求的路径)
        System.out.println("请求的路径:" + url);
        String requestURI = request.getRequestURI();// 获取请求的uri(请求的资源路径)
        System.out.println("请求的uri:" + requestURI);
        String protocol = request.getProtocol();// 获取请求的协议(http/https)
        System.out.println("请求的协议:" + protocol);
        String parameterName = request.getParameter("name");// 获取请求的参数
        System.out.println("请求的参数: " + parameterName);
        String headerAccept = request.getHeader("Accept");//获取请求头中，指定名字的数据
        System.out.println(" Accept :" + headerAccept);
        return "OK";
    }
}
```



##### 响应协议：

响应标头：

```http
HTTP/1.1 200
第一行描述的信息为：http协议版本，以及响应状态码（响应行）
Content-Type: text/html;charset=UTF-8（响应主体的数据类型，在请求体中类似（POST））
Content-Length: 8（响应主体的大小（字节），请求体中类似（POST））
Date: Thu, 13 Mar 2025 01:18:33 GMT
Keep-Alive: timeout=60
Connection: keep-alive
其余部分为响应头
响应体：存储应该响应的数据
```

响应状态码：

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250313101034191.png" alt="image-20250313101034191" style="zoom:50%;" />	

响应数据设置：
Web服务器对http协议的响应数据进行了封装（httpServletResponse），在调用Controller方法的时候传递给了该方法。所以我们开发的时候不需要直接对协议进行处理。（响应状态码以及响应头一般不需要手动的进行设置）

```Java
@RestController
public class ResponseController {

    @RequestMapping("/response")
    public void setResponse(HttpServletResponse response) throws IOException {
        response.setStatus(HttpServletResponse.SC_OK);//设置响应状态码
        response.setHeader("Content-Type", "text/html");//设置响应头(如果还需要进行设置其他内容，可以多次调用)
        response.getWriter().write("<h1>hello response<h1>");//设置响应体(响应体的设置需要用到IO流)
    }

    @RequestMapping("/response2")
    public ResponseEntity<String> setResponse2() {
        return ResponseEntity.status(HttpServletResponse.SC_OK)//设置响应状态码
                .header("Content-Type", "text/html")//设置响应头
                .body("<h1>hello response<h1>");//设置响应体
    }
}
```

**特点：**

**1.基于TCP协议：面向连接，安全**

**2.基于请求响应模型：一次请求对应的就是一次响应**

**3.HTTP协议是无状态的协议：对于事物的处理是没有记忆能力的。每一次的请求与响应都是独立的。**

​													**优点：请求的速度快；缺点：多次请求之间不能共享数据**

#### 案例一：

```Java
@RestController
public class ListController {
    @RequestMapping("/list")
    public String getList() {
        // 因为在编译之后src和resources会在同一目录下
        InputStream is = this.getClass()//解释：获取当前类的字节码文件
                .getClassLoader()//解释：获取类加载器
                .getResourceAsStream("user.txt");//解释：获取资源文件
        ArrayList<String> list = IoUtil.readLines(is, StandardCharsets.UTF_8, new ArrayList<>());//解释：读取文件内容
        List<User> userList = list.stream().map(line -> {
            String[] splits = line.split(",");//解释：将字符串按逗号分割成数组
            Integer id = Integer.parseInt(splits[0]);//解释：将数组的第一元素转换为int类型，并赋值给id
            String userName = splits[1];//解释：将数组的第二元素赋值给userName
            String password = splits[2];//解释：将数组的第三元素赋值给password
            String name = splits[3];//解释：将数组的第四元素赋值给name
            Integer age = Integer.parseInt(splits[4]);//解释：将数组的第五元素转换为int类型，并赋值给age
            LocalDateTime updateTime = LocalDateTime.parse(splits[5], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));//解释：将数组的第六元素转换为LocalDateTime类型，并赋值给updateTime
            return new User(id, userName, password, name, age, updateTime);
        }).collect(Collectors.toList());//解释：将list中的元素转换为User对象，并收集到一个新的集合中
        return JSONUtil.toJsonStr(userList, JSONConfig.create().setDateFormat("yyyy-MM-dd HH:mm:ss"));//解释：将userList转换为json字符串，并返回
    }
}

@RestController
public class ListController {
    @RequestMapping("/list")
    public List<User> getList() {
        InputStream is = this.getClass()//解释：获取当前类的字节码文件
                .getClassLoader()//解释：获取类加载器
                .getResourceAsStream("user.txt");//解释：获取资源文件
        ArrayList<String> list = IoUtil.readLines(is, StandardCharsets.UTF_8, new ArrayList<>());//解释：读取文件内容
        List<User> userList = list.stream().map(line -> {
            String[] splits = line.split(",");//解释：将字符串按逗号分割成数组
            Integer id = Integer.parseInt(splits[0]);//解释：将数组的第一元素转换为int类型，并赋值给id
            String userName = splits[1];//解释：将数组的第二元素赋值给userName
            String password = splits[2];//解释：将数组的第三元素赋值给password
            String name = splits[3];//解释：将数组的第四元素赋值给name
            Integer age = Integer.parseInt(splits[4]);//解释：将数组的第五元素转换为int类型，并赋值给age
            LocalDateTime updateTime = LocalDateTime.parse(splits[5], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));//解释：将数组的第六元素转换为LocalDateTime类型，并赋值给updateTime
            return new User(id, userName, password, name, age, updateTime);
        }).collect(Collectors.toList());//解释：将list中的元素转换为User对象，并收集到一个新的集合中
       // return JSONUtil.toJsonStr(userList, JSONConfig.create().setDateFormat("yyyy-MM-dd HH:mm:ss"));//解释：将userList转换为json字符串，并返回
        return userList;
    }
}
```

@RestController

```Java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody（将controller返回的数据作为响应体返回，如果返回的是集合或者对象，那么会先将其转化为json再返回）
```

#### 分层解耦：

三层架构：

**Controller:控制层，接收前端发送的请求，对请求进行处理，并返回数据**

**Service：业务逻辑层，处理具体业务的逻辑**

**Dao：数据访问层（Data Access Object）-持久层，数据访问的操作，包括数据的增、删、改、查**

```Java
@RestController
public class ListController {
    @Autowired
    private UserService userService;
    @RequestMapping("/list")
    public List<User> getList() {
        List<User> userList = userService.findAll();
        return userList;
    }
}
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;
    @Override
    public List<User> findAll() {
        List<String> list = userDao.findAll();//解释：调用UserDao中的findAll方法，获取所有用户的信息
        List<User> userList = list.stream().map(line -> {
            String[] splits = line.split(",");//解释：将字符串按逗号分割成数组
            Integer id = Integer.parseInt(splits[0]);//解释：将数组的第一元素转换为int类型，并赋值给id
            String userName = splits[1];//解释：将数组的第二元素赋值给userName
            String password = splits[2];//解释：将数组的第三元素赋值给password
            String name = splits[3];//解释：将数组的第四元素赋值给name
            Integer age = Integer.parseInt(splits[4]);//解释：将数组的第五元素转换为int类型，并赋值给age
            LocalDateTime updateTime = LocalDateTime.parse(splits[5], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));//解释：将数组的第六元素转换为LocalDateTime类型，并赋值给updateTime
            return new User(id, userName, password, name, age, updateTime);
        }).toList();//解释：将list中的元素转换为User对象，并收集到一个新的集合中
        return userList;
    }
}
@Repository
public class UserDaoImpl implements UserDao {
    @Override
    public List<String> findAll() {
        InputStream is = this.getClass()//解释：获取当前类的字节码文件
                .getClassLoader()//解释：获取类加载器
                .getResourceAsStream("user.txt");//解释：获取资源文件
        ArrayList<String> list = IoUtil.readLines(is, StandardCharsets.UTF_8, new ArrayList<>());
        return list;
    }
}
```

**控制反转：简称IOC。对象的创建控制权由程序本身转移到外部（IOC容器），这种思想称为控制反转。**

**依赖注入：简称DI。容器为应用程序提供运行时所依赖的资源，称为依赖注入。**

**Bean对象：IOC容器中创建、管理的对象，称为Bean对象。**

```Java
//@Component：加在实现类上，表示将这个类交给IOC容器进行管理
//@Autowired: 在需要使用由IOC容器进行管理的Bean对象的时候，使用该注解可以在IOC容器中找到需要使用的Bean对象，并将该对象的值赋值，也就是完成依赖注入
```

##### IOC详解：

```Java
@Component：声明Bean的基础注解，可以将声明的Bean交给IOC容器管理
@Controller：@Component衍生的注解，用于标注在控制层（在spingboot集成的web开发中，控制层只能使用Controller）
@Srevice：@Component衍生的注解，用于标注在业务层
@Repository：@Component衍生的注解，用于标注在数据访问层（由于Mybatis的整合，用的比较少）
将Bean交给IOC容器管理的时候，如果不指定Bean在IOC容器中的名字，那么默认就是类的首字母小写
也可以指定Bean的名字@Srevice（value="bean的名字"）

以上注解想要其生效的话，还需要被组件扫描注解@ComponentScan进行扫描。
该注解没有显示的配置，但是包含在了启动类注解的@SpringBootApplication中，扫描的范围是扫描当前类（启动类）所在的包以及其子包
```

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250313155250030.png" alt="image-20250313155250030"  />

```Java
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```

##### DI详解：

```Java
//基于@AuotoWired进行依赖注入有三种方式：
//1.属性注入：
    @Autowired
    private UserDao userDao;
//2.构造函数注入：
    private final UserService userService2;
    @Autowired//在类中只有当前一个构造函数的时候，当前注解可以省略
    public ListController(UserService userService2) {
        this.userService2 = userService2;
    }
//3.setter注入：
    private UserService getUserService3;
    @Autowired
    public void setUserService(UserService userService){
        this.getUserService3 = userService;
    }
```

**1.属性注入：优点：代码简单，方便快捷开发；缺点：隐藏了类与类之间的关系，并且可能会破会类的封装性；底层依赖于反射进行属性赋值。**

**2.构造函数注入：优点：能够清晰的看到类的依赖关系，提高了代码的安全性。缺点：代码繁琐，参数过多会造成代码臃肿。注意：只有一个构造函数的时候，@AutoWired注解可以进行省略**

**3.setter方法注入：优点：保持类的封装性，依赖关系清晰。缺点：需要编写setter方法，代码量增加。**

```Java
//@AutoWired：默认是按照类型进行注入的，如果存在一个类型有多个bean，将会报错
//解决方案：
//方案一：在需要优先注入的类上加上注解@Primary
@Primary//如果当前类型有多个bean，优先注入当前类代表的bean
@Service//解释：将UserServiceImpl类注册为服务，并实现UserService接口
public class UserServiceImpl implements UserService {
    @Autowired
    private UserDao userDao;
    @Override
    public List<User> findAll() {
        List<String> list = userDao.findAll();//解释：调用UserDao中的findAll方法，获取所有用户的信息
        List<User> userList = list.stream().map(line -> {
            String[] splits = line.split(",");//解释：将字符串按逗号分割成数组
            Integer id = Integer.parseInt(splits[0]);//解释：将数组的第一元素转换为int类型，并赋值给id
            String userName = splits[1];//解释：将数组的第二元素赋值给userName
            String password = splits[2];//解释：将数组的第三元素赋值给password
            String name = splits[3];//解释：将数组的第四元素赋值给name
            Integer age = Integer.parseInt(splits[4]);//解释：将数组的第五元素转换为int类型，并赋值给age
            LocalDateTime updateTime = LocalDateTime.parse(splits[5], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));//解释：将数组的第六元素转换为LocalDateTime类型，并赋值给updateTime
            return new User(id, userName, password, name, age, updateTime);
        }).toList();//解释：将list中的元素转换为User对象，并收集到一个新的集合中
        return userList;
    }
}
//方案二：使用@Qualifier("指定注入bean的名字")
@Service//解释：将UserServiceImpl类注册为服务，并实现UserService接口
public class UserServiceImpl implements UserService {
    @Autowired
    @Qualifier("userDaoImpl")
    private UserDao userDao;
    @Override
    public List<User> findAll() {
        List<String> list = userDao.findAll();//解释：调用UserDao中的findAll方法，获取所有用户的信息
        List<User> userList = list.stream().map(line -> {
            String[] splits = line.split(",");//解释：将字符串按逗号分割成数组
            Integer id = Integer.parseInt(splits[0]);//解释：将数组的第一元素转换为int类型，并赋值给id
            String userName = splits[1];//解释：将数组的第二元素赋值给userName
            String password = splits[2];//解释：将数组的第三元素赋值给password
            String name = splits[3];//解释：将数组的第四元素赋值给name
            Integer age = Integer.parseInt(splits[4]);//解释：将数组的第五元素转换为int类型，并赋值给age
            LocalDateTime updateTime = LocalDateTime.parse(splits[5], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));//解释：将数组的第六元素转换为LocalDateTime类型，并赋值给updateTime
            return new User(id, userName, password, name, age, updateTime);
        }).toList();//解释：将list中的元素转换为User对象，并收集到一个新的集合中
        return userList;
    }
}
//方案三：使用Java提供注解@Resource（name="需要注入bean的名字"）-->默认按照名称进行注入
@Service//解释：将UserServiceImpl类注册为服务，并实现UserService接口
public class UserServiceImpl implements UserService {
    @Resource(name = "userDaoImpl")
    private UserDao userDao;
    @Override
    public List<User> findAll() {
        List<String> list = userDao.findAll();//解释：调用UserDao中的findAll方法，获取所有用户的信息
        List<User> userList = list.stream().map(line -> {
            String[] splits = line.split(",");//解释：将字符串按逗号分割成数组
            Integer id = Integer.parseInt(splits[0]);//解释：将数组的第一元素转换为int类型，并赋值给id
            String userName = splits[1];//解释：将数组的第二元素赋值给userName
            String password = splits[2];//解释：将数组的第三元素赋值给password
            String name = splits[3];//解释：将数组的第四元素赋值给name
            Integer age = Integer.parseInt(splits[4]);//解释：将数组的第五元素转换为int类型，并赋值给age
            LocalDateTime updateTime = LocalDateTime.parse(splits[5], DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss"));//解释：将数组的第六元素转换为LocalDateTime类型，并赋值给updateTime
            return new User(id, userName, password, name, age, updateTime);
        }).toList();//解释：将list中的元素转换为User对象，并收集到一个新的集合中
        return userList;
    }
}
```

#### MySQL：

MySQL的使用量在降低，因为在8之后的版本已经开始需要收费。

MySQL初始化命令：

```SQL
mysqld --initialize-insecure
```

注册MySQL服务：

```SQL
mysqld -install
```

连接MySQL数据库：

```SQL
mysql -u用户名 -p密码 [-h数据库服务器IP地址 -P端口号]
```

**关系型数据库：建立在关系型模型基础上，由多张相互连接的二维表组成数据库。**

##### SQL：

一种操纵关系型数据库的编程语言，定义操作所有关系型数据库的统一标准。

**DDL:**数据定义语言，用来定义数据库对象（数据库，表，字段）

```SQL
--基础的操纵数据库的指令：
--查询所有的数据库
show databases;--注意不能将databases写成database
--查询当前使用的数据库
select database();--注意不能将select记成show
--使用/切换数据库
use 数据库名;
--创建数据库
create database [if not exists] 数据库名 [default charset utf8mb4]
--create database itcast;
--create database if not exists itcast;创建数据库，如果不存在当前名字的数据库
--删除数据库
drop database [if exists] 数据库名;
--drop database if exists itcast;如果存在当前名字的数据库，那么再进行删除。

-- 在上述的SQL语句中，可以将database换成schema。

--create table 表名(
--	字段1 字段类型 [约束] [comment 字段1注释],
--	......
--	字段n 字段类型 [约束] [comment 字段n注释]
--)[comment 表注释];

-- 创建表,没有对字段进行约束。
create table userInfo(
    id       int comment 'ID,唯一标识',
    userName varchar(30) comment '用户使用的账户名称',
    name     varchar(10) comment '用户的真实姓名',
    age      int comment '用户的年龄',
    gender   char(1) comment '用户的性别'
)comment '用户信息表';

--约束的概念：作用于表中的字段上，规定存储时需要遵循的规则，及限制存储在表中的数据
--约束的目的：保证数据库中数据的正确性，完整性和有效性
--常见约束：
--1.非空约束（限制该字段存储的值不能为null）关键字：not null
--2.唯一约束（保证字段存储的值不能重复）关键字：unique
--3.主键约束（是一行数据的唯一标识，非空且唯一）关键字：primary key
--4.默认约束（保存字段值时如果没有指定存储的字段值，那么就采用提前规定的默认值）关键字：default
--5.外键约束（让两张表的数据建立连接，保证数据的一致性和完整性）关键字：foreign key

--创建表，对字段进行约束
--自增关键字：auto_increment
create table userInfo2
(
    id       int primary key auto_increment comment 'ID，唯一标识,自增',
    userName varchar(30) not null unique comment '用户的账户名称',
    name     varchar(10) not null comment '用户的姓名',
    age      int comment '年龄',
    gender   char(1) default '男' comment '用户的性别'
) comment '用户信息表';

--数据结构：数值类型，字符串类型，日期类型
--数值类型:tinyint(1字节) < smallint(2字节) < mediumint(3字节) < int(4字节) < bigint(8字节) < float(4字节) < double(8字节)  decimal(小数值精度高) 默认是有符号的取值范围（signed） 如果需要无符号，那么需要关键字unsigned描述
--float(5,2),double(5,2),decimal(5,2):5表示整个数字长度，2表示小数位个数
--选取规则：在满足业务需求的情况下，需要选取尽可能占位小的数据类型
--字符串类型：
--char（定长字符串） < varchar (变长字符串) 
--char(10)&varchar(10):前者固定占用10个空间，不能存储1个字符还是多个字符都是10个空间；后者最多占用10个空间，存储不同个字符那么也会动态的占用与字符数相同的空间。char的性能比varchar高，varchar比char更节约空间
--tinyblob tinytext blob text mediumblob mediumtext longblob longtext
--日期类型：date(3字节) time(3字节) year(1字节) datatime(8字节) timestamp(4字节)

--查询数据库中所有表
show tables;
--查询指定表结构
desc tableName;
--查询指定表的创建语句
show create table tableName;
--添加字段
alter table tableName add 字段名 类型 [comment注释][约束];
--修改字段类型
alter table tableName modify 字段名 新类型长度;
--修改字段名与字段类型
alter table tableName change 旧字段名 新字段名 类型 [comment][约束];
--删除字段
alter table tableName drop column 字段名;
--修改表名
alter table tableName rename to 新表名;
--删除表
drop table [if exists] 表名;

-- 查询所有表结构
show tables ;
-- 查询指定表结构
desc emp;
-- 查询指定表的建表语句
show create table emp;
-- 添加字段
alter table userInfo2 add create_time datetime comment '创建时间';
--修改字段类型
alter table userInfo2 modify age tinyint ;
--修改字段名与字段类型
alter table userInfo2 change gender sex tinyint comment '性别；1 男； 2 女';
--删除字段
alter table userInfo2 drop column create_time;
--修改表名
alter table userInfo2 rename to userInfo3;
--删除表
drop table if exists userInfo3 ;
```

**案例一：**

```SQL
--1.页面展示信息为：姓名，性别，头像，所属部门，职位，入职日期，最后操作时间
--2.新增用户信息：用户名，姓名，性别，手机号，职位，薪资，入职日期，头像 
create table emp
(
    id          int unsigned primary key auto_increment comment 'ID',
    user_Name   varchar(20)      not null unique comment '用户名',
    password    varchar(32)      not null default '123456' comment '密码； 默认：123456',
    name        varchar(10)      not null comment '姓名',
    gender      tinyint unsigned not null comment '性别,1 男，2 女',
    phone       char(11)         not null comment '手机号',
    job         tinyint unsigned comment '1 班主任；2 讲师；3 学工主管；4 教研主管；5 咨询师',
    salary      int unsigned comment '薪资',
    entry_data  date comment '入职日期',
    image       varchar(255) comment '头像',
    create_time datetime comment '创建时间',
    update_time datetime comment '最后修改时间'
) comment '员工信息表';
```

**DML:**数据操作语言，用来对数据库表中的数据进行操作（增，删，改）

```sql
--添加insert
--1.指定字段添加数据
insert into 表名(字段名1，字段名2...) values (值1，值2...);
--2.全部字段添加数据
insert into 表名 values(值1,值2，值3...);
--3.批量添加(全部字段)
insert into 表名 values(值1，值2，值3...),(值1，值2，值3..),...
--4.批量添加(指定字段)
insert into tableName(column1，column2) values(值1，值2)，（值1，值2）...

-- 1.为表emp中user_name ,password ,name ,gender ,phone插入数据
insert into emp(user_Name, password, name, gender, phone) values ('lintianxuan', 'ltx123456', '林天玄', 1, '18781220617');
== 2.为表emp所有字段插入值
insert into emp(id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time,
                update_time) values (2,'liyunjun', '12345678', '李云君', 1, '13212319701', 1, 5000, '2025-03-17',
                                    'asdadsadsa', now(), now());
-- 3.批量向emp表的user_name、name、gender字段插入数据
insert into emp(user_name, name, gender) values('zhangsanfen','张三丰',1),('yaoyuegonzhu','邀月宫主',2),('zhujiuzhen','朱九真',2);

--修改update
--1.修改数据
update tableName set 字段名1=值1，字段名2=值2，字段名3=值3,...[where 条件];
-- 1.将emp表中id为1的员工，姓名name字段更新为'张三'
update emp set name='张三' where id = 1;
-- 2.将emp表的所有员工入职日期更新为'2025-03-20'
update emp set entry_data='2025-03-20',update_time=now();

--删除delete
delete from trableName  [where 条件];
-- 1.删除emp表中id为1的员工
delete from emp where id=1;
-- 2.删除emp表中所有员工
delete from emp;
```

**DQL：**数据查询语言，用来查询数据库中表的记录

```SQL
--查询select
select 字段列表 from 表名列表 where 条件列表 group by 分组字段列表 having 分组后条件列表 order by 排序字段列表 limit 分页参数
--基本查询
select...from...
	--1.查询多个字段（建议不要使用通配符 *）
	--2.去除重复记录(distinct)
	select distinct 字段名 from 表名
-- 1.查询指定字段 name，entry_date并返回，查询所有员工的 name, entry_date，并起别名(姓名、入职日期)
select name '姓名',entry_data '入职日期' from emp;
-- 2.查询返回所有字段
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp;	
-- 3.查询已有的员工关联了哪几种职位(不要重复)
select distinct job from emp;
--条件查询(where)
select...from...where..
	--1.比较运算符 
	between...and...(在某个范围之内，包含最小最大值),in(...)(in之后的列表值，多选一),like(模糊匹配),is null(为																						null)
	--2.逻辑运算符
	and(多条件成立),or(多条件任意成立一个),not(不是)
# 1.查询 姓名 为 '邀月宫主' 的员工
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where name='邀月宫主';
# 2.查询 薪资小于等于 5000 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where salary <= 5000;
# 3.查询 没有分配职位 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where job is null;
# 4.查询 有职位 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where job is not null;
# 5.查询 密码不等于 '123456' 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where password <> '123456';
# 6.查询 入职日期 在  '2000-01-01' (包含)  到  '2025-03-20'(包含) 之间的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where entry_data between '2000-01-01' and '2025-03-20';
# 7.查询 入职时间 在 '2000-01-01' (包含) 到 '2025-03-20'(包含) 之间 且 性别为女 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where gender = 0 and entry_data between '2000-01-01' and '2025-03-20';
# 8.查询 职位是 2 (讲师), 3 (学工主管), 4 (教研主管) 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where job in (2,3,4);
# 9.查询 姓名 为两个字的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where name like '__';
# 10.查询 姓 '张' 的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where name like '张%';
# 11.查询 姓名中包含 '二'  的员工信息
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp where name like '%二%';
	--3.聚合函数:聚合函数会忽略空值，null值不做统计
	count(统计数量),max(最大值),min(最小值),avg(平均值),sum(求和)
# 1.统计该企业员工数量,在计算全局的时候优先推荐count(*),或者count(常量)	
select count(job) '员工数量' from emp ;
# 2.统计该企业员工的平均薪资
select avg(salary) '平均薪资' from emp;
# 3.统计该企业员工的最低、最高薪资
select min(salary) from emp;
select max(salary) from emp;
# 4.统计该企业每月要给员工发放的薪资总额(薪资之和)
select sum(salary) from emp;
--分组查询(group by)：分组查询之后，字段不能随意乱写，一般是分组字段名 + 聚合函数
select...from...[where 条件]...group by...[having 分组后过滤条件]
	--执行顺序：where > 聚合函数 > having。where之后不能用聚合函数进行判断，但是having之后可以
-- 1.根据性别分组 , 统计男性和女性员工的数量
select gender,count(*) '员工数量' from emp group by gender;
-- 2.查询入职时间在 '2025-03-20' (包含) 以前的员工 , 并对结果根据职位分组 , 获取员工数量大于等于2的职位
select job, count(*) 'number'
from emp
where entry_data <= '2025-03-20' -- 分组前条件
group by job -- 分组依据，按照job进行分组
having number >= 2; --分组之后的条件
--排序查询(order by)
select...from...[where 条件][group by 分组字段名 having 分组后过滤条件]order by 排序字段 排序规则;
	--排序方式：asc（升序），desc（降序）默认是升序的可以不写。
	--可以进行多字段进行排序，第一个字段值相同的时候才会执行第二字段的排序。
-- 1.根据入职时间, 对员工进行升序排序
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp order by entry_data;
-- 2.根据入职时间，对员工进行降序排序
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp order by entry_data desc ;
-- 3.根据入职时间对公司的员工进行升序排序，入职时间相同，再按照更新时间进行降序排序
select id, user_Name, password, name, gender, phone, job, salary, entry_data, image, create_time, update_time from emp order by entry_data , update_time desc ;
--分页查询(limit 是MySQL数据库中的方言)
select...from...[where 条件][group by 分组字段 having 分组后过滤条件][order by 排序字段 排序规则]limit 起始索引，查询记录数。
-- 1.从起始索引0开始查询员工数据, 每页展示5条记录
select * from emp limit 0,5;
-- 2.查询 第1页 员工数据, 每页展示5条记录
select * from emp limit 5;
-- 3.查询 第2页 员工数据, 每页展示5条记录
select * from emp limit 5,5;
-- 4.查询 第3页 员工数据, 每页展示5条记录
select * from emp limit 10,5;
--起始索引计算公式：（查询页码 - 1）* 每页显示的记录数。
```

**DCL：**数据控制语言，用来创建数据库用户、控制数据库的访问权限

**多表关系：**

**1.一对多：比如一个部门下有多个员工；在多的一方关联少的一方的主键，就能实现一对多的关系。**

注意：可以设置外键约束，但是在项目中，一般不允许设置物理外键。因为设置物理外键会影响增删改的效率，而且使用的场景不多。

**2.一对一：比如一个表中的信息太多，可以对这个单表的信息进行拆分，可以将一些基础字段放置到另一张表中。**

**3.多对多：比如学生与课程之间的关系，如果需要进行两张表多对多关联，那么需要进行第三张表来进行来关联两张表的主键。**

**多表查询：**
**1.内连接，相当于两张表的交集部分。**

```SQL
--显示内连接
SELECT E.*,D.name FROM emp E INNER JOIN dept D WHERE E.dept_id = D.id;
--隐示内连接
SELECT E.*,D.name FROM emp E , dept D WHERE E.dept_id = D.id;
```

**2.外连接：左外连接，查询左表的所有数据，以及两张表的交集部分；右外连接，查询右表的所有数据，以及两张表的交集部分**

```SQL
--左外连接,条件成立则显示另一张表中的数据，不成立则单独显示本表的数据，另一个表中的字段值为初始值
SELECT E.*,D.name FROM emp E LEFT JOIN dept D ON E.dept_id = D.id;
--右外连接，条件成立则显示另一张表中的数据，不成立则单独显示本表的数据，另一个表中的字段值为初始值
SELECT E.*,D.name FROM emp E RIGHT JOIN dept D ON E.dept_id = D.id;
```

**3.自连接：进行自连接的时候需要取别名来分别（可以使用内连接，也可以使用外连接）**

**子查询：**

**1.标量子查询，返回单个值**

**2.列子查询，返回的结果为一列**

**3.行子查询，返回的结果为一行**

```SQL
SELECT e.* FROM emp e WHERE (e.salary,e.job) = (select salary,job from emp where name = '李忠');
```

**4.表子查询，返回的结果为多行多列**

```SQL
# 获取每个部门中薪资最高的员工信息
# 1.获取员工的最高薪资
select emp.dept_id,max(emp.salary) from emp group by emp.dept_id;
# 2.结合
select e.* from emp E,(select emp.dept_id,max(emp.salary) AS salary from emp group by emp.dept_id) F WHERE E.dept_id = F.dept_id AND E.salary = F.salary
```

#### JDBC：

就是使用Java语言来操作关系型数据库的API。

**案例一：执行DML语句进行数据的修改**

```Java
@Test
public void jdbcDemo() throws Exception {
    //1.获取驱动
    Class.forName("com.mysql.cj.jdbc.Driver");

    //2.获取连接
    String url = "jdbc:mysql://localhost:3306/db01";//数据库地址
    String username = "root";
    String password = "root";
    Connection connection = DriverManager.getConnection(url, username, password);

    //3.获取执行sql的Statement对象
    Statement statement = connection.createStatement();

    //4.执行sql语句
    int i = statement.executeUpdate("UPDATE userInfo SET age = 25 WHERE id = 1");
    System.out.println(i);


    //5.释放资源
    statement.close();
    connection.close();
}
```

**案例二：执行DQL语句进行数据查询**

```Java
    @Test
    public void jdbcDemo2() throws Exception {
        //1.注册驱动
        Class.forName("com.mysql.cj.jdbc.Driver");

        //2.连接数据库
        String url = "jdbc:mysql://localhost:3306/db01";
        String username = "root";
        String password = "root";
        Connection connection = DriverManager.getConnection(url, username, password);

        //3.获取执行sql的Statement对象
        Statement statement = connection.createStatement();

        //4.执行sql语句
        ResultSet resultSet = statement.executeQuery("SELECT id,username,name,password,age,gender " +
                "FROM user WHERE username = 'daqiao' and password = '123456'");
        User user = null;
        while (resultSet.next()) {
            user = new User(resultSet.getInt("id"), resultSet.getString("username"),
                    resultSet.getString("name"), resultSet.getString("password"),
                    resultSet.getInt("age"), resultSet.getInt("gender"));
        }
        System.out.println(user.toString());
    }
```

**预编译SQL:**

预编译SQL在项目中非常的常见，使用预编译SQL的好处是在项目运行中性能会更好，还能防止SQL注入的风险。

**案例三：预编译SQL执行DQL语句**

```Java
@Test
public void jdbcDemo3() throws Exception {
    //1.注册驱动
    Class.forName("com.mysql.cj.jdbc.Driver");

    //2.连接数据库
    String url = "jdbc:mysql://localhost:3306/db01";
    String username = "root";
    String password = "root";
    Connection connection = DriverManager.getConnection(url, username, password);

    //3.预编译SQL
    String sql = "SELECT id,username,name,password,age,gender FROM user WHERE username = ? and password = ?";
    PreparedStatement preparedStatement = connection.prepareStatement(sql);
    preparedStatement.setString(1, "daqiao");
    preparedStatement.setString(2, "123456");

    //4.执行SQL
    ResultSet resultSet = preparedStatement.executeQuery();
    User user = null;
    while (resultSet.next()) {
        user = new User(resultSet.getInt("id"), resultSet.getString("username"),
                resultSet.getString("name"), resultSet.getString("password"),
                resultSet.getInt("age"), resultSet.getInt("gender"));
    }
    System.out.println(user.toString());

    //5.释放资源
    resultSet.close();
    preparedStatement.close();
    connection.close();
}
```

#### Mybatis：

持久层框架，简化JDBC开发。

**案例一：查询所有的员工信息，并返回输出**

```Java
// 封装类
@Data
@AllArgsConstructor
@NoArgsConstructor
public class User {
    private Integer id;
    private String username;
    private String name;
    private String password;
    private Integer age;
    private Integer gender;

    @Override
    public String toString() {
        return "User{" +
                "id=" + id +
                ", username='" + username + '\'' +
                ", name='" + name + '\'' +
                ", password='" + password + '\'' +
                ", age=" + age +
                ", gender=" + gender +
                '}';
    }
}

@Mapper // 在运行的时候会动态的额为该接口创建一个实现类对象（代理对象），并且会自动的将接口的实现类对象注入到spring的IOC容器中
public interface UserMapper {
    /**
     * 查询所有用户
     * @return List<User> 用户集合
     */
    @Select("select id,username,name,password,age,gender from user")
    List<User> findAll();
}

@SpringBootTest //SpringBoot单元测试
class MybatisQuickApplicationTests {

    @Autowired
    @Qualifier("userMapper")
    private UserMapper userMapper;

    @Test
    void userMapperTest() {
        List<User> userList = userMapper.findAll();
        userList.forEach(System.out::println);
    }
}
```

**注意：@Mapper注释需要写在接口上。该注解的作用是在程序启动的时候会自动的创建一个该接口的实现类（代理对象）并且会将创建出来的实现类自动的放在Spring的IOC容器中，在需要使用的时候直接通过依赖注入就能够使用。**

**数据库连接池：是一个容器，负责分配、管理数据库连接（Connection），允许程序重复的运用一个数据库连接，而不是重新创建一个。释放空闲时间超过最大空闲时间的连接，来避免因为没有释放连接而引来的数据库连接的遗漏（解释：如果一个用户获得连接池中的一个连接，但是不去进行数据库操作，只是占用一个连接，mybatis会自动去检测连接的空闲时间，如果超过了预设的最大空闲时间，这个连接会直接归还给数据库连接池。）如果要实现数据库连接池的创建，那么就需要实现官方的接口DataSource。**

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250317114255591.png" alt="image-20250317114255591" style="zoom:50%;" />

**案例二：根据id删除用户信息**

```Java
@Delete("delete from user where id=#{id}")
int deleteById(Integer id);

@Delete("delete from user where id=#{id}")//#{} -->是占位符
int deleteById(Integer id);
```

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250317120541771.png" alt="image-20250317120541771" style="zoom:50%;" />

**案例三：新增用户信息**

```Java
@Insert("insert into user(username, name, password, age, gender) values(#{username},#{name},#{password},#{age},#{gender})")
int insertByUser(User user);

@Test
void userMapperTest3(){
    User user = new User();
    user.setUsername("liyunjun");
    user.setName("李云君");
    user.setPassword("123456");
    user.setAge(18);
    user.setGender(1);
    int i = userMapper.insertByUser(user);
    System.out.println(i);
}
```

**案例三：更具用户id更新用户信息**

```Java
@Update("update user set username=#{username},name=#{name},password=#{password},age=#{age},gender=#{gender} where id = #{id}")
int updateById(User user);


@Test
void userMapperTest4(){
    User user = new User();
    user.setId(7);
    user.setUsername("liyunjun");
    user.setName("李云君");
    user.setPassword("123456");
    user.setAge(22);
    user.setGender(1);
    int i = userMapper.updateById(user);
    System.out.println(i);
}
```

**案例四：根据用户名和密码查询用户信息**

```Java
@Select("select id,username,name,password,age,gender from user where username=#{username} and password=#{password}")
User findByUsernameAndPassword(@Param("username") String username, @Param("password") String password);

void userMapperTest5(){
    User user = userMapper.findByUsernameAndPassword("liyunjun","123456");
    System.out.println(user);
}
```

@param注解的作用是为接口的方法形参起名字的。（由于用户名唯一的，所以查询返回的结果最多只有一个，可以直接封装到一个对象中），基于官方骨架创建的SpringBoot项目中，接口编译时会保留方法形参名，@Param注解可以省略（#{形参名}）

##### xml映射配置文件：

**规则：**

**1.xml映射配置文件的名称与mapper接口名称一致，并且xml映射文件和mapper接口放置在相同包下。**

**2.xml映射文件的namespace属性为mapper接口全限定名一致**

**3.xml映射文件中sql语句的id与mapper接口中的方法名一致，并保持返回类型一致。**

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "https://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.lihonbin.mapper.UserMapper">
    <!--resultType:查询返回的单条记录所封装的类型-->
    <!-- 查询所有用户信息 -->
    <select id="findAll" resultType="com.lihonbin.pojo.User">
        select id, username, name, password, age, gender  from user
    </select>
</mapper>
```

使用规则（官方规定）：如果是简单的SQL语句官方建议使用的是注解来映射，复杂的SQL语句建议使用的是XML映射的方式。

#### 开发实战：

##### 开发规范：Resultful风格

```Java
//GET请求---表示查询
//POST请求---表示新增
//DELETE请求---表示删除
//PUT请求---表示修改
```

##### 数据封装：

**第一种方式：使用@Results注解以及@Result注解进行手动的映射**

```Java
@Results({
        @Result(column = "create_time", property = "createTime"),
        @Result(column = "update_time", property = "updateTime")
})
@Select("select id, name, create_time, update_time from dept order by update_time desc ")
List<Dept> getDeptList();
```

**第二种方式：运用SQL语句中的起别名的方式来进行映射。**

```Java
@Select("select id, name, create_time createTime, update_time updateTime from dept order by update_time desc ")
List<Dept> getDeptList();
```

**第三种方式：开启驼峰命名规则，让mybatis进行自动映射**

```yml
mybatis:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
    #开启驼峰命名映射
    map-underscore-to-camel-case: true
```

##### NGINX：

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250318135918233.png" alt="image-20250318135918233" style="zoom:50%;" />

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250318140034649.png" alt="image-20250318140034649" style="zoom:50%;" />

**反向代理技术：可以为后续对后端请求进行进行负责均衡，而且不容易暴露后端的接口会更加的安全。**

##### 获取简单参数：

**方式一：使用Java提供的HttpServletRequest封装类来获取。**

```Java
public Result deleteDeptById(HttpServletRequest request) {
    Integer id = Integer.parseInt(request.getParameter("id"));
    deptService.deleteDeptById(id);
    return Result.success();
}
```

**方式二：使用注解@RequsetParam将请求参数直接绑定给方法形参**

```Java
//一旦声明了@RequestParam，该参数在请求的时候默认必须进行传递，如果不进行传递会报错（默认：required=true）
public Result deleteDeptById(@RequestParam("id") Integer id) {
    deptService.deleteDeptById(id);
    return Result.success();
}
```

**[推荐]方式三：如果参数名与形参变量名相同，直接定义方法形参即可接收。（省略@RequestParam）**

```Java
public Result deleteDeptById(Integer id) {
    deptService.deleteDeptById(id);
    return Result.success();
}
```

##### 获取JSON请求参数：

**使用@RequstBody来接收前端发送的json数据，并映射到Java的类中。**

```Java
@PutMapping
public Result updateDept(@RequestBody Dept dept) {
    deptService.updateDept(dept);
    return Result.success();
}
```

##### 获取路径参数：

**使用@PathVariable注解来进行接收，如果路径的参数名与controller中方法的形参名一致的话，那么这个注解可以进行省略。**

```Java
@GetMapping("/{id}")
public Result getDeptById(@PathVariable("id") Integer id) {
    Dept dept = deptService.getDeptById(id);
    return Result.success(dept);
}
```

##### 日志技术：

**记录应用程序的运行信息、状态信息、错误信息。**

**1.JUL:官方日志框架，配置简单，性能较差，不够灵活。**

**2.Log4j:流行的日志框架，提供了灵活的配置选项，支持多种输出目标。**

**3.Logback：基于Log4j升级而来，提供了更多的功能和配置选项，性能优于Log4j**

**4.slf4j：简单日志门面，提供了一套日志操作的标准接口以及抽象类，允许应用程序使用不同的底层日志框架。**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration debug="false">
    <!--控制台输出-->
    <!--consoleAppender 表示的是向控制台进行输出-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d表示日期，%thread表示线程名，%-5level:级别从左显示5个字符宽度， %logger{36}：最长36个字符，最长切割-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 按照每天生成日志文件 -->
    <!--RollingFileAppender 表示的是向文件进行输出-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <!-- 日志文件输出的文件名, %i表示序号 -->
            <FileNamePattern>D:/JavaWorkPace/_2025.03.12.WebFoundation/TliasTeachingManagementSystem-%d{yyyy-MM-dd}-%i.log</FileNamePattern>
            <!-- 最多保留的历史日志文件数量 -->
            <MaxHistory>30</MaxHistory>
            <!-- 最大文件大小，超过这个大小会触发滚动到新文件，默认为 10MB -->
            <maxFileSize>10MB</maxFileSize>
        </rollingPolicy>

        <encoder class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <!--格式化输出：%d 表示日期，%thread 表示线程名，%-5level表示级别从左显示5个字符宽度，%msg表示日志消息，%n表示换行符 -->
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50}-%msg%n</pattern>
        </encoder>
    </appender>


    <!-- 日志输出级别 -->
    <root level="ALL">
        <!--输出到控制台-->
        <appender-ref ref="STDOUT"/>
        <!--输出到文件-->
        <appender-ref ref="FILE"/>
    </root>
</configuration>
```

**配置文件：配置文件控制的是日志输出的，可以用来配置输出的格式，位置，日志的开关等；常用的两种输出日志位置是：控制台输出、系统文件输出。**

**日志输出级别：级别由低到高**

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250319105509662.png" alt="image-20250319105509662" style="zoom:80%;" />

##### 事务：

是一组操作的集合，是一个不可分割的工作单位。事务会把所有的操作作为一个整体一起向系统提交或撤销操作请求，所以这些操作，要么成功，要么失败。

**事务执行流程：开启事务--提交事务/回滚事务**

```SQL
--开启事务
start transaction;
--提交事务
commit;
--回滚事务
rollback;
```

**事务的四大特性：ACID**

**A：原子性，要么同时成功，要么同时失败。**

**C：事务完成之后，必须让所有数据都保持一致。**

**I：隔离性。**

**D：持久性，事务一旦提交或者回滚数据的改变都是永久的。**

**Spring中的事务管理使用@Transactional，可以作用于方法，类，接口；一般写在方法上，默认的事务规则是方法运行错误才会进行事务的回滚。通过使用属性rollbackFor，可以来规定出现何种异常才出现事务的回滚。如果一个开启事务的方法去调用另一个被事务控制的方法，那么就需要指定另一个方法的在运行时的事务是加入到现有的事务中，还是重新开启一个新的事务。**

**propagation：**

<img src="C:\Users\李洪斌\AppData\Roaming\Typora\typora-user-images\image-20250319175341030.png" alt="image-20250319175341030" style="zoom: 80%;" />



##### SpringAOP：

面向切面编程

**1.连接点joinPoint：可以被AOP控制的方法（暗含方法执行时的相关信息）**

```Java
//该概念是一种广义的概念，我的理解是：在SpringBoot项目中，基本上所有的方法都可以被称为连接点。包含Controller中的方法，service中的方法以及mapper中的方法等。
```

**2.通知Advice：指哪些重复的逻辑，也就是共性功能**

```Java
这里的通知指的就是在连接点前后需要执行的逻辑代码。
    @Around("execution(* com.lihonbin.service.impl.*.*(..))")// 指定切点-环绕通知
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
    
        Object result = joinPoint.proceed();//在本方法内，除去本行都可以算是通知
       
    	long end = System.currentTimeMillis();
        System.out.println(joinPoint.getSignature() + "方法执行耗时：" + (end - start) + "ms");
        return result;
    }
```

**3.切入点pointcut：匹配连接点的条件**

```Java
//也就是实际被AOP所控制的类
@Around("execution(* com.lihonbin.service.impl.*.*(..))")//指定的切入点表达式中的所有方法
```

**4.切面aspect：描述通知与切入点的对应关系**

```Java
    @Around("execution(* com.lihonbin.service.impl.*.*(..))")// 指定切点-环绕通知
    public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
    
        Object result = joinPoint.proceed();
       
    	long end = System.currentTimeMillis();
        System.out.println(joinPoint.getSignature() + "方法执行耗时：" + (end - start) + "ms");
        return result;
    }
//以上所有内容表示切面，包含切面的类被称为切面类
```

**5.目标对象：Target，通知所有应用的对象**

```Java
//切入点表达式所指定的所有类
```

执行流程：

1. 执行切面中要去执行原方法之前的逻辑->执行原方法->执行原方法之后的逻辑

2. AOP是根据动态代理的方式实现的：

   1. 生成一个动态代理对象，对象和原本的目标对象继承、接口一致。

   2. 在动态代理对象中，有被切入的方法。方法的内容将被重写。

      ```Java
      
      //通知
      @Around("execution(* com.lihonbin.service.impl.*.*(..))")// 指定切点-环绕通知
      public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
          long start = System.currentTimeMillis();
          Object result = joinPoint.proceed();
          long end = System.currentTimeMillis();
          System.out.println(joinPoint.getSignature() + "方法执行耗时：" + (end - start) + "ms");
          return result;
      }
      //切入点
      @Override
      public Emp getEmpById(Integer id) {
          Emp emp = empMapper.getEmpById(id);
          empExprService.selectEmpExprByEmpId(emp);
          return emp;
      }
      //代理对象中的内容
      @Override
      public Emp getEmpById(Integer id) {
          long start = System.currentTimeMillis();
          Object result = joinPoint.proceed();    
          Emp emp = empMapper.getEmpById(id);
          empExprService.selectEmpExprByEmpId(emp);
          long end = System.currentTimeMillis();
          System.out.println(joinPoint.getSignature() + "方法执行耗时：" + (end - start) + "ms");    
          return emp;
      }
      ```

通知类型：

- 前置通知，在目标方法前执行。@Before
- 后置通知，在目标方法后执行，不论有无异常。@After
- 环绕通知，在目标方法前后都执行。@Around
- 返回后通知，目标方法返回后执行，有异常不执行。@AfterReturning
- 异常后通知，目标方法发生异常后执行。@AfterThrowing

@Pointcut：可以将公共的切入点表达式抽取出来，需要时直接引用该表达式即可。

```Java
@Pointcut("execution(* com.lihonbin.service.impl.*.*(..))")
private void pt(){}
@Around("pt()")
```

通知顺序：

```Java
//不同的切面类中
//目标方法执行前的通知方法：按照切面类的字母排名靠前的先执行
//目标方法执行后的通知方法：按照切面类的字母排名靠后的先执行
//辅助理解：可以理解为栈模式，先进先出，后进后出。
//可以使用注解：@order(1)--前置通知：数字越小先执行；后置通知：数字越小后执行
```

##### 全局异常处理器

```Java
@RestControllerAdvice
public class GlobalExceptionHandler {

    //处理全局异常
    @ExceptionHandler
    public Result handleException(Exception e) {
        e.printStackTrace();
        return Result.error("操作失败！请一段时间后重试！");
    }
}
```

##### 会话技术

会话：指的是在打开浏览器访问web服务器资源的时候，会话建立。直到有一方断开连接，会话结束。在一次会话的过程中，可以包含多次的请求和响应。

会话跟踪：维护浏览器状态的方法，服务器需要识别多次请求是否来自于同一浏览器，以便在同一次会话的多次请求间共享数据。

- Cookie(客户端会话跟踪技术):

  - 优点：
    - Http协议中支持的技术（请求:Cookie/响应头:Set-Cookie）
  - 缺点：
    - 移动端App无法使用Cookie
    - 不安全，用户可以自己禁用Cookie
    - Cookie不能进行跨域
      - 跨域的区分：协议、IP/域名、端口

- Session(服务端会话跟踪技术，基于Cookie实现):

  - 优点：
    - 存储在服务端，安全
  - 缺点：
    - 服务器集群环境下无法直接使用Session
    - Cookie的缺点

- 令牌技术:

  在登录成功之后，会生成一个访问权限的令牌并将该令牌返还给客户端，在客户端进行下一次请求的时候，在请求中会携带令牌，服务器会拦截请求并检验令牌的正确性和有效性。

  - 优点：
    - 支持PC端、移动端
    - 解决集群环境下的认证问题
    - 减轻服务端存储压力
  - 缺点：需要手动实现。

##### Filter

Filter过滤器是JavaWeb三大组件（servlet、filter、listener）。过滤器可以把对资源的请求拦截下来，从而实现一些特殊的功能。

实现步骤：

1. 定义一个filter，实现filter接口，并实现其所有方法

   ```Java
   @WebFilter("/*")
   public class LoginFilter implements Filter {
   
       @Override
       public void init(FilterConfig filterConfig) throws ServletException {
           Filter.super.init(filterConfig);
       }
   
       @Override
       public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
           HttpServletRequest request = (HttpServletRequest) servletRequest;
           HttpServletResponse response = (HttpServletResponse) servletResponse;
   
           String url = request.getRequestURL().toString();
           if (url.contains("/login")) {
               filterChain.doFilter(servletRequest, servletResponse);
           }
           String token = request.getHeader("token");
           if (!StringUtils.hasLength(token)) {
               response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
               return;
           }
           try {
               JwtUtils.parseToken(token);
           } catch (Exception e) {
               response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
           }
           filterChain.doFilter(servletRequest, servletResponse);
   
       }
   
       @Override
       public void destroy() {
           Filter.super.destroy();
       }
   }
   ```

2. 配置filter，filter类上加上注解@WebFilter注解，引导类上加@ServletComponetScan开启Servlet组件支持。

执行流程：

1. 执行放行前的逻辑。
2. 放行（doFilter）
3. 执行放行后的逻辑。

过滤器链：

可以配置多个过滤器，形成一个过滤器链。顺序是：注解配置的Filter，优先级是按照过滤器类名的自认顺序进行过滤的。

##### Interceptor

是一种动态拦截方法调用的机制，类似于过滤器。Spring框架提供，主要用来动态拦截控制器方法的执行。拦截请求，在指定的方法调用前后，根据业务需要执行预选设定的代码。

实现步骤：

1. 定义拦截器，实现HandlerInterceptor接口，并实现其所有方法

   ```Java
   @Component// 将当前类标记为拦截器
   public class LoginInterceptor implements HandlerInterceptor {
   
       @Override
   // 在请求处理之前进行调用（Controller方法调用之前）,返回true表示继续流程（如调用下一个Interceptor）,返回false表示中断流程（如不再调用其他Interceptor和Controller）
       public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
           //1.获取请求路径
           String url = request.getRequestURL().toString();
           if (url.contains("/login")) {// 放行登录请求
               return true;
           }
           // 2.判断是否包含token
           String token = request.getHeader("token");
           if (!StringUtils.hasLength(token)) {// 如果没有token，则返回401
               response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
               return false;
           }
           // 3.解析token
           try {
               JwtUtils.parseToken(token);
           } catch (Exception e) {// 如果解析失败，则返回401
               response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
               return false;
           }
           // 4.放行
           return true;
       }
   
       @Override// 请求处理之后进行调用（Controller方法调用之后），但是在DispatcherServlet进行视图返回渲染之前（Controller方法调用之后）
       public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
       }
   
       @Override// 整个请求结束之后被调用，也就是在DispatcherServlet 渲染了对应的视图之后执行（主要是用于进行资源清理工作）
       public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
       }
   }
   ```

2. 注册拦截器

   ```Java
   @Configuration// 将当前类标记为配置类
   public class WebConfig implements WebMvcConfigurer {
   
       @Autowired
       private LoginInterceptor loginInterceptor;
   
       @Override
       public void addInterceptors(InterceptorRegistry registry) {
           // 注册拦截器,并指定需要拦截的请求路径
           registry.addInterceptor(loginInterceptor).addPathPatterns("/**").excludePathPatterns("/login");
       }
   }
   ```

过滤器和拦截器同时存在的时候，先执行过滤器再执行拦截器。

### Spring相关原理篇

#### SpringBoot配置优先级 

配置文件：properties、yml、yaml

上述三种配置文件同时 存在的情况下，properties文件会被框架优先的进行加载。如果没有properties文件，那么第二优先加载的文件是yml文件，最后才是yaml文件。

在启动的时候也能 进行配置,命令行的优先级高于系统配置

```bash
#Java系统配置
-Dserver.port=9090
#命令行参数
--server.port=1000
#运行程序的打包为的jar包
java -Dserver.port=9090 -jar 打包的jar包 --server.port=1000
```

#### Bean管理

##### bean 的作用域

-spring支持五种作用域

- singleton：IOC容器中同名称的bean只有一个实例（单例）spring默认
- prototype：每次使用该bean时会创建新的实例（非单例/多例）
- request：每个请求范围内会创建新的实例（web环境）
- session：每个会话范围内会创建新的实例（web环境）
- applaction：每个应用范围内会创建新的实例（web环境）

```Java
@Scop("prototype") //在bean类上使用该注解，可以指定为多例
//IOC容器：ApplicationContext
//默认bean的创建是在项目启动的时候进行的，创建完毕会将创建出来的bean对象放入IOC容器中管理
@Lazy //可以指定bean对象的创建时间，使用该注解可以使被该注解标注的类只有在使用的时候才开始创建。
```

1. 无状态的bean（singleton）：无状态指的是，在bean中没有进行额外数据的保存，这样可以避免多线程相关的问题。
2. 有状态的bean（prototype）：有状态指的是，在bean中有进行额外的数据保存，可能会有多线程风险，使用多例可以进行避免。

##### 第三方bean

第三方的jar包中的类，如果要交给IOC容器进行管理，是无法使用@Component以及衍生的注解声明bean的，这个时候就需要使用@bean注解。使用@Bean注解可以通过其name或者value属性来声明bean的名称，如果不指定，那么默认的bean的名称就是方法名。如果第三方的bean需要依赖其他bean对象，直接在bean定义方法中设置形参即可，容器会根据类型自动装配。

```Java
@SpringBootApplication//不建议把第三方bean的管理放入启动类中，可以创建一个单独的配置类进行配置。
public class TliasTeachingManagementSystemApplication {

    public static void main(String[] args) {
        SpringApplication.run(TliasTeachingManagementSystemApplication.class, args);
    }

    @Bean
    public JwtUtils jwtUtils() {
        return new JwtUtils();
    }
    
}
```

#### SpringBoot的原理

##### 起步依赖

在pom文件中的starter坐标。其本质就是Maven中的依赖传递来进行依赖的注入。

##### 自动配置

在spring项目启动之后，一些配置类、bean对象就自动的存入到了IOC容器中，不需要我们进行手动的声明，简化了开发。

- 自动配置实现的方案

  - 方案一：第三方包中类上加上注解@Component注解，在项目的启动类上加上注解@ComponentScan来设置包扫描的路径，虽然启动类注解包含包扫描的功能，但是如果加上了注解@ComponentScan注解那么默认的包扫描就会被替换，所以需要在@ComponentScan的属性basePackage上加上第三方包和原本默认的包路径。（不推荐，繁琐且性能低）
  - 方案二：使用@Import注解，这个注解可以导入普通的实体类，也可以导入配置类，可以导入实现了importSelector接口的实现类。
  - 方案三：可以将@import注解封装到一个全新的注解中来实现，@EnableXxx注解。

- Spring如何实现的bean注入

  在启动类上有一个核心注解@SpringBootApplaction，其中整理了三个核心的注解。分别是@ComponentScan，这个注解就是制定了默认的包扫描；@SpringBootConfiguration，这个注解包含了@configuration注解，说明这个类也是配置类，所以在启动类中也可以进行bean的注册；@EnableAotoConfiguration，这个注解主要的作用就是进行bean的注册，其包含了@Import注解，其中注入的类是AutoConfigurationImportSelector。在这个类中实现了接口ImpoetSelector接口中的selectImports方法，在这个方法内就指定了需要注入的类，这些类的全类名被写在一个文件中，这个文件是.imports结尾的文件。

- @Conditional

  - 作用：按照一定的条件进行判断，在满足条件之后才会注册对应的bean到ioc容器中
  - 位置：方法，类
  - @Conditional本身是一个父注解，其衍生的注解
    - @ConditionalOnClass：判断环境中是否有对应的字节码文件，有才注册bean到ioc容器中
    - @ConditionalOnMissingBean：判断环境中没有对应的bean（类型或名称），才注册bean到容器中
    - @ConditionalOnProperty：判断配置文件中有对应属性和值，才注册bean到ioc容器中







## Web基础问题

#### HTTP协议：

##### 请求协议：

**1.HTTP请求数据需要程序员自己解析吗？**

回答：不需要程序员自己进行解析，在请求协议交给后端的时候Tomcat会将请求数据的内容封装到httpServletRequest类中。在调用Controller方法的时候会将封装了请求数据的httpServletRequest传递给该方法。

**2.如何获取请求数据？**

回答：首先请求数据发送只会web服务器会将发送的请求数据进行封装，在调用Controller方法的时候会将该封装类传递给该方法。随后可以通过该类中的一些方法获取封装在内的请求数据。

##### 响应协议：

**1.HTTP响应数据需要程序员自己手动设定吗？**

回答：不需要，web服务器会将需要响应的数据封装到HttpServletResponse中。

**2.响应状态码、响应头需要手动指定吗？**

回答：不需要，web服务器会自动的进行设定。

#### 分层解耦：

##### IOC&DI:

**1.如何将一个类交给IOC容器管理？**

回答：首先需要在该类上加上注解@Component，需要注意的是加在实现类上不是接口上。但是如果只是单独的加上注解那么不一定会将该类交给IOC容器进行管理。因为在springboot中，如果需要将类交给IOC容器进行管理，除了加上注解之外，还需要被组件扫描器进行扫描，组件扫描器默认扫描的是启动类所在的包及其子包，所以如果一个类交给IOC容器进行管理，需要加上注解和被组件扫描器扫描到。

**2.如何从IOC容器中找到该类型的bean，然后完成依赖注入？**

回答：spring提供了一个注解是@Autowired，该注解的作用就是在IOC容器中找到需要使用的bean，然后对该bean对象进行赋值。该注解默认是按照类型进行注入的，所以如果一个接口出现多个实现类并且都交给了IOC容器进行管理，那么就会spring不知道将哪个bean进行注入，所以有如下三种解决方式，第一种就是@Autowired与@Qualifier两个注解一起使用，@Qualifier可以指定注入类的名称，在将类交给IOC容器进行管理的时候如果不指定类在IOC容器中的名称的话，那么默认的就是该类的首字母小写。第二种方式就是在需要默认注入的类上加上新注解@Primary，第三种就是Java自带注解@Resource，根据该注解的name属性指定需要注入类的名称。

**3.@Autowired与@Resource的区别？**

回答：1.两者的出处不同；2.@Autowired默认是按照类型进行注入的，@Resource默认是按照名称进行注入的。

#### MySQL：

**1.MySQL数据库中的约束有哪些，对应的关键字是什么？**

回答：约束分别有1.主键约束，关键字是pirmary key；2.外键约束，关键字是foreign key；3.非空约束，not null；4.唯一约束，unique；5.默认约束，default

**2.如何实现主键自增的效果？**

回答：在主键约束之后加上关键字，auto_increment。主键自增的效果体现在会在上一条数据的主键基础上进行自增。

**3.一个字段上是否可以添加多个约束？**

回答：可以，比如可以同时添加非空与唯一的约束。

**4.数值类型在定义的时候，后面加上了unsigned关键字是什么意思？**

回答：表示当前字段所记录的数值的范围是从0开始的，不包含负数。

**5.char和varchar的区别是什么？什么时候用char，什么时候用varchar？**

回答：首先两者都是存储字符串类型的，char是定长字符串，varchar是变长字符串。char在规定了存储最大长度之后，那么在存储字符串不超过这个范围的时候，不论存储多少个字符串那么它都需要在空间上占据最大长度的空间；varchar会动态的改变存储的空间，char在性能上优于varchar，varchar在磁盘空间的占用上优于char。在知道需要存储的字符串是固定长度的时候，选择char。在不知道存储的字符串的长度时候，使用的是varchar。****

**6.基本查询语法？如何为查询返回的字段设置别名？如何去除查询返回的重复值？**

回答：select 字段列表 from 表名列表。使用as关键字，可以省略，使用关键字distinct。

**7.null值判断？模糊匹配？多个条件？**

回答：1.is null/is not null ;2._表示一个字符，%表示多个字符;3.用 and 或者 or 分开

**8.where与having之间的区别？**

回答：where与having的执行时机不同，where是在分组之前进行过滤，不满足where的条件那么就不会参与分组；having是在分组之后对结果进条件过滤的。where与having判断条件不同，where不能对聚合函数进行判断，having可以对聚合函数进行判断。

#### JDBC：

**1.JDBC是什么？JDBC的执行流程？**

回答：JDBC是Java提供的操纵关系型数据库的一套API。JDBC执行的流程是，首先需要引入数据库厂商给出的依赖包。其次的顺序是，注册驱动--建立数据库连接--获取执行SQL语句的statement对象--执行SQL--释放资源。

**2.JDBC执行DML语句？DQL语句？DQL语句执行完毕结果集ResultSet解析？**

回答：执行DML语句使用的是statement中的excuteUpdate方法，该方法的返回值为影响数据中数据的行数。DQL语句使用的是statement中excuteQuery方法，该方法的返回值是一个结果集。ResultSet解析,resultSet.next()让光标下移一行，resultSet.getXxx(字段名)获取字段数据。

**3.如何执行预编译SQL？为什么要使用预编译SQL？**

回答：在JDBC中需要使用一个对象为PreparedStatement对象来对SQL进行预编译，在进行条件判断的时候用占位符‘？’书写，后续在执行之前用setString(index,条件)来进行占位符的替换。使用预编译SQL的好处是性能会更好，安全性更高。性能更好的原因会将当前预编译的SQL通过解析优化之后保存在缓存中，之后执行的时候直接从缓存中拿取预编译的SQL就行。

#### Mybatis：

**1.JDBC与mybatis 的区别？**

回答：前者基本都是硬编码，耦合度太高不易于项目的维护，并且会造成资源的浪费。后者易于维护，配置简单，有数据库的连接池不会造成资源的浪费。

**2.Mybatis执行 DML语句的时候，返回值是什么？**

回答：返回值是int类型数字，具体表示的是执行该SQL语句影响的记录数。

**3.mybatis中#与$的区别是什么？**

回答：#是占位符。会替换？生成预编译SQL；$是字符连接符号，将参数值直接拼接在SQL中。

**4.@Param注解的使用场景？**

回答：如果在接口方法的形参上，需要传递多个参数，那么就需要使用@Param注解为参数起名字。如果是基于SpringBoot官方骨架创建的项目中，@Param是可以进行省略的。

#### NGINX：

**1.反向代理？反向代理的配置？**

回答：反向代理是一种网络架构技术，通过反向代理服务器为后端服务器做代理（安全，灵活，负责均衡）location：定义路径匹配规则 ；rewrite：定义路径重写的规则；proxy_pass:代理转发指令，后续跟代理转发的路径。

#### 日志技术：

**1.Logback记录日志的步骤？**

回答：引入Logback的依赖，配置文件logback.xml；定义日志对象logger，调用方法记录日志。

#### Cookie：

**1.Cookie会话跟踪方案的原理？**

回答：HTTP协议中的请求头和响应头，服务端会将Cookie对象中的内容放入响应头中的Set-Cookie中，浏览器接收之后会将其中的内容保存到浏览器中，在当前会话中请求服务端时，会将保存在浏览器中的Cookie全部放入请求头中的Cookie中，服务端可以获取其中的内容进行甄别。

**2.Cookie技术的优缺点？**

回答：优点是实现起来简单，http协议支持这项技术。缺点是：移动端不能使用这项技术、Cookie不能进行跨域、用户能自行的对Cookie进行设置不安全。

#### Session：

**1.Session会话跟踪方案的原理？**

回答：Session其实还是基于Cookie实现的，但是信息的内容存储的位置发生了变化，从客户端能够查看到具体的内容到服务端存储具体的信息，返还给客户端的内容只是Session在服务端中的ID值。（HttpSession）

**2.Session会话跟踪方案的优缺点？**

回答：优点：Session会话技术存储在服务端更加的安全。缺点：因为是基于Cookie实现的，所以Cookie的所有缺点都有，而且Session技术不能支持服务器集群模式。因为同一次会话可能请求的是服务器集群的不能服务器。

#### Filter：

**1.Filter的执行流程？**

回答：Filter是JavaWeb的三大组件之一，其作用是拦截对资源的请求并对决定是否放行该请求。具体的执行流程是：拦截对资源的请求->放行->请求资源->返回过滤器执行放行后的逻辑

#### Bean：

**1.Spring中的bean是单例还是多例的？单例的bean是什么时候实例化？**

回答：Spring中的bean默认是单例的，这样设置可以减少资源的浪费。如果需要设置成多例的bean，那么在需要设置的类上加上注解@Scop（"prototype"）就可以设置为多例。单例的bean默认的都是饥饿式的加载，都会在项目启动的时候进行加载。如果要修改单例bean加载的时间，可以使用注解@Lazy,这样单例的bean就不会在项目启动的时候进行加载，首次的初始化会在这个bean被使用的时候。

**2.Spring容器的bean是线程安全的吗？**

回答：Spring中的bean的线程安全取决于bean的状态和bean的作用域。如果是无状态的bean那么Spring容器中的bean就是线程安全的，如果是有状态的bean并且是单例模式，那么这个bean就是线程不安全的，如果这个有状态的bean是多例的模式那么这个bean就是线程安全的。

**3.什么时候使用@component？什么时候使用@Bean？**

回答：在项目中，如果是自己本身写的类那么可以使用@Component注解类声明bean，如果是引用的第三方依赖那么就需要使用@Bean注解来注册第三方的bean对象。

**4.为什么第三方依赖中使用@Component及其衍生注解声明bean不生效？**

回答：声明的bean的方式有很多，但是如果只是简单的声明bean，IOC容器不一定会进行管理。因为如果需要IOC容器来管理声明的bean，那么就需要进行一次组件扫描，IOC会将扫描到的需要管理的bean注入到容器中，在启动类注解中会进行默认的组件扫描，扫描的范围是启动类所在的包和这个包的子包，引入的第三方依赖包不在这个范围内，所以进行扫描的时候扫描不到这些需要进行IOC容器管理的bean，如果需要这个bean进行生效，那么就需要手动的指定需要扫描的包的路径，进行手动指定之后，默认的扫描包的路径就会被替换，所以在指定的时候还需要将原本默认扫描包的路径进行指定。

**5.有哪些方案可以使第三放的bean进行生效？**

回答：如果第三方的类上有@Component注解，那么就可以使用@ComponentScan来指定包扫描的路径，将第三方包的路径加入就能进行扫描。也可以使用@Import注解来进行导入，该注解可以导入的内容有，普通类，配置类，实现了接口ImportSelector的实现类。也可以定义一个全新的注解来整理Import注解。

**6.@Conditional注解及其衍生的注解的作用是什么？**

回答：该注解及其衍生的注解的作用是判断是否将bean注册到容器的条件是否满足。

**7.自动配置的核心是什么？如何自定义？**

回答：自动配置的核心是@EnableAutoConfiguation注解中的@Import注解中字节码文件中的方法SelectImports方法中的逻辑。自己实现自动配置则需要将需要自动配置的类写入一个.imports文件中。