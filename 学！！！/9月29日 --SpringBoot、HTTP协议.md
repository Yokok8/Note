# Spring Boot 是什么

相比与SpringFramework，配置简化，快速入门

**Spring Boot 可以帮助我们非常快速地构建应用程序、简化开发、提高效率。**&#8203;
 
## 官网那句英文怎么理解
> Takes an **opinionated view** of building Spring applications and gets you up and running as quickly as possible.

- **opinionated** = "有主见的" → 在 Spring 里指 **约定优于配置**
  潜台词是：**别纠结怎么配了，我替你选好默认方案，你直接用**
- gets you up and running as quickly as possible = 让你**尽可能快地跑起来**

## 它究竟简化了什么（三大特性）

| 特性 | 以前多麻烦 | 现在 |
|---|---|---|
| **起步依赖** | 手动一个个人肉引 jar，还怕版本冲突 | 一个 `spring-boot-starter-web` **全带齐** |
| **自动配置** | 写一大堆 xml / 配置类 | 按引入的依赖**自动装配 Bean** |
| **内嵌服务器** | 装 Tomcat、打 war 包、手动部署 | 内嵌 Tomcat，**main 方法直接启动** |

## 一句话定位
**Spring Boot 不替代 Spring，它是 Spring 的"简化封装"。**&#8203;
底层还是 Spring 那一套（IOC、DI、AOP），只是把繁琐的配置都替你做了。
所以**不懂 Spring 直接学 Spring Boot，会变成只会抄配置** —— 这也是为什么前面反射、注解要先学。

## 一句话记
**起步依赖管"要什么"，自动配置管"怎么配"，内嵌服务器管"怎么跑"。**&#8203;

---
# HTTP 协议

## 概念（课件原文）
**Hyper Text Transfer Protocol（超文本传输协议）**&#8203;，规定了**浏览器和服务器之间数据传输的规则**。

拆开看这三个词：
- Hyper Text = 超文本（不只是文字，还有图片、视频、超链接）
- Transfer = 传输
- Protocol = 协议（就是"规则"）

**一句话：HTTP 是浏览器和服务器之间"说话的规矩"。**&#8203;

## 三大特点

| 特点 | 含义 | 怎么理解 |
|---|---|---|
| 基于 **TCP** 协议 | 面向连接、安全 | HTTP 属于**应用层**，底层靠 TCP 传数据 |
| 基于**请求-响应**模型 | 一次请求对应一次响应 | **先有请求，才有响应** —— 服务器不会主动推给你 |
| **无状态**协议 | 对事务处理没有记忆能力，每次请求-响应都是独立的 | 服务器**不记得你上次来过** |

## "无状态"是把双刃剑

| | 说明 |
|---|---|
| **优点** | **速度快** —— 不用为每个用户维护状态，省资源 |
| **缺点** | **多次请求间不能共享数据** —— 比如登录成功后，下一次请求服务器又不知道你是谁了 |

👉 这个缺点靠 **Cookie / Session / Token** 来补，后面会专门学。

## 和刚学的网络编程对上了（重点）

| 层次 | 协议 |
|---|---|
| **应用层** | HTTP、HTTPS、FTP |
| **传输层** | **TCP**、UDP |

打个比方：
- **TCP 负责"把字节可靠地送过去"**&#8203;
- **HTTP 负责"这些字节是什么意思"**&#8203;

所以你之前写的 `Socket` 是"搬运工"，HTTP 是"语言规则"。浏览器和服务器之间，实际就是**一条 TCP 连接上传 HTTP 格式的文本**。

## 一句话记
**HTTP = 浏览器和服务器说话的规则；跑在 TCP 上，一问一答，而且不记事。**&#8203;
"不记事"这件事，靠 Cookie / Session 来补。


---

# HTTP 请求数据格式

## 三部分结构

| 部分 | 位置 | 内容 |
|---|---|---|
| **请求行** | 第 1 行 | 请求方式 + 资源路径 + 协议版本 |
| **请求头** | 第 2 行开始 | 附加说明，每行一个 `key: value` |
| **请求体** | **空行之后** | 真正要发的数据，**只有 POST 有** |

