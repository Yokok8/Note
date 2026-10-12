# MyBatis —— 删除用户（delete）+ #{} 与 ${}

## 一、@Delete 的三种写法

需求：**根据 ID 删除用户信息**
SQL：`delete from user where id = 5;`

| # | 写法 | 问题 |
|---|---|---|
| ✗ | `@Delete("delete from user where id = 5")` | **id 硬编码**在 SQL 里，写死了只能删 5 号 |
| ? | `@Delete("delete from user where id = #{id}")`<br>`public void deleteById(Integer id);` | 参数化了，但**返回值是 void** —— 拿不到"删了几行" |
| ✓ | `@Delete("delete from user where id = #{id}")`<br>`public Integer deleteById(Integer id);` | 正解：**参数化 + 返回影响行数** |

**三条演进一句话：**
先别把值写死（改成 `#{}`）→ 再把返回值定下来（改成 `Integer`）。

## 二、注意
**DML 语句执行完毕的返回值，表示该 DML 语句执行完毕影响的行数。**&#8203;

| 返回值类型 | 你能得到什么 | 场景 |
|---|---|---|
| `void` | 什么都没有 | 只管删，不关心结果 |
| `Integer` | **影响行数** | **返回 0 说明这个 id 根本不存在**（能区分"删成功"和"没删到"） |

所以删除接口一般写成 `Integer` 或 `int`，前端好判断"删除失败"还是"数据不存在"。

## 三、⭐ `#{}` 与 `${}` 对比

| 符号 | 说明 | 场景 | 优缺点 |
|---|---|---|---|
| **`#{}`** | **占位符**。执行时会将 `#{}` 替换为 **`?`**，生成**预编译 SQL** | **参数值传递** | **安全、性能高（推荐）**&#8203; |
| **`${}`** | **拼接符**。直接将参数**拼接在 SQL 语句**中，存在 SQL 注入问题 | **表名、字段名**动态设置时使用 | **不安全、性能低** |

**课件两个代码示例：**

    @Delete("delete from dept where id = #{id}")                                  -- 传值 → #
    @Select("select id,name,score from ${tableName} order by ${sortField}")      -- 传表名/字段名 → $

## 四、为什么一个安全一个不安全（本质）

**这就是前面"预编译 SQL vs 静态 SQL"的翻版，只是换成了 MyBatis 写法：**

| | `#{}` | `${}` |
|---|---|---|
| MyBatis 做什么 | 替换成 **`?`** | **直接拼字符串** |
| 底层对应 | **PreparedStatement** | **Statement** |
| 参数身份 | 只是个**数据**，不参与语法解析 | 成了 **SQL 语法的一部分** |
| SQL 注入 | **防得住** | **防不住** |
| 性能 | 预编译结果可复用 | 每条 SQL 都当新语句编译 |

**一句话：`#{}` 是"把值填进空格"，`${}` 是"把你的话拼进我的句子"。**&#8203;

## 五、既然 `${}` 危险，为什么还留着

因为**表名、字段名、`order by` 后面的排序字段，根本没法参数化**。
`select * from ?` 这种写法 MySQL 直接不认 —— 这些位置**只能拼接**。

⚠ 所以用 `${}` 时**必须自己做白名单校验**：

    // 思路示意：只允许前端传约定的几个字段名，其余一律拒绝
    if (!List.of("id", "name", "score").contains(sortField)) {
        throw new IllegalArgumentException("非法排序字段");
    }

**绝不能把前端传来的字符串直接塞进 `${}`。**&#8203;

## 六、易错点

