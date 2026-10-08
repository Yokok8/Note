# JDBC —— 介绍

## 一、定义（课件原文）
**JDBC（Java DataBase Connectivity），就是使用 Java 语言操作关系型数据库的一套 API。**&#8203;

一句话：**用 Java 代码去操作数据库的标准方式。**&#8203;

## 二、本质（课件三条，核心中的核心）

| # | 原文 | 怎么理解 |
|---|---|---|
| 1 | **sun 公司官方定义的一套操作所有关系型数据库的规范，即接口** | JDBC **是接口，不是实现** —— 本身干不了活 |
| 2 | **各个数据库厂商去实现这套接口，提供数据库驱动 jar 包** | 真正干活的是**各厂商的驱动** |
| 3 | **我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动 jar 包中的实现类** | 你写 `Connection`，运行时跑的是 MySQL 驱动里的实现 |

⭐ **记这一句就够：JDBC 是"接口 + 规范"，驱动 jar 是"实现类"。**&#8203;

## 三、和你学过的知识串起来（重点）

| 之前学的                    | 在 JDBC 里的体现                                        |
| ----------------------- | -------------------------------------------------- |
| **接口 + 实现类**（IOC/DI 一节） | JDBC 是接口，驱动 jar 是实现类                               |
| **反射**（Class.forName）   | 加载驱动就是 `Class.forName("com.mysql.cj.jdbc.Driver")` |
| **多态**                  | 你声明 `Connection` 接口类型，实际对象是 MySQL 驱动的实现            |
| **Maven 依赖管理**          | 驱动就是一条 `<dependency>`                              |

**所以前面学的没白学 —— JDBC 就是这些基础知识的组合应用。**&#8203;

---

# JDBC —— 入门程序_增删数据

## 一、需求
基于 JDBC 程序，执行 update 语句：`update user set age = 25 where id = 1`
DML语句
## 二、准备工作
1. 创建一个 **Maven 项目**
2. **引入依赖**（mysql 驱动）
3. 准备好**数据库表 user**

    <dependency>
        <groupId>com.mysql</groupId>
        <artifactId>mysql-connector-j</artifactId>
        <version>8.0.33</version>
    </dependency>

## 三、五步完整代码（背下来，以后所有 JDBC 都是这个骨架）
```java
    public class JdbcDemo {
        public static void main(String[] args) throws Exception {
            // 1. 注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");

            // 2. 获取连接
            String url = "jdbc:mysql://localhost:3306/web01";
            String username = "root";
            String password = "1234";
            Connection connection = DriverManager.getConnection(url, username, password);

            // 3. 获取 SQL 语句执行对象
            Statement statement = connection.createStatement();

            // 4. 执行 SQL
            int i = statement.executeUpdate("update user set age = 25 where id = 1");

            // 5. 释放资源
            statement.close();
            connection.close();
        }
    }
```
## 四、逐行拆解

| 步 | 代码 | 干什么 |
|---|---|---|
| ① | `Class.forName("com.mysql.cj.jdbc.Driver")` | **用反射把驱动类加载进来**（驱动 jar 里注册自己的实现） |
| ② | `DriverManager.getConnection(url, user, pwd)` | 拿到与数据库的**连接对象** `Connection` |
| ③ | `connection.createStatement()` | 创建**SQL 执行对象**，SQL 通过它执行 |
| ④ | `statement.executeUpdate(sql)` | 执行**增删改** SQL，返回 **int = 受影响的行数** |
| ⑤ | `statement.close(); connection.close();` | 释放资源（**先开后关**） |

## 五、URL 怎么读（重点）

    jdbc:mysql://localhost:3306/web01
    └─┬─┘ └─┬─┘ └──┬──┘ └┬─┘ └─┬─┘
    协议   数据库  主机   端口  数据库名

| 部分 | 含义 | 可以省吗 |
|---|---|---|
| `jdbc:mysql:` | 用哪种数据库的驱动 | 不能 |
| `localhost` | 主机（也可以写 IP） | 不能 |
| `3306` | 端口（MySQL 默认 3306） | **本机默认端口可以省** |
| `web01` | **要连的数据库名** | 不能，省了会报 `No database selected` |

