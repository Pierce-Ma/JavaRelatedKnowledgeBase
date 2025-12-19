
# 快速入门
现有一张 User 表，其表结构如下：

| id  | name   | age | email                                           |
| --- | ------ | --- | ----------------------------------------------- |
| 1   | Jone   | 18  | [test1@baomidou.com](mailto:test1@baomidou.com) |
| 2   | Jack   | 20  | [test2@baomidou.com](mailto:test2@baomidou.com) |
| 3   | Tom    | 28  | [test3@baomidou.com](mailto:test3@baomidou.com) |
| 4   | Sandy  | 21  | [test4@baomidou.com](mailto:test4@baomidou.com) |
| 5   | Billie | 24  | [test5@baomidou.com](mailto:test5@baomidou.com) |
其对应的数据库 Schema 脚本如下：
DROP TABLE IF EXISTS `user`;

CREATE TABLE `user`
(
    id BIGINT NOT NULL COMMENT '主键ID',
    name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
    age INT NULL DEFAULT NULL COMMENT '年龄',
    email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
    PRIMARY KEY (id)
);
其对应的数据库 Data 脚本如下：
DELETE FROM `user`;

INSERT INTO `user` (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');

**接下来先引入依赖，如果是Maven类型的（大部分现在都是Maven）**
<dependency>

<groupId>com.baomidou</groupId>

<artifactId>mybatis-plus-spring-boot3-starter</artifactId>

<version>3.5.15</version>

</dependency>


在application.yml文件中我们添加H2数据库的相关配置：
spring:
  datasource:
    driver-class-name: org.h2.Driver
    username: root
    password: test
  sql:
    init:
      schema-locations: classpath:db/schema-h2.sql
      data-locations: classpath:db/data-h2.sql
在 Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：
@SpringBootApplication
//注意，这里的包不是的地址不是随便写的，**它扫描的是Mapper 接口的包路径**
@MapperScan("com.baomidou.mybatisplus.samples.quickstart.mapper")
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}

**接着编写实体类（这里使用了lombok）**

@Data
@TableName("`user`")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}

**编写 Mapper 接口类 `UserMapper.java`：**
public interface UserMapper extends BaseMapper<User> {

}

**添加测试类，进行功能测试:**
@SpringBootTest
public class SampleTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        //z这里的selectlist写Null就是选择所有*
        List<User> userList = userMapper.selectList(null);
        //> 我期望数据库里现在**刚好有 5 条用户数据**，> 如果不是 5 条，这个测试就是错的，这里的“”这个地方应该写的是报错信息，但是这里懒得写了，毕竟是简单的入门案例
        Assert.isTrue(5 == userList.size(), "");
        userList.forEach(System.out::println);
    }

}
控制台输出：
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
User(id=5, name=Billie, age=24, email=test5@baomidou.com)


# 常用注解
## @TableName
表名注解，标识实体类对应的表，**实体类的类名（转成小写后）和数据库表名相同时**，可以不指定该注解。
- 描述：表名注解，标识实体类对应的表
- 使用位置：实体类
-@TableName("sys_user")
public class User {
    private Long id;
    private String name;
    private Integer age;
    private String email;
}
关于tablename中属性的说明后面补说明

## @TableId

- 描述：主键注解
- 使用位置：实体类主键字段
- @TableName("sys_user")
public class User {
    @TableId
    private Long id;
    private String name;
    private Integer age;
    private String email;
}

## @TableField

描述：字段注解（非主键
@TableName("sys_user")
public class User {
    @TableId
    private Long id;
    @TableField("nickname")
    private String name;
    private Integer age;
    private String email;
}

## @Version

- 描述：乐观锁注解、标记 ​`@Verison`​ 在字段上
## @EnumValue

- 描述：普通枚举类注解(注解在枚举字段上)
-
# - .CRUD 接口
mp封装了一些最基础的CRUD方法，只需要直接继承mp提供的接口，无需编写任何SQL，即可食用。mp提供了两套接口，分别是`Mapper CRUD`接口和`Service CRUD`接口。并且mp还提供了条件构造器`Wrapper`，可以方便地组装SQL语句中的WHERE条件。

  ##  Service CRUD 接口
  通用 Service CRUD 封装IService接口，进一步封装 CRUD 采用 `get 查询单行` `remove 删除` `list 查询集合` `page 分页`
  
  首先，新建一个接口，继承`IService`
  public interface UserService extends IService<User> {
}
​创建这个接口的实现类，并继承`ServiceImpl`
@Slf4j
@Service
public class UserServiceImpl extends ServiceImpl<UserDAO, User> implements UserService {
​
}


最后使用`userSerive`就可以调用相关方法:userService.