## 请求行逐段拆解

    GET /brand/findAll?name=OPPO&status=1 HTTP/1.1

- `GET` → 请求方式
- `/brand/findAll?name=OPPO&status=1` → 资源路径，`?` 后面是**参数**
- `HTTP/1.1` → 协议版本

## 请求头常见字段

| 字段 | 作用 |
|---|---|
| `Host` | 目标主机（域名:端口） |
| `User-Agent` | 客户端标识（操作系统 + 浏览器版本） |
| `Accept` | 我能接收的数据类型 |
| `Accept-Encoding` | 我能接收的压缩格式（gzip） |
| `Accept-Language` | 语言偏好 |
| `Cookie` | 带上本地存的 Cookie（**就是"无状态"的补丁**） |
| `Content-Type` | **请求体的数据类型**（如 `application/json`） |
| `Content-Length` | 请求体的字节长度 |

最该记住的两个：**`Content-Type`**（请求体是什么格式）和 **`Cookie`**（是谁在请求）。

## ⭐ GET vs POST（本页重点）

| | GET | POST |
|---|---|---|
| 参数在哪 | **请求行**（URL 的 `?` 后面） | **请求体** |
| 有没有请求体 | **没有** | 有 |
| 大小限制 | 浏览器中**有限制** | **没有限制** |
| 安全性 | 参数暴露在地址栏 | 参数藏在请求体，相对安全 |

对照例子：
- **GET**：`GET /brand/findAll?name=OPPO&status=1 HTTP/1.1`（参数直接在路径里）
- **POST**：`POST /brand HTTP/1.1`，路径干干净净，参数在下面请求体里
  `{"status":1,"brandName":"黑马","companyName":"黑马程序员"}`

## 易错点
1. **请求头和请求体之间有一个空行**分隔，别以为漏了内容
2. GET 也能传参，只是参数在 URL 里 —— 不是"GET 不能带参数"
3. 课件说 POST"没有大小限制"是**相对说法**：实际 Tomcat、Nginx 都有最大请求体配置，超了照样被拒
4. 请求头格式是 `key: value`，**冒号后面有一个空格**
5. `Content-Type` 描述的是**请求体**格式，别和响应头搞混

## 一句话记
**第一行说"要什么"，中间几行说"我是谁、我能收什么"，最后带数据 —— 只有 POST 才带请求体。**&#8203;
**GET 参数在地址栏，POST 参数在请求体。**&#8203;

---

# HTTP 请求数据的获取（HttpServletRequest）

## 核心一句话（课件原文）
Web 服务器（**Tomcat**）对 HTTP 协议的请求数据进行**解析**，并进行了**封装**（`HttpServletRequest`），在调用 Controller 方法的时候**传递给了该方法**。
这样，就使得程序员**不必直接对协议进行操作**，让 Web 开发更加便捷。

## 具体发生了什么

| 步 | 谁做的 | 做什么 |
|---|---|---|
| ① | 浏览器 | 发出 HTTP 请求文本 |
| ② | **Tomcat** | 解析文本（请求行 / 请求头 / 请求体） |
| ③ | **Tomcat** | 封装成一个 `HttpServletRequest` 对象 |
| ④ | **Tomcat** | 调用你的 Controller 方法，**把对象当参数塞进去** |
| ⑤ | 你 | 直接 `request.getXxx()` 取值，**完全不碰协议原文** |

## 对比一下你之前手写 Socket（这个对比最能说明框架的价值）

**如果自己写，要处理一个 HTTP 请求得：**
1. `ServerSocket` 监听端口
2. `accept()` 拿到连接
3. 从输入流一个字节一个字节读
4. 按 `\r\n` 切分，找到请求行、请求头
5. 解析 `GET /xx?a=1 HTTP/1.1`，手动抠出路径和参数
6. 还要处理编码、边界情况、异常……

**现在呢？**
```java
    @GetMapping("/brand/findAll")
    public Result findAll(HttpServletRequest request) {
        String name = request.getParameter("name");   // 一行就拿到 OPPO
        return Result.success();
    }
```
👉 **这就是框架存在的意义：把"协议细节"包起来，让你只写业务逻辑。**&#8203;