可以加参数（后面会用到）：

    jdbc:mysql://localhost:3306/web01?useSSL=false&serverTimezone=Asia/Shanghai&characterEncoding=utf8

## 六、易错点
1. **五个步骤的顺序不能乱** —— 必须先有连接，才能创建 statement
2. **`Class.forName` 抛的是 `ClassNotFoundException`**（编译时异常，必须处理）—— 这里直接 `throws Exception` 图省事
3. **驱动类名别写错**：MySQL 8 是 `com.mysql.cj.jdbc.Driver`（**有 `.cj`**）
4. **`executeUpdate` 只用于增删改** —— 返回值是"影响了几行"；**查询要用 `executeQuery`**（下一张讲）
5. **端口默认 3306 可以省略**，但写全更好读：`localhost:3306`
6. **释放资源必须写** —— 不关连接会一直占着数据库的连接数，时间一长数据库就拒绝新连接
7. **真实项目不会这么裸写** —— 后面会换成**连接池（Druid）**&#8203;，并在 `finally` 里关资源
8. 驱动注册其实**可以省略**（MySQL 5+ 驱动 jar 有 SPI 自动注册），但课件保留这一步是为了讲清原理

## 七、一句话记
**注册驱动 → 获取连接 → 获取执行对象 → 执行 SQL → 释放资源。**&#8203;
**`forName` 加载驱动，`getConnection` 拿连接，`createStatement` 拿执行对象，`executeUpdate` 执行增删改并返回影响行数，最后 `close`。**&#8203;

---

# JDBC —— 查询数据（ResultSet）

## 一、需求
基于 JDBC 执行 select 语句，把查询结果**封装到 User 对象**中。

    select * from user where username = 'daqiao' and password = '123456'

## 二、ResultSet（结果集对象）

    ResultSet rs = statement.executeQuery(sql);

**两个核心方法：**

### ① `next()`
**将光标从当前位置向前移动一行，并判断当前行是否为有效行**，返回 boolean。

| 返回值 | 含义 |
|---|---|
| `true` | 有效行，当前行有数据 |
| `false` | 无效行，当前行没有数据 |

### ② `getXxx(...)`
**获取数据**，可以根据**列的编号**获取，也可以根据**列名**获取 —— **推荐用列名**。

    int id = rs.getInt("id");              // 按列名（推荐）
    String name = rs.getString("name");
    int id2 = rs.getInt(1);                // 按列编号，从 1 开始

## 三、完整代码（查询 + 封装对象）
```java
    public class JdbcQueryDemo {
        public static void main(String[] args) throws Exception {
            // 1. 注册驱动
            Class.forName("com.mysql.cj.jdbc.Driver");

            // 2. 获取连接
            String url = "jdbc:mysql://localhost:3306/web01";
            Connection connection = DriverManager.getConnection(url, "root", "1234");

            // 3. 获取执行对象
            Statement statement = connection.createStatement();

            // 4. 执行查询（用 executeQuery）
            String sql = "select * from user where username = 'daqiao' and password = '123456'";
            ResultSet rs = statement.executeQuery(sql);

            // 5. 解析结果：光标逐行下移，取当前行数据
            User user = null;
            while (rs.next()) {
                user = new User();
                user.setId(rs.getInt("id"));
                user.setUsername(rs.getString("username"));
                user.setPassword(rs.getString("password"));
                user.setName(rs.getString("name"));
            }
            System.out.println(user);

            // 6. 释放资源（先开后关）
            rs.close();
            statement.close();
            connection.close();
        }
    }
```
## 四、`executeQuery` vs `executeUpdate`

| 方法              | 用于                                | 返回                 |
| --------------- | --------------------------------- | ------------------ |
| `executeQuery`  | **查询**（select）                    | **ResultSet**（结果集） |
| `executeUpdate` | **增删改**（insert / update / delete） | **int**（受影响的行数）    |

