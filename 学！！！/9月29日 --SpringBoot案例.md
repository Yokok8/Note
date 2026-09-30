#  需求说明

## 一、需求原文
**基于 SpringBoot 开发 web 程序，完成用户列表的渲染展示。**&#8203;

拆开看就三个词：
- **web 程序** → 要用浏览器能访问（所以要走 HTTP 接口）
- **用户列表** → 数据是"一堆用户"，返回的是**数组**
- **渲染展示** → 展示交给前端，我们只管**把数据给出去**

## 二、完整数据流（4 步）

| 步 | 谁 | 做什么 | 用什么 |
|---|---|---|---|
| ① | 浏览器 | 访问 `http://localhost:8080/user.html` | 静态页面（前面学的 `resources/static`） |
| ② | 前端页面 | 发 **ajax** 请求到 `http://localhost:8080/list` | 请求后端要数据 |
| ③ | **服务端** | 加载 `user.txt` 里的数据，**响应 JSON 格式** | **← 这部分是你要写的** |
| ④ | 前端页面 | 拿到 JSON，**渲染到表格里** | 前端自己处理 |

⭐ 看清这四步的分工：**① ④ 是前端的事，② 是浏览器自动发的，只有 ③ 要你写。**&#8203;
你写的代码一共就干两件事：**读文件 + 返回数据。**&#8203;

## 三、数据长什么样

`user.txt` 每行一条记录，字段用逗号分隔：

    id,username,password,name,age,updateTime
    1,daqiao,1234567890,大乔,22,2024-07-15 15:05:45

| 列 | 说明 |
|---|---|
| id | 编号 |
| username | 用户名 |
| password | 密码 |
| name | 姓名 |
| age | 年龄 |
| updateTime | 最后修改时间 |

## 四、接口约定

| 项 | 值 |
|---|---|
| 请求方式 | **GET** |
| 请求路径 | `/list` |
| 请求参数 | 无 |
| 响应格式 | **JSON 数组** |

响应长这样：

    [
      {"id":1,"username":"daqiao","name":"大乔","age":22,"updateTime":"2024-07-15 15:05:45"},
      {"id":2,"username":"xiaoqiao","name":"小乔","age":18,"updateTime":"2024-07-15 15:15:09"}
    ]

## 五、这个案例的真正意义（别只当成练习题）

| 你看到的 | 它在教你什么 |
|---|---|
| 前端页面已经给好了 | **前后端分离**：后端只负责出数据，不管页面长什么样 |
| 数据源是 `user.txt` 不是数据库 | 还没到数据库，**先用 IO 流顶替**（正好复习刚学的 IO） |
| 返回的是 JSON | `@ResponseBody` 的**实战落地** —— `List<User>` 自动变 JSON 数组 |
| 路径叫 `/list` | 这就是后面 RESTful 风格的雏形 |

**一句话：这就是"最小可用的前后端分离后端"。**&#8203; 后面把 txt 换成 MySQL、把方法拆成三层，就是正式项目的样子。

## 六、易错点（提前避坑）
1. **静态页面路径不写 static**：访问 `localhost:8080/user.html`，不是 `/static/user.html`
2. 接口路径是 **`/list`**，别自己发明 `/userList`（前端已经按 `/list` 写死了，路径不一致直接 404）
3. 返回的必须是 **`List<User>`（数组）**&#8203;，不是单个 `User`，否则前端 `forEach` 渲染会报错
4. `user.txt` 每行按**逗号切分**，切完要有 **6 段**，字段顺序必须和 User 类对上
5. **文件读不到**（`FileNotFoundException`）时先看路径 —— 用 `resource` 目录下的相对路径，别写你电脑上的绝对路径
6. 这个案例**暂时不用 `Result` 封装**，直接返回 List，让前端好处理（后面案例才引入）

## 七、一句话记
**浏览器打开 user.html → 页面 ajax 请求 /list → 后端读 user.txt 返回 JSON → 页面渲染表格。**&#8203;
**你要写的只有中间那一步。**&#8203;







# 案例小结：静态资源 + @ResponseBody

## 一、静态资源存放位置

**位置：`resources/static`**（完整路径 `src/main/resources/static`）

| 问题 | 答案 |
|---|---|
| 放什么 | HTML、CSS、JS、图片等**不需要程序处理**的文件 |
| 怎么访问 | **不写 static**，直接从根路径访问 |
| 例子 | `static/index.html` → 访问 `http://localhost:8080/index.html` |

**为什么会这样？** Spring Boot 项目和传统 web 项目不一样：
- 没有 `webapp` 目录，也没有外置 Tomcat
- 静态文件放 `static` 下，Spring Boot **自动**把它们当静态资源处理

其他也能生效的默认目录（知道就行）：`resources/public`、`resources/resources`、`resources/META-INF/resources`

补充：如果 `static` 下有 `index.html`，访问根路径 `/` 会**默认返回它**。

## 二、@ResponseBody 注解的作用（三条）

1. 将 controller 方法的返回值**直接写入 HTTP 响应体**
2. 如果返回值是**对象或集合**，会**先转成 JSON**，再响应
3. **`@RestController` = `@Controller` + `@ResponseBody`**

### 加不加的区别（核心理解）

| | 不加 @ResponseBody | 加了 @ResponseBody |
|---|---|---|
| 返回值被当成 | **视图名**（页面名字） | **数据** |
| 结果 | 去找页面、跳转过去 | 直接写进响应体发回浏览器 |
| 用在什么场景 | 传统服务端渲染（JSP / Thymeleaf） | **前后端分离（现在的主流）**&#8203; |

所以 `@RestController` 一写，这个类里**所有方法**的返回值都当数据返回 —— 这就是为什么我们写接口几乎都用它。

### 转 JSON 是谁干的？
`@ResponseBody` 生效后，由 **HttpMessageConverter（消息转换器）**&#8203; 负责把返回值序列化写进响应体。
对象 / 集合 → JSON 用的是 **Jackson**（Spring Boot 起步依赖里自带）。

## 三、易错点
1. `@ResponseBody` **可以加在类上**，表示该类所有方法都生效；`@RestController` **只能加在类上**，别往方法上放
2. **返回 `String` 时不会转 JSON** —— 原样写入响应体
   - 返回 `"abc"`，页面看到的是 `abc`，**不是** `"abc"`（带引号的才是 JSON 字符串）
   - 只有返回**对象 / 集合**才会走 JSON 转换
3. 路径里**不写 static**：写 `/static/index.html` 是**错的**，应该写 `/index.html`
4. 静态资源和接口路径**冲突时**，接口优先级更高（先匹配 Controller，再找静态资源）

## 四、一句话记
**静态资源扔 `static`，访问时把 `static` 去掉。**&#8203;
**加了 @ResponseBody 就是"返回数据"，不加就是"去找页面"；对象转 JSON，字符串原样出。**&#8203;
**@RestController 就是 @Controller 加了 @ResponseBody。**&#8203;