## HttpServletRequest 常用方法（先眼熟）

| 方法 | 作用 |
|---|---|
| `getParameter(String name)` | 取请求参数 —— **最常用** |
| `getParameterValues(String name)` | 取多个值（如复选框） |
| `getMethod()` | 取请求方式（GET / POST） |
| `getRequestURI()` / `getRequestURL()` | 取请求路径 |
| `getHeader(String name)` | 取请求头，如 `getHeader("User-Agent")` |

## 一句话记
**Tomcat 是"翻译官"：把 HTTP 原文翻译成 `HttpServletRequest` 对象，递给你的方法。**&#8203;
**你只管调方法取数据，协议的事它全包了。**&#8203;

---
# HTTP 响应数据格式 + 状态码

## 一、响应报文三部分

    HTTP/1.1 200 OK                     ← 响应行
    Content-Type: application/json      ← 响应头
    Transfer-Encoding: chunked
    Date: Tue, 10 May 2022 07:51:07 GMT
    Keep-Alive: timeout=60
    Connection: keep-alive
                                        ← 空行
    {"id":1,"brandName":"阿里巴巴"}       ← 响应体

| 部分 | 位置 | 内容 |
|---|---|---|
| **响应行** | 第 1 行 | 协议版本 + **状态码** + 状态描述 |
| **响应头** | 第 2 行开始 | `key: value` 格式的附加说明 |
| **响应体** | 最后 | 真正返回给浏览器的数据（HTML / JSON / 图片…） |

对照请求报文记：**结构一模一样，只有第一行的内容不同**。
- 请求行：`GET /path HTTP/1.1`（方式 + 路径 + 版本）
- 响应行：`HTTP/1.1 200 OK`（版本 + 状态码 + 描述）

## 二、状态码五大类（核心）

| 类别 | 含义 | 记忆点 |
|---|---|---|
| 1xx | 临时状态码 | 收到了，**你继续发或忽略它** |
| 2xx | 成功 | 请求已接收，处理完成 |
| 3xx | 重定向 | 我这没有/换地方了，**你再发一次请求** |
| 4xx | **客户端错误** | **锅在客户端**：资源不存在、未授权、禁止访问 |
| 5xx | **服务器错误** | **锅在服务端**：程序抛异常 |

⭐ 判断口诀：**4 开头怪客户端，5 开头怪服务端。**&#8203;

## 三、常见状态码（眼熟即可）

| 码 | 含义 | 什么时候见到 |
|---|---|---|
| 200 | OK | 一切正常，最常看到 |
| 302 | 重定向 | 访问 `a.com` 跳到 `b.com` |
| 304 | 资源未修改 | 走浏览器缓存，没重新下载 |
| 400 | 请求参数错误 | 参数格式/必填项不对 |
| 401 | 未授权 | 没登录、token 失效 |
| 403 | 禁止访问 | 登录了但没权限 |
| 404 | 资源不存在 | **URL 写错了** |
| 500 | 服务器内部错误 | **后台代码抛异常了** |

排查优先级：**看到 404 查 URL，看到 500 查服务端日志。**&#8203;

## 四、响应头常见字段

| 字段 | 含义 | 细节 |
|---|---|---|
| `Content-Type` | 响应内容的**类型** | `text/html`、`application/json` |
| `Content-Length` | 响应内容的**长度** | **单位字节**，不是字符 |
| `Content-Encoding` | 响应内容的**压缩算法** | 如 `gzip`（和请求头的 Accept-Encoding 对应） |
| `Cache-Control` | 客户端**该怎么缓存** | `max-age=300` = 最多缓存 300 秒 |
| `Set-Cookie` | 告诉浏览器**给当前域设 Cookie** | **就是"无状态"的补丁** |

记法：**Content-* 三个都在描述"响应体"（什么类型、多大、压没压）；Cache-Control 管存不存；Set-Cookie 管认人。**&#8203;

## 五、易错点
1. **响应体的编码看 `Content-Type` 后面的 charset**，不含 charset 时浏览器可能乱码
2. `Content-Length` 是**字节数**，中文一个字在 UTF-8 占 3 字节
3. **`Content-Encoding` ≠ `Content-Type`**：前者说"压没压缩"，后者说"是什么格式"
4. **重定向要发两次请求**：第一次 3xx，浏览器自动再发一次才拿到 200
5. **404 不是服务器挂了**，是路径对不上 —— 服务器其实正常响应了