**记法：Query 有返回值（结果集），Update 只告诉你改了几行。**&#8203;

## 五、易错点

| # | 坑 |
|---|---|
| 1 | **必须先 `next()` 才能取值** —— 光标初始**不指向任何一行**（在表头上方），直接 `getXxx` 会报错 |
| 2 | **按列编号取值时从 1 开始，不是 0** —— `rs.getInt(0)` 直接抛异常 |
| 3 | **`getXxx` 的类型要和字段匹配** —— `getInt` / `getString` / `getDouble` / `getDate`，用错了可能报错或取到意外值 |
| 4 | **`User` 对象要在循环体内部 `new`** —— 写在外面所有行会共用同一个对象，最后只剩最后一条数据 |
| 5 | **列名指的是"结果集的列名"**&#8203; —— SQL 里用了别名（`select name as n`），就要用 `getString("n")` |
| 6 | **没有查询结果时循环体一次都不执行** —— `user` 保持 `null`，别忘了判空 |
| 7 | **`ResultSet` 也要关闭** —— 它是资源，别只关 statement 和 connection |
| 8 | **SQL 里拼接字符串有注入风险** —— `"...where username = '" + username + "'"` 是**不安全**写法，后面用 **PreparedStatement** 解决 |
| 9 | 推荐用**列名**而不是列编号 —— 列顺序变化时列名写法不受影响，代码可读性也更好 |

## 六、一句话记
**`executeQuery` 拿结果集，`next()` 让光标下移一行并判断有没有数据，`getXxx(列名)` 取当前行的值。**&#8203;
**标准写法：`while (rs.next()) { 每轮取一行的数据 }` —— 光标初始在表头上方，所以不先 `next()` 什么都取不到。**&#8203;

---

# JDBC —— 预编译 SQL（PreparedStatement）

## 一、两种写法对比（课件图 1）

| | ① 静态 SQL（参数硬编码） | ② 预编译 SQL（参数动态传递） |
|---|---|---|
| 用谁 | `Statement` | **`PreparedStatement`** |
| 创建 | `connection.createStatement()` | `conn.prepareStatement(sql)` ← **要把 SQL 传进去** |
| 参数 | 拼进字符串，写死 | SQL 里用 **`?`** 占位，后面再传 |
| 怎么赋参 | 拼接字符串 | `pstmt.setString(1, "daqiao")` |
| 怎么执行 | `executeUpdate(带参SQL)` | `executeUpdate()` / `executeQuery()` **不带参** |
| 安全 | 有注入风险 | **安全** |
| 性能 | 每次都要解析编译 | **更高**（编译一次，缓存复用） |

## 二、代码对照

**静态 SQL（硬编码）：**
```java
    Statement statement = connection.createStatement();
    int i = statement.executeUpdate("update user set age = 25 where id = 1");
    System.out.println("SQL执行完毕，影响的记录数为: " + i);
```
**预编译 SQL（参数动态传递）：**
```java
    PreparedStatement pstmt = conn.prepareStatement(
            "SELECT * FROM user WHERE username = ? AND password = ?");
    pstmt.setString(1, "daqiao");
    pstmt.setString(2, "123456");
    ResultSet resultSet = pstmt.executeQuery();
```
**三条变化要看清：**
1. 对象从 `Statement` 换成 **`PreparedStatement`**
2. SQL 在 **`prepareStatement(sql)` 时就传进去**（不是执行时才传）
3. 参数用 **`setXxx(序号, 值)`** 一个个填，执行方法**不再带 SQL**

## 三、优势一：防止 SQL 注入，更安全

**定义（课件原文）：**
SQL 注入 = 通过控制输入来修改事先定义好的 SQL 语句，以达到执行代码对服务器进行攻击的方法。

**攻击怎么发生的（看上面那张图）：**

| 输入 | 拼出来的 SQL | 结果 |
|---|---|---|
| 密码填 `123456` | `... username='daqiao' and password='123456'` | 密码对才通过 |
| 密码填 `' or '1'='1` | `... username='daqiao' and password=''`**` or '1'='1'`** | **`'1'='1'` 恒真 → 全部命中 → 登录成功** |

