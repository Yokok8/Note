# SpringBoot 配置文件格式（properties / yaml）

## 一、三种配置文件（课件原文）
**SpringBoot 项目提供了多种属性配置方式：`properties`、`yaml`、`yml`。**&#8203;

| 文件名 | 格式 | 现在用得多吗 |
|---|---|---|
| `application.properties` | key=value 扁平式 | 老项目常见 |
| `application.yaml` | 缩进层级式 | **主流，推荐** |
| `application.yml` | 同上（`.yaml` 的简写，效果一样） | **主流，推荐** |

⚠️ **同一个项目里三种同时存在时，`properties` 优先级最高**，容易造成"改了不生效"的假象 —— 所以**只留一种**。

## 二、两种格式对比（本页重点）

**application.properties：**

    spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
    spring.datasource.url=jdbc:mysql://localhost:3306/web01
    spring.datasource.username=root
    spring.datasource.password=1234

👉 问题：**前缀 `spring.datasource` 重复了四遍，层级结构不清晰**（课件批注原文）。

**application.yaml：**

    spring:
      datasource:
        driver-class-name: com.mysql.cj.jdbc.Driver
        url: jdbc:mysql://localhost:3306/web01
        username: root
        password: 1234

👉 优点：**简洁、数据为中心**，一眼看出谁属于谁（课件批注原文）。

### 本质：键名是"路径"，缩进是"分段"

| properties 的写法 | yaml 的写法 |
|---|---|
| `spring.datasource.url` | `spring:` → 缩进 `datasource:` → 再缩进 `url:` |
| 用 `.` 把层级连起来 | 用**缩进**把层级排出来 |

**两者完全等价，只是表达方式不同。**&#8203;

## 三、⭐ yml 语法四条规则（课件原文，必记）

| # | 规则 |
|---|---|
| 1 | **数值前边必须有空格**，作为分隔符（`port: 8080`） |
| 2 | 使用**缩进**表示层级关系，缩进时**不允许使用 Tab 键，只能用空格**（IDEA 中会自动将 Tab 转换成空格） |
| 3 | 缩进的**空格数目不重要**，只要**相同层级的元素左侧对齐**即可 |
| 4 | **`#` 表示注释**，从这个字符一直到行尾都会被解析器忽略 |

### 逐条举例

    server:
      port: 8080
      address: 127.0.0.1

| 规则 | 在图里怎么看 |
|---|---|
| ① 空格分隔 | `port:` 后面**必须有一个空格**再写值；`port:8080` 是**错的** |
| ② 只用空格 | `port` 前面那两个空格是**空格**，不是 Tab；混用直接报错 |
| ③ 层级对齐 | `port` 和 `address` 左侧对齐 → 说明是**同一层** |
| ④ `#` 注释 | `#配置服务器相关信息` 整行被忽略 |

## 四、定义对象 / Map

    user:
      name: 张三
      age: 18
      password: 123456

**写法：`对象名:` 换行 + 缩进写各个键值。**&#8203;

对应 properties 就是 `user.name=张三`、`user.age=18`。

## 五、定义数组 / List / Set

    hobby:
      - java
      - game
      - sport

**写法：`名称:` 换行 + 缩进用 `- `（短横线 + 空格）逐项列出。**&#8203;

| 细节 | 说明 |
|---|---|
| `-` 后面**必须有空格** | `-java` 是错的，会被当成一个整体 |
| 每项独占一行 | 缩进对齐 |
| 也可以写成行内式 | `hobby: [java, game, sport]` |

## 六、⚠️ 课件的重点注意（原文）

> 在 yml 格式的配置文件中，如果配置项的值是**以 0 开头**的，值需要使用 `''` 引起来，因为以 0 开头在 yml 中表示**八进制**的数据。

    password: '0123'      -- 正确，存的是字符串 "0123"
    password: 0123        -- 危险！会被当成八进制 → 实际值是 83

**最容易踩的场景**：手机号、身份证号、编号、密码 —— 这种东西一旦被转成八进制，数据就废了。

## 七、易错点汇总

| # | 坑 |
|---|---|
| 1 | **没有空格**：`port:8080` → 报错或解析异常 |
| 2 | **用了 Tab 缩进** → 报错（`found character '\t' that cannot start any token`） |
| 3 | **同层没对齐**（多一个/少一个空格）→ 层级别解析错，配置悄悄失效 |
| 4 | **0 开头的值不加引号** → 被当八进制 |
| 5 | **值里含 `:` 不加引号** → 如 `time: 2026-10-11 12:00:00`，冒号会被当层级分隔；**加引号包起来** |
| 6 | **`-` 后面忘空格** → 数组项解析失败 |
| 7 | **yaml 里写 `=`** → 不合法，yaml 用 `:` |
| 8 | **中文乱码** → 文件编码要 UTF-8 |
| 9 | **文件名必须是 `application`** —— 叫 `app.yaml` SpringBoot 认不到 |

## 八、怎么选

| 场景 | 建议 |
|---|---|
| 新项目 | **用 `application.yml`**（层级清晰，配置一多优势明显） |
| 只有三五项配置 | properties 也够用 |
| 老项目维护 | 跟着现有的走，别混用 |

## 九、一句话记
**properties 是"点名式"（前缀一遍遍重复），yaml 是"缩进式"（谁属于谁一目了然）。**&#8203;
**yml 四条铁律：冒号后加空格、缩进只用空格、同层左对齐、`#` 是注释。**&#8203;
**0 开头的值加引号，否则变八进制。**&#8203;
