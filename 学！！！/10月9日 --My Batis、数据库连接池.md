# 什么是 MyBatis

## 一句话定义（背这句）
**MyBatis 是一款优秀的持久层框架，用于简化 JDBC 的开发。**&#8203;

两个关键词：
- **持久层** → 它管的是 **Dao 层**（controller → service → **dao** → 数据库）
- **简化 JDBC** → 前面手写的五步套路（注册驱动、拿连接、建 Statement、执行、关资源），它全替你包了

## 历史沿革（眼熟即可）
| 时间 | 事件 |
|---|---|
| 最早 | Apache 开源项目 **iBatis** |
| 2010 年 | 从 apache 迁移到 google code，**改名为 MyBatis** |
| 2013 年 11 月 | 迁移到 **Github** |

## 官网
https://mybatis.org/mybatis-3/zh_CN/index.html

## 对照你刚学的东西（重要）
| 之前手写 JDBC | MyBatis 里 |
|---|---|
| 自己拼 SQL、自己拿 Connection | 交给 MyBatis |
| 自己 `rs.getXxx()` 一列列取 | **自动映射到对象** |
| 担心 SQL 注入 | 底层用的就是 **PreparedStatement** |

**所以 MyBatis 不是新东西，是把 JDBC 那套"自动化"了。**&#8203;

## 一句话记
**MyBatis = 持久层框架，专门简化 JDBC；只管 Dao 层，SQL 还是你写，其余它包。**&#8203;

---

# MyBatis 入门 —— 查询所有用户数据

## 一、四步总览

| 阶段      | 步   | 做什么                                             |
| ------- | --- | ----------------------------------------------- |
| **准备**  | ①   | 创建 **SpringBoot 工程**、引入 MyBatis 相关依赖            |
| **准备**  | ②   | 准备数据库表 **user**、实体类 **User**                    |
| **准备**  | ③   | **配置 MyBatis**（application.properties 中数据库连接信息） |
| **写代码** | ④   | 编写 **持久层接口**，定义 SQL（**注解 / XML**）               |

前 3 步是搭环境，真正要写的只有第 ④ 步。

## 二、依赖（建工程时勾选）

| 依赖 | 作用 |
|---|---|
| **MyBatis Framework** | MyBatis 起步依赖 |
| **MySQL Driver** | 数据库驱动（**别忘了勾，漏了连不上库**） |

（Lombok 可选，用来省 getter/setter）

## 三、配置 application.properties

    spring.datasource.url=jdbc:mysql://localhost:3306/web
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.username=root
    spring.datasource.password=1234

**四个 key 和 JDBC 五步里的"获取连接"一一对应：**

| 配置项 | 对应 JDBC 的 |
|---|---|
| `spring.datasource.url` | `getConnection(url, ...)` |
| `spring.datasource.username` | 用户名 |
| `spring.datasource.password` | 密码 |
| `spring.datasource.driver-class-name` | `Class.forName("...Driver")` |

⚠ 注意：`driver-class-name` 是 `com.mysql.cj.jdbc.Driver`（**有 `.cj`**）。

## 四、实体类 User

    public class User {
        private Integer id;         // ID
        private String username;    // 用户名
        private String password;    // 密码
        private String name;        // 姓名
        private Integer age;        // 年龄
        // 省略 getter / setter / toString
    }

**字段名要和数据库表的字段名对得上**（MyBatis 靠名字自动映射）。

## 五、Mapper 接口（核心，第 ④ 步）

    @Mapper
    public interface UserMapper {
        @Select("select * from user")
        public List<User> findAll();
    }

| 注解 | 作用 |
|---|---|
| **`@Mapper`** | 告诉 MyBatis「**这是持久层接口**」，启动时给它**生成实现类**并注入容器 |
| **`@Select("SQL")`** | 把 SQL 直接写在注解里（另一种方式：写在 **XML** 里） |

**注意：接口里只有方法声明，没有方法体 —— SQL 是通过注解告诉它的。**&#8203;

## 六、为什么接口没有实现类也能调用？（重点理解）

对照上面那张图：

| 步 | 谁做的 |
|---|---|
| ① | **你**：只写接口 + `@Select` 注解 |
| ② | **MyBatis**：启动时扫描 `@Mapper`，用**动态代理**生成实现类，放进 IOC 容器 |
| ③ | **MyBatis**：读 `application.properties`，用数据源（连接池）连数据库 |
| ④ | **MyBatis**：执行注解里那段 SQL，把每一行**自动封装成 `User`** |

所以你在 Service 里 `@Autowired private UserMapper userMapper;` 拿到的，**其实是 MyBatis 生成的代理对象**。
调 `userMapper.findAll()` 时，代理替你：执行 SQL → 遍历结果集 → `new User()` → setter 赋值 → 装进 List。

👉 **这正是反射 / 动态代理的实战应用** —— 你前面学反射没白学。

## 七、课件提示（底部原文）
**MyBatis 的持久层接口命名规范为 `XxxMapper`，也称为 Mapper 接口。**&#8203;

（所以叫 `UserMapper`、`DeptMapper`、`EmpMapper`，不要叫 `UserDaoImpl` 那种老名字）