⚠️ 关键就是：**用户的输入被当成了 SQL 语法的一部分**，一个引号就能把判断条件"切开"，自己接上新的条件。
（更狠的输入能直接 `drop table`，把整张表删掉。）

**预编译为什么能防住？**
- SQL 在 `prepareStatement()` 时**就已经编译好了**，结构被**固定死**
- 后面 `setString()` 传进去的值，**只会被当成"值"**&#8203;，不会参与语法解析
- 所以哪怕你输入 `' or '1'='1`，它也只是"一个密码字符串"，改不了 SQL 结构

**一句话：静态 SQL 是"先把你的话拼进我的句子里"，预编译是"我的句子先定好，你的话只能填进空格里"。**&#8203;

## 四、优势二：性能更高

**静态 SQL —— 3 条 SQL 走 3 遍流程：**

    delete from user where id = 1;   ┐
    delete from user where id = 2;   ├─ 3 次 ─→ 解析检查 → 优化 → 编译 → 执行
    delete from user where id = 3;   ┘

**预编译 SQL —— 1 条 SQL 只编译 1 次：**

    delete from user where id = ?;  ─ 1 次 ─→ 解析检查 → 优化 → 编译 → 【缓存】
    （参数：1 / 2 / 3 分别传）                        ↓ 后续直接复用
                                                    执行

MySQL 端流程：**SQL 语法解析检查 → 优化 SQL → 编译 SQL →（缓存）→ 执行 SQL**

**原因**：带 `?` 的 SQL 只有**一个"模板"**&#8203;，MySQL 只需编译一次，结果留在缓存里；后续换参数直接复用，**跳过了前面三步**。
参数值变了，SQL 结构没变 —— **变的东西不参与编译，所以能复用。**&#8203;

## 五、`setXxx` 有哪些（眼熟即可）

| 方法 | 对应类型 |
|---|---|
| `setString(序号, 值)` | varchar / char / text |
| `setInt(序号, 值)` | int |
| `setDouble(序号, 值)` | double / decimal |
| `setDate(序号, 值)` | date |
| `setObject(序号, 值)` | 通用（不想挑类型时用） |

## 六、易错点

| # | 坑 |
|---|---|
| 1 | **`?` 的序号从 1 开始**，不是 0 —— `setString(0, ...)` 直接报错 |
| 2 | **`?` 一定要用英文半角**，别写成中文全角 `？` |
| 3 | **`?` 不要加引号** —— 写 `'?'` 会被当成字符串字面量，不再是占位符 |
| 4 | **每个 `?` 都要赋值**，少赋一个执行时报错 |
| 5 | **执行方法别再传 SQL** —— `pstmt.executeQuery()` 空参；再传 SQL 就是方法签名不对 |
| 6 | **`?` 只能占"值"的位置** —— 表名、字段名、`order by` 的字段**不能用 `?`**，只能拼接，且**必须做白名单校验** |
| 7 | **`PreparedStatement` 是 `Statement` 的子接口** —— 所以 `Statement` 的用法它都有，只是更安全 |
| 8 | **`PreparedStatement`、`ResultSet`、`Connection` 都要 `close`**（先开后关） |
| 9 | 它防注入靠的是**预编译 + 参数当纯数据**，不是靠"过滤特殊字符"——别想着自己写替换逻辑来防注入 |
| 10 | 连接池（Druid）环境下，预编译还能让**同一个连接复用已编译的语句**，优势更明显 |

## 七、一句话记
**静态 SQL：`createStatement()` + 参数拼进字符串 → 每次重新编译，还能被注入。**&#8203;
**预编译 SQL：`prepareStatement(带 ? 的SQL)` + `setXxx(序号, 值)` → 结构先定死、编译一次缓存复用。**&#8203;
**安全靠"你的输入进不了语法层"，快靠"模板只编译一次"。**&#8203;