| # | 坑 |
|---|---|
| 1 | **`#{}` 里的名字写错** → 报参数找不到（多参数时尤其容易，后面要靠 `@Param`） |
| 2 | **`${}` 拿来传"值"**&#8203; → 等于主动送注入漏洞，这是最严重的一条 |
| 3 | **`${}` 传表名/字段名也必须校验** → 否则前端能传 `user; drop table ...` 之类 |
| 4 | **`delete` 忘写 `where`** → 整表数据清空（老坑，写 delete 前先 `select` 确认） |
| 5 | **返回 `0` 不等于 `null`** → `Integer` 返回 0 表示"影响 0 行"（id 不存在），不是没返回值 |
| 6 | **`@Delete` / `@Insert` / `@Update` / `@Select` 别用混** → 对应 DML 和 DQL，用错注解语义不清 |
| 7 | 单参数时 `#{id}` 里名字**可以随便写**（MyBatis 直接取值）；但**建议和方法参数名保持一致**，多参数时就必须靠 `@Param` 指定 |
| 8 | `#{}` 会自动处理类型（字符串自动加引号），**你自己在 SQL 里再套引号反而出错** |

## 七、一句话记
**`#{}` 替换成 `?` → 预编译 → 安全又快 → 传"值"用它。**&#8203;
**`${}` 直接拼字符串 → 有注入风险 → 只能拼"表名 / 字段名"，且必须白名单校验。**&#8203;
**delete 的返回值是影响行数，返回 0 说明没删到。**&#8203;


---

# MyBatis —— 新增用户（insert）

## 一、需求 + SQL
**需求：添加一个用户**

    insert into user(username,password,name,age) values('zhouyu','123456','周瑜',20);

## 二、两种写法

**✗ 硬编码 —— 值全写死在 SQL 里：**

    @Insert("insert into user(username,password,name,age) values('zhouyu', '123456', '周瑜', 20)")
    public void insert();

**✓ 参数化 —— 用一个对象接收所有值：**

    @Insert("insert into user(username,password,name,age) values(#{username},#{password},#{name},#{age})")
    public void insert(User user);

**三处变化：**

| 变化点 | 之前 | 现在 |
|---|---|---|
| values 里的值 | 字面量 `'zhouyu'` | **`#{属性名}`** |
| 方法参数 | 空 | **`User user`** |
| 参数个数 | 相当于 4 个值写死 | **一个对象全装下** |

## 三、⭐ 参数是对象时，`#{}` 里写什么（本页核心）

