# tlias 实战 · Filter（过滤器）快速入门

## 一、核心速记（图上两步）

```text
第 1 步 定义 Filter：定义一个类，实现 Filter 接口，并实现其所有方法
第 2 步 配置 Filter：Filter 类上加 @WebFilter 注解，配置拦截路径
                    引导类上加 @ServletComponentScan 开启 Servlet 组件支持
```

- 缺任何一步都白干：**少了第 2 步，Filter 根本不执行，日志里啥也看不到**（最常见坑）
- Filter 属于 Servlet 规范的三大组件之一：Servlet（处理请求）、Filter（过滤请求）、Listener（监听事件）
- 它工作在 **Servlet 容器层**（Tomcat），比 Spring MVC 的 Controller 更早，所以能拦下所有请求

## 二、第 1 步：Filter 的三个方法（完整代码）

```java
@WebFilter(urlPatterns = "/*")                     // ② 配置：拦截所有请求
public class DemoFilter implements Filter {        // ① 定义：实现 Filter 接口

    // 初始化方法，web服务器启动、创建 Filter 实例时调用，只调用一次
    public void init(FilterConfig filterConfig) throws ServletException {
        System.out.println("init ...");
    }

    // 拦截到请求时调用该方法，可以调用多次
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse,
                         FilterChain chain) throws Exception {
        System.out.println("拦截到了请求...");
        chain.doFilter(servletRequest, servletResponse);   // ★放行
    }

    // 销毁方法，web服务器关闭时调用，只调用一次
    public void destroy() {
        System.out.println("destroy ...");
    }
}
```

生命周期一览（这三个方法就是考点）：

| 方法 | 何时调用 | 调用次数 | 用途 |
| --- | --- | --- | --- |
| `init(FilterConfig)` | web 服务器启动、创建 Filter 实例时 | **1 次** | 初始化资源（如读配置） |
| `doFilter(req, res, chain)` | 每次请求被拦截到时 | **N 次** | 真正的业务：校验、记日志、放行 |
| `destroy()` | web 服务器关闭时 | **1 次** | 释放资源 |

★ `chain.doFilter(servletRequest, servletResponse)` 这一行是整个 Filter 的灵魂：

- 写了它 = 放行，请求会继续往下走到 Servlet / Controller
- 不写它 = **请求到此为止被吞掉**，前端一直转圈、拿不到任何响应
- 写两次 = 请求被重复执行两遍（危险，常见手误）
- 它后面的代码会在 Controller 执行完之后再执行（这就是"请求前 / 响应后"都能插手的原因）

## 三、第 2 步：@WebFilter + 引导类开关

```java
// 过滤器的拦截路径
@WebFilter(urlPatterns = "/*")
```

```java
// 引导类（启动类）
@ServletComponentScan          // ★ 开启 Servlet 组件支持，扫描 @WebFilter/@WebServlet/@WebListener
@SpringBootApplication
public class TliasManagementApplication {
    public static void main(String[] args) {
        SpringApplication.run(TliasManagementApplication.class, args);
    }
}
```

- `@ServletComponentScan` 是 Spring Boot 提供的，作用是**让容器去扫这些 Servlet 注解**，默认它不扫
- 它必须加在**引导类（有 `@SpringBootApplication` 的那个类）**上，加在 Controller 或 Filter 自己身上无效

拦截路径 `urlPatterns` 的四种写法：

| 写法 | 含义 |
| --- | --- |
| `/*` | 拦截所有请求（登录校验一般先用这个，再排除） |
| `/emps/*` | 拦截某个目录下所有 |
| `/login` | 精确匹配某一个 |
| `*.do` | 后缀匹配（少见） |

## 四、执行顺序（理解这个就理解"统一拦截"）

```text
浏览器请求
  → Filter.init（仅第一次启动时）
  → Filter.doFilter 前半段  ← 在这里校验令牌
  → chain.doFilter 放行
      → Spring MVC / Controller / Service / Mapper
  → Filter.doFilter 后半段  ← 可以在这里加工响应
  → 浏览器收到响应
```

- 所以"统一拦截"的活儿最适合放这：**一次编写，所有接口生效**
- 想"放行登录接口"，就在 `doFilter` 里判断 URL，不是的话直接 `return`

## 五、易错点

| 坑 | 现象 |
| --- | --- |
| 引导类忘加 `@ServletComponentScan` | Filter 完全不执行，控制台连 "init ..." 都没有 |
| 忘了 `chain.doFilter(...)` | 请求被吞，前端无响应、Postman 一直等 |
| `chain.doFilter` 写了两次 | 一个请求执行两遍（新增接口会插两条数据） |
| 导包用 `javax.servlet.Filter` | Spring Boot 3 要用 **`jakarta.servlet.*`**，导错启动报错 |
| `urlPatterns` 写成 `"*"` 少了斜杠 | 路径不合法，启动或匹配失效 |
| 以为能直接 `@Autowired` 注入 | `@WebFilter` 的 Filter 由 **Servlet 容器**创建，不是 Spring Bean，注入可能拿到 null |
| `doFilter` 参数当 `HttpServletRequest` 用 | 形参类型是 `ServletRequest`，要取请求头必须强转 `(HttpServletRequest)` |