## 六、一句话记
**响应行说结果（几几几），响应头说这包数据什么样，响应体才是真东西。**&#8203;
**4 开头自己检查，5 开头去看后台日志。**&#8203;

---

# HTTP 响应数据的设置（两种方式）

## 核心认知（先记这句）
两种方式要做的事**完全一样**：设 **状态码 + 响应头 + 响应体**，只是写法不同。

## 方式一 · HttpServletResponse（命令式）

    @RequestMapping("/response")
    public void response(HttpServletResponse response) throws IOException {
        response.setStatus(401);                                    // 1. 状态码
        response.setHeader("itheima", "itheima");                   // 2. 响应头
        response.getWriter().write("<h1>Hello Response</h1>");      // 3. 响应体
    }

- **返回值是 `void`** —— 数据已经通过 response 对象写出去了，不需要 return
- `HttpServletResponse` 是 **Tomcat 传进来的参数**（和之前的 `HttpServletRequest` 一样）
- 三个方法：`setStatus` / `setHeader` / `getWriter().write(...)`

## 方式二 · ResponseEntity（声明式 + 链式）

    @RequestMapping("/response")
    public ResponseEntity<String> response() {
        return ResponseEntity.status(401)                           // 1. 状态码
                .header("group", "itcast")                          // 2. 响应头
                .body("<h1>Hello Response</h1>");                   // 3. 响应体
    }

- **返回值必须改**成 `ResponseEntity<泛型>`，泛型写**响应体的类型**（这里是 String）
- 三件事靠**链式调用**：`status()` → `header()` → `body()`
- 数据是 **return 出去**的，不是自己往流里写

## 两种方式对比（重点）

| | HttpServletResponse | ResponseEntity |
|---|---|---|
| 谁给的 | **Tomcat 当参数传进来** | **自己 new / 链式构建** |
| 返回值 | `void` | `ResponseEntity<T>` |
| 写法 | 命令式，一行一个 set | 链式，一行接一行 |
| 状态码 | `setStatus(int)` | `.status(int)` |
| 响应头 | `setHeader(k, v)` | `.header(k, v)` |
| 响应体 | `getWriter().write(内容)` | `.body(内容)` |
| 管不管流 | **要自己管**（getWriter / getOutputStream） | **不用管**，框架替你写 |

## 底层发生了什么

| 方式 | SpringMVC 的处理 |
|---|---|
| 方式一 | 发现参数里有 response，方法内部已经把报文写完了，**直接返回** |
| 方式二 | 拿到返回的 ResponseEntity，**解析它的 status / header / body**，再拼成报文写出去 |

两种最后都生成左边那样一段响应报文：

    HTTP/1.1 200 OK
    Server: Apache-Coyote/1.1
    Content-Type: text/html;charset=UTF-8
    Content-Length: ...
    Date: Wed, 06 Sep 2023 02:32:51 GMT

    <h1>Hello HTTP ~</h1>

## 易错点
1. **`setStatus` 只是设状态码，不影响后面写多少内容** —— 设 401 照样能把响应体写出去
2. **`setHeader` 是覆盖，`addHeader` 是追加**：同名 key 用 set 会覆盖上一次的值
3. `getWriter()` 拿的是**字符流**（适合写文本，自动处理编码）；要输出**图片/文件**得用 `getOutputStream()`（字节流）
4. **两个流不能同时用**：用完一个再拿另一个会抛 `IllegalStateException`
5. 用 `ResponseEntity` 时**别忘了改返回值类型**，否则 `.body()` 的结果不会被当成响应体
6. 方式一方法上写 `void`，如果顺手 return 了字符串，会被当成**视图名**去跳页面（不是写进响应体）

## 一句话记
**要设的就三样：状态码、响应头、响应体。**&#8203;
**方式一靠"传进来的 response"一个个 set，方式二靠"返回 ResponseEntity"链式拼。**&#8203;
**一个自己写流，一个交给框架写。**&#8203;