**写"对象的属性名"**&#8203; —— 课件就是这么批注的。

    public void insert(User user);
                       ↑ 参数是对象
    ... values(#{username},#{password},#{name},#{age})
                       ↑ 这里填的是 User 的属性名

**别和上一页搞混：**

| | delete（参数是单个值） | insert（参数是对象） |
|---|---|---|
| 方法签名 | `deleteById(Integer id)` | `insert(User user)` |
| `#{}` 里写的 | **参数名** `#{id}` | **对象属性名** `#{username}` |

## 四、为什么能取到值（原理）

MyBatis 拿到 `User user` 这个对象后：

1. 看到 `#{username}` → 去找这个对象的 **`getUsername()`** 方法
2. 调用它拿到值 → 填进 SQL 的 `?` 位置
3. 四个 `#{}` → 调四次 getter

👉 **又是反射 + getter**。所以：
- **实体类里必须有 getter 方法**（只写 private 字段会取到 `null`）
- **`#{}` 里的名字必须和字段名完全一致**（大小写敏感）

## 五、用对象传参的好处

如果不封装对象，就得这样写：

    @Insert("insert into user(username,password,name,age) values(#{username},#{password},#{name},#{age})")
    public void insert(@Param("username") String username, @Param("password") String password,
                       @Param("name") String name, @Param("age") Integer age);

**对比一下：**

| | 零散参数 | 对象参数 |
|---|---|---|
| 参数个数 | 4 个 | **1 个** |
| 多参数要不要 `@Param` | **要**（否则 `#{}` 取不到） | **不用** |
| 以后加字段 | 改方法签名 | **只改实体类** |
| 可读性 | 一长串 | 清爽 |

**所以能封装成对象就封装成对象** —— 这也是为什么前面要先建 `User` 实体类。

## 六、易错点

| # | 坑 |
|---|---|
| 1 | **`#{}` 里写成了表字段名** → 如果 Java 属性叫 `userName`、表字段叫 `user_name`，那 `#{}` 里要写 **Java 属性名** `userName` |
| 2 | **属性名大小写写错** → `#{UserName}` 找不到 getter，报错 |
| 3 | **实体类忘了写 getter** → 不报错但存进去全是 `null`（很隐蔽） |
| 4 | **字段列表和 values 个数对不上** → 直接 SQL 语法错误 |
| 5 | **不要手写引号**：写成 `values('#{username}')` 是错的 —— `#{}` 会自动处理类型和引号 |
| 6 | **insert 不用写 id** —— 有 `auto_increment`，数据库自己生成 |
| 7 | **有非空约束的字段必须有值** —— 对象里对应属性为 null 就会报错 |
| 8 | **insert 之后想拿到自增 id** → 光这样写拿不到，要用 `@Options(useGeneratedKeys = true, keyProperty = "id")`（下一张） |
| 9 | 方法名建议见名知意（`insert` / `add` / `save`），团队里统一 |

## 七、一句话记
**`@Insert` + `#{属性名}` + 一个 `User` 对象 = 搞定新增。**&#8203;
**`#{}` 里写的是"对象属性名"，MyBatis 靠 getter 反射取值 —— 所以实体类必须有 getter、名字必须一模一样。**&#8203;
**能封装对象就封装对象：参数少、不用 `@Param`、加字段不用改签名。**&#8203;

---

# MyBatis —— 修改用户（update）

## 一、需求 + SQL（课件原文）
**需求：根据 ID 更新用户信息**

    update user set username = 'zhouyu', password = '123456', name = '周瑜', age = 20 where id = 1;

## 二、两种写法（看演进）

**✗ 硬编码 —— 值全写死：**

    @Update("update user set username = 'zhouyu', password = '123456', name = '周瑜', age = 20 where id = 1")
    public void update();

**✓ 参数化 —— 一个对象全装下：**

    @Update("update user set username=#{username}, password=#{password}, name=#{name}, age=#{age} where id=#{id}")
    public void update(User user);

**三处变化和 insert 一模一样：**

| 变化点 | 之前 | 现在 |
|---|---|---|
| set 里的值 | 字面量 `'zhouyu'` | **`#{属性名}`** |
| where 条件 | 写死 `id = 1` | **`where id=#{id}`** |
| 方法参数 | 空 | **`User user`** |

## 三、⭐ update vs insert（本页真正要记的）

| | insert 新增 | update 修改 |
|---|---|---|
| id 字段 | **不用写**（数据库自增） | **必须写**（靠它定位改哪行） |
| where | **没有** | **必须有** |
| 改动结果 | **多出一行** | **改已有的行** |
| `#{}` 里写什么 | 对象属性名 | 对象属性名（**同一套规则**） |
| 忘写 where 的后果 | 不存在这个问题 | **全表被改成同一个值** |

**一句话：insert 是"往表里塞一行"，update 是"找到那一行再改"—— 找，靠的就是 where 里的 id。**&#8203;

## 四、`#{}` 里为什么写 `#{username}` 而不是别的东西

MyBatis 拿到 `User user` 对象后，看到 `#{username}`：
1. 去找对象的 **`getUsername()`** 方法 → 调它拿值
2. 值填进 SQL 的 `?` 位置
3. 五个 `#{}` → 调五次 getter

所以：
- **实体类必须有 getter**（只写 private 字段会全变成 `null`）
- **`#{}` 里写 Java 属性名**，不是表字段名（Java 叫 `userName`、表叫 `user_name` 时写 `userName`）

## 五、返回值：void 还是 Integer

课件写的是 `void`，但实际项目里建议改成 **`Integer`**：

| 返回值 | 你能得到什么 |
|---|---|
| `void` | 什么都拿不到 |
| `Integer` | **影响行数** —— 返回 **0 说明这个 id 根本不存在** |

    @Update("update user set username=#{username}, password=#{password}, name=#{name}, age=#{age} where id=#{id}")
    public Integer update(User user);

（DML 语句的返回值 = 影响的行数，这是上一张讲过的规则，insert / update / delete 通用。）

## 六、易错点

| # | 坑 |
|---|---|
| 1 | **忘写 where** → 全表数据被改成同一份（SQL 层的老坑，MyBatis 里同样致命） |
| 2 | **`#{}` 名字和属性名大小写不一致** → 找不到 getter 直接报错 |
| 3 | **实体类没写 getter** → 不报错但全更新成 `null`（最隐蔽的一种） |
| 4 | **手写引号** `set name = '#{name}'` 是错的 —— `#{}` 会自动处理类型和引号 |
| 5 | **改 id 本身要谨慎** —— 它可能被别的表引用着 |
| 6 | **只想改一个字段，却传了整个对象** —— 其他属性为 `null` 就会把那些字段一起冲掉；要么先查出完整对象再改，要么用动态 SQL（`<if>` 标签，后面会学） |
| 7 | **方法参数只有一个 `User` 对象时不用 `@Param`** —— 多个参数时才需要 |
| 8 | **表字段是关键字**时要加反引号：`` update `user` set ... `` |

## 七、一句话记
**`@Update` + `set 字段=#{属性名} ... where id=#{id}` + 一个 `User` 对象 = 搞定修改。**&#8203;
**insert 不用 id 不用 where（多一行），update 必须有 where（改已有的行）—— 忘写 where，全表遭殃。**&#8203;

---

# MyBatis —— 查询用户（select + @Param）

## 一、需求 + SQL（课件原文）
**需求：根据用户名和密码查询用户信息**

    select * from user where username = 'zhouyu' and password = '666888';

⚠ 这是**按两个条件查**，所以方法需要**两个参数** —— `@Param` 就是为这个场景准备的。

## 二、标准写法（带 @Param）

    @Select("select * from user where username=#{username} and password=#{password}")
    public User findByUsernameAndPassword(@Param("username") String username,
                                          @Param("password") String password);

**三个细节：**

| 细节 | 说明 |
|---|---|
| 返回类型 `User` | 查的是**一条**记录 → 返回单个对象（不是 `List<User>`）；**查不到返回 `null`** |
| 方法名 `findByUsernameAndPassword` | 见名知意，`findBy` + 条件，团队的通用命名法 |
| `@Param("username")` | 给**形参起名字**，`#{}` 里写的就这个名字 |

## 三、⭐ @Param 的作用（课件原文一句话）
**`@Param` 注解的作用是为接口的方法形参起名字的。**&#8203;

### 为什么需要起名字？

Java 编译成 `.class` 后，**方法的形参名默认会丢失**（变成无意义的 `arg0`、`arg1`）。
MyBatis 拿不到 `username` 这个名字，只能拿到：

| 它默认给的备选名 |
|---|
| `#{arg0}` / `#{arg1}`（按位置） |
| `#{param1}` / `#{param2}`（按编号） |

所以你不写 `@Param`，`#{username}` 它就找不到 → **报参数找不到的错误**。

👉 **一句话：`@Param` 是给 MyBatis 递一张"名字对照表"，告诉它 `#{}` 里的名字对应第几个参数。**&#8203;

## 四、课件的"说明"（可以省略的情况）

> 基于官方骨架创建的 Spring Boot 项目中，接口编译时会保留方法形参名，**@Param 注解可以省略（`#{形参名}`）**&#8203;。

    @Select("select * from user where username=#{uname} and password=#{pwd}")
    public User findByUsernameAndPassword(String uname, String pwd);

原因是 **Spring Boot 官方骨架（`spring-boot-starter-parent`）在编译插件里开了 `-parameters` 参数**，字节码里保留了真实的形参名，所以 MyBatis 能直接读到 `uname` / `pwd`。

### 两种写法对比

| | 写 `@Param` | 不写 `@Param` |
|---|---|---|
| `#{}` 里写 | `@Param` 里的名字（如 `username`） | **形参名**（如 `uname`） |
| 依赖环境 | **不依赖**，任何项目都能跑 | 依赖编译时保留形参名（`-parameters`） |
| 稳定性 | **最稳** | 换个构建配置（如 Lombok 编译、旧版插件）就可能失效 |

**建议：多个参数时老老实实写 `@Param`** —— 注释里的名字和 `#{}` 里的名字一致，读代码也不用回头对形参。

## 五、参数个数 → `#{}` 里怎么写（综合前面几张）

| 方法参数 | `#{}` 里写什么 | 例子 |
|---|---|---|
| **单个简单类型** | **随便写**（MyBatis 不挑名字） | `deleteById(Integer id)` → `#{id}` 或 `#{aaa}` 都行 |
| **单个对象** | **对象属性名** | `insert(User user)` → `#{username}` |
| **多个参数** | **`@Param` 里的名字**（或形参名） | `#{username}`、`#{password}` |

## 六、易错点

| # | 坑 |
|---|---|
| 1 | **多参数不写 `@Param`，`#{}` 里写业务名** → 报找不到参数，只能写 `arg0` / `param1` |
| 2 | **`@Param("username")` 里的字符串和 `#{}` 里的名字不一致** → 同样报错 |
| 3 | **查一条记录却返回 `List<User>`** → 能跑但语义不对；一条就用 `User`，多条才用 `List` |
| 4 | **查不到时返回 `null`** → 调用方**必须判空**，否则 `user.getUsername()` 直接空指针 |
| 5 | **返回多条时用 `User` 接收** → 报 `TooManyResultsException`，这是典型的"用户名重复"信号 |
| 6 | **忘了 `#{}` 用 `#`** → 写成 `${}` 有注入风险（前面刚讲过） |
| 7 | 参数名用 `username` 时注意别和方法里的**字段名**混 —— 这里的名字是"给 MyBatis 看的"，不是属性名 |
| 8 | 别把 `@Param` 当成参数校验注解（那是 `@NotNull` 那一类），它**只管命名** |

## 七、一句话记
**`@Param` = 给形参起名字，让 `#{}` 能对上号。**&#8203;
**单个值是随便写、单个对象写属性名、多个参数必须靠 `@Param`（编译后形参名会丢）。**&#8203;
**Spring Boot 官方骨架保留了形参名所以能省，但写上最稳。**&#8203;

---

# MyBatis —— XML 映射配置

## 一、两种配置 SQL 的方式（课件原文）
**在 MyBatis 中，既可以通过「注解」配置 SQL 语句，也可以通过「XML 配置文件」配置 SQL 语句。**&#8203;

| 方式 | 写法 | 适合 |
|---|---|---|
| **注解** | `@Select("...")` 写在接口方法上 | **简单 SQL**（单表 CRUD）、看得直观 |
| **XML** | `.xml` 文件里写 `<select>` 标签 | **复杂 SQL**（多表关联、动态 SQL） |

**所以不是"XML 取代注解"，而是"复杂了才搬到 XML"。**&#8203;

## 二、⭐ 三条默认规则（课件原文，必须全对上）

| # | 规则 | 说白了 |
|---|---|---|
| 1 | XML 映射文件的**名称与 Mapper 接口名称一致**，并且和 Mapper 接口**放在相同包下**（**同包同名**） | 两个文件要能"认出彼此" |
| 2 | XML 映射文件的 **`namespace` 属性为 Mapper 接口全限定名** | XML 声明"我服务于哪个接口" |
| 3 | XML 映射文件中 SQL 语句的 **`id` 与 Mapper 接口中的方法名一致**，并**保持返回类型一致** | 一条 SQL 对应一个方法 |

**一句话：文件名对上 → namespace 对上 → id 对上，三层对应，绑死在一条线上。**&#8203;

（课件图里要特别注意：接口在 `src/main/java/com/itheima/mapper/UserMapper`，XML 在 `src/main/resources/com/itheima/mapper/UserMapper.xml` —— **一个在 java 下，一个在 resources 下**，包名和文件名完全一致，这就是"同包同名"的意思。）

## 三、两边的代码对照

**Mapper 接口：**

    @Mapper
    public interface UserMapper {
        public List<User> findAll();
    }

**XML 映射文件：**

    <?xml version="1.0" encoding="UTF-8" ?>
    <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
            "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
            
    <mapper namespace="com.itheima.mapper.UserMapper">
    
        <select id="findAll" resultType="com.itheima.pojo.User">
            select id, username, password, name, age from user
        </select>
        
    </mapper>

**逐项对应：**

| 接口里                                    | XML 里                                       |
| -------------------------------------- | ------------------------------------------- |
| `UserMapper`（接口名）                      | `UserMapper.xml`（文件名）                       |
| `com.itheima.mapper.UserMapper`（包名+类名） | `namespace="com.itheima.mapper.UserMapper"` |
| `findAll()`（方法名）                       | `<select id="findAll">`                     |
| `List<User>` 的 `User`                  | `resultType="com.itheima.pojo.User"`        |

## 四、几个必须有的细节

| 项                  | 说明                                                                    |
| ------------------ | --------------------------------------------------------------------- |
| **XML 头部声明**       | `<?xml ...?>` + `DOCTYPE`（约束），**少了 DTD 标签提示不了、也可能校验失败**               |
| **根标签 `<mapper>`** | 所有 SQL 写在它里面                                                          |
| **`resultType`**   | 写**实体类的全限定名**（单条记录映射成什么对象）；返回 `List<User>` 时**仍然写 `User`**，不用写 `List` |
| **接口仍要 `@Mapper`** | XML 只管 SQL，接口还是要靠 `@Mapper` 让 MyBatis 生成代理对象                          |

## 五、易错点

| # | 坑 |
|---|---|
| 1 | **XML 建在了 java 目录下** → 默认**不会被打包进 classpath**，导致找不到映射文件；**正确位置是 `resources` 下同包路径** |
| 2 | **`namespace` 写错（漏包名 / 拼错）**&#8203; → 启动报 `Invalid bound statement (not found)`，这是最经典的报错 |
| 3 | **`id` 和方法名不匹配** → 同样是 `Invalid bound statement (not found)` |
| 4 | **`resultType` 写成了 `List<User>`** → 报错，它要的是**元素的类型**（`User`） |
| 5 | **`resultType` 写成 `resultMap`** → 语义完全不同（`resultMap` 是自定义映射，后面才学） |
| 6 | **XML 里 SQL 末尾的分号** → 一般不加（`;` 在某些场景会报错），按课件写法**不写分号** |
| 7 | **同名文件同时有注解和 XML 的同名 SQL** → 会冲突报错，一个方法只能用一种方式 |
| 8 | **接口和 XML 只有一半对上**（比如文件名对但 namespace 错）→ 照样启动失败，**三条必须全对** |

## 六、这一页的真正意义

注解写复杂 SQL 会**越来越难读**（比如多条件动态查询要拼一堆 `<script>`）；
XML 天生适合**大段 SQL + 动态标签**，还能用工具做语法高亮和校验。

👉 **简单 SQL 用注解、复杂 SQL 用 XML，是实际项目的常规做法。**&#8203; 后面学的 **动态 SQL（`<if>` / `<where>` / `<foreach>`）必须在 XML 里写** —— 所以这一页是那部分的前置。

## 七、一句话记
**XML 映射三条铁律：同包同名、namespace = 接口全限定名、id = 方法名（返回类型也要一致）。**&#8203;
**接口在 java 下、XML 在 resources 下的同一个包路径里；对不上就报 `Invalid bound statement (not found)`。**&#8203;

---

# XML 映射文件 —— 辅助配置

## 一、指定 XML 映射文件位置

    # 指定XML映射配置文件的位置
    mybatis.mapper-locations=classpath:mapper/*.xml

写在 `application.properties` 里，告诉 MyBatis **去哪儿找 XML 映射文件**。

**这行怎么读（对照上图）：**

| 部分 | 含义 |
|---|---|
| `classpath:` | 从**类路径根**开始找（就是编译后 `target/classes`，也就是 `resources` 里的内容） |
| `mapper/` | `mapper` 目录下的文件（路径和接口所在包名对应） |
| `*.xml` | 所有以 `.xml` 结尾的文件（`*` 是通配符，匹配任意文件名） |

## 二、常见疑问：这行必须配吗？

**不一定。**&#8203; MyBatis（尤其配合 Spring Boot 时）**有默认搜索规则**：
只要满足上一页那三条"同包同名"铁律，通常**不配也能找到**。

| 情况 | 要不要配 |
|---|---|
| 接口与 XML 同包同名、放在 `resources/mapper` 下 | 一般**可以省略** |
| 启动报 `Invalid bound statement (not found)` | **补上这行**，或检查三条规则 |
| XML 放在自定义目录（如 `resources/myxml`） | **必须配**，改成对应的路径 |
| 想更保险、更明确 | 写上无妨（黑马课件就是这么做的） |

## 三、`classpath:` 和 `classpath*:` 的区别（知道就行）

| 写法 | 含义 |
|---|---|
| `classpath:` | 只找**第一个**匹配到的路径 |
| `classpath*:` | 找**所有** jar / 目录里匹配的路径（**更保险**，推荐） |

单项目里两者没区别；项目分模块打包成多个 jar 时，`classpath*:` 才找得全。

## 四、MyBatisX 插件（课件后半部分）

**MyBatisX 是一款基于 IDEA 的快速开发 MyBatis 的插件，为效率而生。**&#8203;

| 项 | 信息 |
|---|---|
| 安装方式 | `Settings` → `Plugins` → `Marketplace` 搜 `MyBatisX` → `Install` |
| 核心功能 | **mapper 接口与 xml 之间来回跳转**（点小图标直接跳） |
| | mapper / xml 的自动提示 |
| | 自动生成语句（mapper 和 xml 互相生成提示） |

**它解决什么痛点？**
前面说了"接口在 java 下、XML 在 resources 下，要同包同名" —— 但两个文件**离得很远**，来回找很烦。
装上它，接口方法旁边会出现一个**小图标**，一点就跳到对应 XML 的 SQL 上，反之亦然。

（**Lombok 已装的话，这个也建议装** —— 属于开发提效插件，不是运行必需。）

## 五、易错点

| # | 坑 |
|---|---|
| 1 | **配了 `mapper-locations` 但路径写错** → 照样 `Invalid bound statement (not found)`，注意 `classpath:` 后面**没有斜杠** |
| 2 | **XML 放在了 `java` 目录下** → 默认不参与打包，配置再对也找不到；**XML 放 `resources`** |
| 3 | **只装 MyBatisX 不装 Lombok** → 没影响，但两个一起装开发体验最好 |
| 4 | 装了插件**不代表不用遵守三条映射规则** → 插件只是帮你跳转/提示，规则错了照样运行失败 |
| 5 | 配置项名前缀是 `mybatis.`，**不是 `mybatis-plus.`** —— 两者是不同框架（MyBatis-Plus 有自己的配置方式） |

## 六、一句话记
**`mybatis.mapper-locations=classpath:mapper/*.xml` —— 告诉 MyBatis 去哪找 XML；同包同名满足时可省，报找不到就补上。**&#8203;
**MyBatisX 是 IDEA 插件，专治"接口和 XML 离得远、来回找"，装上就能互跳。**&#8203;