## 八、易错点

| #   | 坑                                                                                    |
| --- | ------------------------------------------------------------------------------------ |
| 1   | **忘加 `@Mapper`** → 启动报「找不到 Bean」，这是最常见的坑                                             |
| 2   | **只引了 MyBatis 起步依赖，没引 MySQL 驱动** → 启动报数据源错误                                          |
| 3   | **`driver-class-name` 写成 `com.mysql.jdbc.Driver`（5.x 老名字）**&#8203; → 报找不到驱动类         |
| 4   | **配置的库名 / 端口 / 密码和本地不一致** → 连接失败（图里库名是 `web`，不是 `web01`）                             |
| 5   | **实体类字段名和表字段名对不上** → 该字段取到 `null`（表用下划线 `user_name`、Java 用驼峰 `userName` 时要开**驼峰映射**） |
| 6   | **实体类必须有 getter/setter** → MyBatis 靠 setter 赋值，只写 private 字段会全是 null                 |
| 7   | 表名 / 字段名是关键字时，SQL 里要用反引号包起来                                                          |
| 8   | `@Select` 写在方法上，**一个方法对应一条 SQL**；SQL 复杂或多变时改用 XML                                    |

## 九、一句话记
**建工程勾依赖 → 备表和实体类 → 配置里写四项数据源 → 写 `@Mapper` 接口 + `@Select` SQL。**&#8203;
**接口没实现类也能调，因为 MyBatis 用动态代理替你在运行时生成了实现。**&#8203;

---
# JDBC vs MyBatis（对比总结）

## 一、三个痛点 → 三个解法（背这张）

| JDBC 的问题 | 具体表现 | MyBatis 的解法 |
|---|---|---|
| ① **硬编码** | 驱动类名、URL、用户名、密码全写在 **Java 代码里** | 抽到 **application.properties** 配置文件 |
| ② **繁琐** | 手动遍历 `ResultSet`、一列列 `getXxx`、手动 `new User`、再 `add` 进 List | 一个接口 + `@Select`，**自动映射**成对象 |
| ③ **资源浪费、性能降低** | 每次操作都要**新建连接**，用完就关 | 底层用**数据库连接池**，连接复用、用完归还 |

## 二、代码量对比（一眼看出差距）

**JDBC —— 要写三段：**
```java
    // 1. 准备工作（硬编码）
    Class.forName("com.mysql.cj.jdbc.Driver");
    String url = "jdbc:mysql://localhost:3306/web";
    Connection connection = DriverManager.getConnection(url, "root", "1234");
    PreparedStatement ps = connection.prepareStatement("select * from user");

    // 2. 执行 SQL，处理结果（繁琐）
    List<User> userList = new ArrayList<>();
    ResultSet resultSet = ps.executeQuery();
    while (resultSet.next()) {
        int id = resultSet.getInt("id");
        String username = resultSet.getString("username");
        String name = resultSet.getString("name");
        String password = resultSet.getString("password");
        int age = resultSet.getInt("age");
        User user = new User(id, username, password, name, age);
        userList.add(user);
    }

    // 3. 释放资源
    resultSet.close();
    ps.close();
    connection.close();
```
**MyBatis —— 配置 + 接口：**
```java
    # application.properties
    spring.datasource.url=jdbc:mysql://localhost:3306/web
    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.username=root
    spring.datasource.password=1234

    @Mapper
    public interface UserMapper {
        @Select("select * from user")
        public List<User> findAll();
    }
```
**上面那一大段 JDBC 代码，被压缩成了 3 行接口。**&#8203;

## 三、硬编码到底坏在哪

| | 硬编码（JDBC） | 配置外置（MyBatis） |
|---|---|---|
| 改数据库地址 | **改 Java 代码 → 重新编译打包上线** | **改一行配置 → 重启即可** |
| 密码泄露风险 | 密码进源码、进 git 仓库 | 配置可单独管理、可加密 |

## 四、连接池为什么能提升性能

| | 每次新建连接（JDBC） | 连接池（MyBatis 底层） |
|---|---|---|
| 建连接的开销 | **TCP 三次握手 + 身份认证**，一次几十毫秒 | 启动时先建好一批，**用完还回池子** |
| 连接数控制 | 无法控制，并发一高可能把库打崩 | 池子有上限，**保护数据库** |
| 效果 | 并发上来就卡 | 响应快、稳定 |

连接池里的那些"连接"，就是你之前学的 TCP 连接 —— **建一次很贵，所以要复用。**&#8203;

## 五、三个容易搞错的认知

| # | 说明 |
|---|---|
| 1 | **MyBatis 底层还是 JDBC**（还是 `PreparedStatement`、还是 `ResultSet`）——是**封装关系，不是替代关系** |
| 2 | **连接池不是 MyBatis 提供的**，是 Spring Boot 自动配的（默认 **HikariCP**，项目里常换成 **Druid**） |
| 3 | 用了 MyBatis **不代表 SQL 就高效** —— 该优化 SQL 还得优化（比如别滥用 `select *`） |