### 关于注入那条的补救

后面登录校验会用到，先知道有这条路：去掉 `@WebFilter`，改成 `@Component` + 配置类注册，这样它才是 Spring Bean。

```java
@Bean
public FilterRegistrationBean<DemoFilter> register(DemoFilter filter) {
    FilterRegistrationBean<DemoFilter> bean = new FilterRegistrationBean<>(filter);
    bean.addUrlPatterns("/*");
    bean.setOrder(1);          // 多个 Filter 时的执行顺序
    return bean;
}
```

## 六、口诀

```text
init 一次起，doFilter 次次跑，destroy 一次收。
不写 chain.doFilter，请求直接进黑洞。
@WebFilter 定路径，引导类上 @ServletComponentScan 开闸。
```

---

# tlias 实战 · 登录校验 Filter（设计 + 流程）

## 第一部分：登录校验 Filter 的两个思考题

### 一、图上的场景

```text
浏览器 ──请求──▶ ┌──────────────┐ ──▶ Login / Emp / Dept / Report
       ◀─响应─── │    Filter     │
                 └──────────────┘
```

- 图上黄色竖块就是 Filter，它站在**所有 Controller 前面**
- 箭头是"分叉"的：Filter 之后才进到具体哪个模块 → 所以它是**统一入口**
- 和上一页 `DemoFilter` 的区别：这不再是打印日志的玩具，而是要**决定请求能不能往下走**

### 二、思考题 1：所有请求，拦截到之后都要校验令牌吗？

答案（图上红字）：**有一个例外 —— 登录请求。**

为什么？

- 登录请求本身就是来"换取令牌"的，此时用户手里**还没有令牌**
- 如果连它也要校验 → 第一次登录必然 401 → **永远登不上（死锁）**
- 所以判断逻辑必须是：**是登录请求 → 直接放行；其他请求 → 校验令牌**

### 三、思考题 2：什么情况下才可以放行？

答案（图上红字）：**有令牌，且令牌校验通过（合法）**，否则都返回未登录错误结果。

拆成两个必要条件（缺一不可）：

| 条件 | 不满足时 | 说明 |
| --- | --- | --- |
| ① 请求头里带了令牌 | 返回 401 | 前端没登录 / 忘了带 |
| ② 令牌能被正确解析（验签 + 未过期） | 返回 401 | 伪造的、改过的、过期的 |

- 「未登录错误结果」在黑马这套里约定为 **HTTP 状态码 401（Unauthorized）**
- 注意：401 是**状态码**，不是 `Result.error(...)` 那种 200 包着错误信息的自定义结构
- 前端 axios 通常会统一拦 401 → 清 token → 跳回登录页

### 四、为什么校验放在 Filter 而不是每个 Controller

| 方案 | 工作量 | 风险 |
| --- | --- | --- |
| 每个接口开头自己校验 | N 个接口写 N 遍 | 漏写一个就裸奔 |
| Interceptor | 需注册 + 配置 | 只覆盖 MVC 请求 |
| **Filter（本项目选择）** | 写一次 / 全站生效 | 静态资源也会被拦（需放行） |

- 这就是流程图开头那个"统一拦截"的意义：**把横切逻辑从业务里抽出来**

---

## 第二部分：令牌校验 Filter —— 六步流程

### 五、图上六步（原文照搬）

1. 获取请求 url
2. 判断请求 url 中是否包含 login，如果包含，说明是登录操作，放行
3. 获取请求头中的令牌（token）
4. 判断令牌是否存在，如果不存在，响应 401
5. 解析 token，如果解析失败，响应 401
6. 放行

### 六、流程图对照（图上右侧那张）

```text
请求
  ↓
获取请求路径          （示例：http://localhost:8080/login  /  http://localhost:8080/emps）
  ↓
判断是否是登录请求 ──是──▶ 放行
  ↓ 否
获取请求头 token
  ↓
判断是否有 token ──否──▶ 响应 401
  ↓ 是
解析 token ──失败──▶ 响应 401
  ↓ 成功
放行
```

- 这张图就是 if-else 的可视化，**每一条"否"分支都必须 `return`**，不能往下漏
- 三个出口只有一个通向业务：放行；另外两条都是 401

### 七、六个步骤对应到代码（完整骨架）