## 六、易错点
1. 别以为"用了 MyBatis 就不用懂 JDBC" —— 面试和排错都靠底层原理（连接池打满、SQL 慢查询，都要回到 JDBC 层面看）
2. **实体类必须有 getter/setter** —— MyBatis 靠 **setter 赋值**，只有 private 字段会全是 `null`
3. 自动映射靠**字段名对上**，表字段是 `user_name`、Java 是 `userName` 时要开**驼峰映射**（`map-underscore-to-camel-case`）
4. `@Select` 里的 SQL **是真正的 SQL**，仍要注意注入风险 —— 用 `#{}`（预编译）而不是 `${}`（字符串拼接）
5. 配置文件里的 `driver-class-name` 别写成 5.x 的老名字 `com.mysql.jdbc.Driver`

## 七、一句话记
**JDBC 是手动挡，MyBatis 是自动挡 —— 发动机（JDBC）没换，硬编码、繁琐代码、连接管理它全替你干了。**&#8203;
**三大改进对应三件事：配置搬出去、映射自动化、连接池复用。**&#8203;

---

# 数据库连接池

## 一、定义（课件原文，背这句）
**数据库连接池是个容器，负责分配、管理数据库连接（Connection）。**&#8203;

三个关键词：**容器**、**分配**、**管理**。
它装在应用和数据库之间，应用不直接建连接，而是找它要。

## 二、三条核心说明（课件原文）
1. 它允许应用程序**重复使用一个现有的数据库连接**，而不是**再重新建立一个**
2. **释放空闲时间超过最大空闲时间的连接**，来避免因为没有释放连接而引起的**数据库连接遗漏**
3. **优势：资源重用 / 提升系统响应速度 / 避免数据库连接遗漏**

## 三、为什么需要它（回到"建连接有多贵"）

| 建一次连接的代价 | 说明 |
|---|---|
| TCP 三次握手 | 前面网络编程学过，来回一轮 |
| 身份认证 | 校验用户名密码 |
| 数据库侧分配资源 | 开线程、占内存 |

**一次几十毫秒**。如果每个请求都新建一次，并发一上来就被这个开销拖垮 —— 而真正执行 SQL 可能只要 1 毫秒。

## 四、工作方式（对照上面那张图）

| 步 | 发生什么 |
|---|---|
| ① | 应用需要连接 → 从池子里**借**一个（不新建） |
| ② | 用完 → **归还**给池子，**连接不销毁**，留给下一个请求 |
| ③ | 某个连接**空闲太久** → 超过"最大空闲时间" → 池子**回收释放**掉 |

**核心就一句话：借出去、还回来、循环用；太久不用就回收。**&#8203;

## 五、三大优势逐条解读

| 优势 | 为什么能做到 |
|---|---|
| **资源重用** | 连接不销毁，同一批连接被反复使用 |
| **提升系统响应速度** | 省掉了建连接的那几十毫秒，直接拿来就能用 |
| **避免数据库连接遗漏** | 空闲超时自动回收 —— **即使你代码忘了 close，池子也会兜底收回来** |

## 六、"最大空闲时间"是什么

- 池子里的连接如果**长时间没人借**，占着也没意义 → 超过设定时间就**释放**
- 为什么要有它？因为**连接不是越多越好** —— 每个连接在数据库那侧都占着线程和内存
- 它是**自动兜底**：代码忘了 `close()`，不至于把连接永久漏掉

## 七、常见连接池（眼熟）

| 连接池 | 特点 |
|---|---|
| **HikariCP** | **Spring Boot 2.x+ 默认**，性能最好 |
| **Druid** | 阿里出品，自带**监控页面**，国内项目常用 |
| C3P0 / DBCP | 老牌，现在基本被前两个取代 |

## 八、易错点

| # | 坑 |
|---|---|
| 1 |  **“归还" ≠ "关闭”** &#8203; —— 你调 `connection.close()` 时，池子包装过的连接其实是**还回池子**，不是真断掉 |
| 2 | **用了连接池也必须 `close()`** —— 不归还就会慢慢耗光池子，之后所有请求**排队等待甚至超时**，这是线上常见故障 |
| 3 | **连接池不是无限的** —— 有上限，借完就得等（或报 `Timeout waiting for connection`） |
| 4 | **连接不是越多越好** —— 每个连接都占数据库资源，配太大反而更慢 |
| 5 | **连接池本身不产生连接** —— 它只是容器，连接最终还是通过 **JDBC 驱动**建立（回到前面学的五步） |
| 6 | 多数据源 / 多线程环境下，连接**不能跨线程用**，借了必须在同一次操作里归还 |
| 7 | 池子大小要配合数据库的 `max_connections` —— 池子配 100 但数据库只允许 50，照样连不上 |

## 九、一句话记
**连接池 = 连接的中转仓。借出去、还回来、不销毁、可复用；太久没人用就自动回收。**&#8203;
**三大好处：省资源（重用）、快（免建连接）、不漏（超时兜底）。**&#8203;
**它管的还是 JDBC 的 `Connection` —— 底层没换，只是把"建连接"这件贵事变成了一次性投入。**&#8203;