```java
@WebFilter(urlPatterns = "/*")          // 拦截所有请求
public class TokenFilter implements Filter {

    @Override
    public void doFilter(ServletRequest req, ServletResponse resp, FilterChain chain)
            throws IOException, ServletException {

        // 形参是 ServletRequest，要取 url / 请求头必须强转
        HttpServletRequest request = (HttpServletRequest) req;
        HttpServletResponse response = (HttpServletResponse) resp;

        // 第 1 步：获取请求 url
        String url = request.getRequestURL().toString();

        // 第 2 步：包含 login → 登录请求 → 直接放行
        if (url.contains("login")) {
            chain.doFilter(request, response);   // 放行
            return;                              // ★必须 return
        }

        // 第 3 步：获取请求头中的令牌
        String token = request.getHeader("token");

        // 第 4 步：令牌不存在 → 响应 401
        if (!StringUtils.hasLength(token)) {
            response.setStatus(401);
            return;                              // ★必须 return
        }

        // 第 5 步：解析 token，失败 → 响应 401
        try {
            JwtUtils.parseJWT(token);
        } catch (Exception e) {
            response.setStatus(401);
            return;                              // ★必须 return
        }

        // 第 6 步：放行
        chain.doFilter(request, response);
    }
}
```

#### ① getRequestURL 与 getRequestURI 的区别（面试常问）

| 方法 | 返回 | 示例 |
| --- | --- | --- |
| `request.getRequestURL()` | 完整 URL（含协议/IP/端口） | `http://localhost:8080/emps` |
| `request.getRequestURI()` | 只有路径部分 | `/emps` |

- 图上绿字用的就是完整 URL，所以代码配 `getRequestURL()`
- 两者用 `contains("login")` 判断结果一样，但要知道差别

#### ② StringUtils.hasLength 的判断范围

`StringUtils.hasLength(token)` 判断的是"非 null 且长度 > 0"，
等价于 `token != null && !token.isEmpty()`（注意没有判空白串，用 `hasText` 才判空白）。

### 八、易错点（这一段是本节最难的地方）

| 坑 | 后果 |
| --- | --- |
| 401 之后**忘了 `return`** | 继续往下执行 → 又走到 `chain.doFilter` → 401 白设了，请求照样放行 |
| 放行写成 `chain.doFilter(req, resp)` 但类型不对 | 编译没问题，但下游取不到；统一传强转后的 request/response |
| 忘记强转 `ServletRequest` | 拿不到 `getRequestURL` / `getHeader`（这两方法在 `HttpServletRequest` 上） |
| 前端请求头键名和 `getHeader("token")` 不一致 | 前端叫 Authorization、后端拿 token → 永远 401 |
| 只拦 `/emps/*` 漏了 `/depts`、`/report` | 一部分接口裸奔 |
| 拦截所有但没放行登录页静态资源 | 打开前端页面直接 401，白屏 |
| `contains("login")` 匹配过宽 | 若将来有 `/loginlog` 之类接口会被误放行（课件简化写法） |
| 用 `@WebFilter` 还想 `@Autowired` JwtUtils | JwtUtils 是 static 工具类，直接类名调用即可，不需要注入 |

补充：`urlPatterns="/*"` 会连**静态资源（html/js/css）和 `/login` 之外的一切**都拦，
实际项目里通常再加一段 `if (url.contains("/login")) 放行` 已够用，因为前后端分离时页面在另一个端口/域名。

### 九、口诀

```text
先判断：是不是登录？是就放行，否则往下查。
再取牌：没有牌 401，牌是假的 401。
三个 401 都要 return，少一个就前功尽弃。
校验通过才 chain.doFilter，这一步才是"进门"。
```

---

# Filter详解

## 一、Filter 执行流程

一句话：Filter 在请求到达资源的前后各插入一段逻辑，做"拦截 + 增强"。

流程：
    浏览器请求 → Filter 放行前逻辑 → 访问资源(Login/Emp/Dept/Report)
    → 回到 Filter 执行放行后逻辑 → 响应返回浏览器

代码分界线（chain.doFilter 上下就是前后）：
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain) {
        System.out.println("放行前逻辑");      // ①②
        chain.doFilter(request, response);    // ③ 放行，去访问资源
        System.out.println("放行后逻辑");      // ④ 资源访问完，回到这里
    }

两个必背问题：
    1. 资源访问完成后还会回到 Filter 吗？ —— 会
    2. 回到 Filter 是重新执行一遍吗？ —— 不是，只执行放行后的逻辑

## 二、Filter 拦截路径

    @WebFilter(urlPatterns = "/*")
    public class DemoFilter implements Filter { }

    拦截路径      urlPatterns 值    含义
    拦截具体路径   /login           只有访问 /login 时才被拦截
    目录拦截       /emps/*          访问 /emps 下所有资源都被拦截
    拦截所有       /*               访问所有资源都被拦截

## 三、Filter 过滤器链

概念：一个 web 应用可以配置多个过滤器，它们串起来就形成过滤器链。

执行顺序（后进先出，像栈）：
    请求 → Filter1 放行前 → Filter2 放行前 → 资源
    → Filter2 放行后 → Filter1 放行后 → 响应

优先级：注解方式配置的 Filter，按过滤器类名（字符串）自然排序，
        例如 AFilter 排在 BFilter 前面。

