# 一、拦截器 Interceptor

## 一、概念

一句话：Interceptor 是 Spring 框架提供的一种"动态拦截方法调用"的机制，
        拦截对象是 Controller 方法的执行。

拆开记三个词：
    动态      —— 运行时决定是否拦截，不写死在业务代码里
    拦截方法  —— 拦的是"方法调用"，不是拦截整个请求
    类似过滤器 —— 用法和 Filter 很像，都是包裹在目标逻辑前后

## 二、作用

在指定方法调用前后，按业务需要执行预先设定的代码。

典型落地场景：
    登录校验    判断请求是否携带 token，未登录直接打回
    权限验证    区分管理员 / 普通用户能访问哪些接口
    统一日志    记录接口调用时间、方法名、耗时
    性能监控    统计慢接口，排查性能瓶颈

## 三、执行流程

    浏览器发送请求
        ↓
    Interceptor（执行放行前逻辑）
        ↓
    目标资源：Login / Emp / Dept / Report
        ↓
    Interceptor（执行放行后逻辑）
        ↓
    响应返回浏览器

注意：和 Filter 一样，资源访问完后会回到拦截器，执行的是放行后逻辑，
      不会重新执行一遍。

## 四、与 Filter 的核心区别

    维度        Filter                     Interceptor
    规范归属    Servlet 规范（JavaWeb）     Spring MVC 框架提供
    拦截目标    所有资源（含 html/css/js）  只有 Controller 方法
    运行依赖    依赖 Tomcat 容器            依赖 Spring 容器
    典型用途    编码、跨域、请求包装         登录校验、权限、日志

一句话记忆：Filter 在大门口拦所有请求，Interceptor 在 Controller 门口拦业务方法。

---

# 二、Interceptor 快速入门

两步走：① 定义拦截器  ② 注册拦截器

## 1. 定义拦截器（实现 HandlerInterceptor）

```java
@Component
public class DemoInterceptor implements HandlerInterceptor {

    // 目标资源方法执行前执行；返回 true 放行，返回 false 不放行
    @Override
    public boolean preHandle(HttpServletRequest req, HttpServletResponse resp, Object handler) throws Exception {
        return true;
    }

    // 目标资源方法执行后执行
    @Override
    public void postHandle(HttpServletRequest req, HttpServletResponse resp, Object handler, ModelAndView mv) throws Exception {
        System.out.println("postHandle...");
    }

    // 视图渲染完毕后执行，最后执行
    @Override
    public void afterCompletion(HttpServletRequest req, HttpServletResponse resp, Object handler, Exception ex) throws Exception {
        System.out.println("afterCompletion...");
    }
}
```

三个方法对照表：
    方法               执行时机                  能不能拦住请求
    preHandle          目标方法执行前             能，返回 false 直接中断
    postHandle         目标方法执行后，视图渲染前   不能
    afterCompletion    视图渲染完毕后，最后执行     不能

## 2. 注册拦截器（配置类实现 WebMvcConfigurer）

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {

    @Autowired
    private DemoInterceptor demoInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(demoInterceptor).addPathPatterns("/**");
    }
}
```

路径模式说明：
    /**        拦截所有请求（推荐写法）
    /*         只拦一级路径
    补充：需要排除某些路径时加
        .excludePathPatterns("/login")   // 登录接口不拦

## 易错点
    1. preHandle 必须有返回值，忘记 return true 会导致所有请求被拦死
    2. 拦截器类要加 @Component，否则注入不进配置类
    3. 注册时漏掉 addPathPatterns，拦截器等于没生效
    4. 课件截图里 postHandle 打印的是 "preHandle..."，是笔误，实际应为 postHandle


---

# 三、令牌校验 - 在拦截器中校验

## 1. 校验六步

    1. 获取请求 url
    2. 判断 url 中是否包含 login，包含说明是登录操作，直接放行
    3. 获取请求头中的令牌（token）
    4. 判断令牌是否存在，不存在 → 响应 401
    5. 解析 token，解析失败 → 响应 401
    6. 全部通过 → 放行

## 2. 流程图（文字版）

    请求
      ↓
    获取请求路径  （例：http://localhost:8080/login、http://localhost:8080/emps）
      ↓
    判断是否是登录请求
      ├─ 是 → 放行
      └─ 否 → 获取请求头 token
              ↓
            判断是否有 token
              ├─ 否 → 响应 401
              └─ 是 → 解析 token
                      ├─ 解析失败 → 响应 401
                      └─ 解析成功 → 放行

核心思路：只有"登录接口"和"令牌合法"两条路能走到放行，
        其余所有分支都停在 401。

## 3. 对应代码骨架

```java
@Override
public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {

    // 1. 获取请求 url
    String url = request.getRequestURL().toString();

    // 2. 判断是否是登录请求，是则直接放行
    if (url.contains("login")) {
        return true;
    }

    // 3. 获取请求头中的令牌
    String token = request.getHeader("token");

    // 4. 判断令牌是否存在（null 或空串都算不存在）
    if (!StringUtils.hasLength(token)) {
        response.setStatus(401);
        return false;
    }

    // 5. 解析令牌，失败说明伪造或过期
    try {
        JwtUtils.parseJWT(token);
    } catch (Exception e) {
        response.setStatus(401);
        return false;
    }

    // 6. 校验通过，放行
    return true;
}
```

## 易错点
    1. 返回 401 后必须 return false，否则请求仍会继续往下走
    2. token 不存在 和 解析失败 是两种情况，都要拦，别只写一个
    3. StringUtils 用 org.springframework.util.StringUtils，hasLength 同时处理 null 和 ""
    4. 用 url.contains("login") 判断是偷懒写法，严格场景应走 excludePathPatterns


---

# 四、拦截器 - 拦截路径

## 1. 配置方式

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(demoInterceptor)
            .addPathPatterns("/**")          // 需要拦截哪些资源
            .excludePathPatterns("/login");  // 不需要拦截哪些资源（排除）
}
```

两个方法分工明确：
    addPathPatterns       拦截哪些
    excludePathPatterns   在拦截范围内，再排除掉哪些

## 2. 通配符对照表

    拦截路径      含义                能匹配                        不能匹配
    /*           一级路径            /depts、/emps、/login         /depts/1
    /**          任意级路径          /depts、/depts/1、/depts/1/2  无
    /depts/*     /depts 下的一级路径  /depts/1                     /depts/1/2、/depts
    /depts/**    /depts 下的任意级    /depts、/depts/1、/depts/1/2  /emps/1

## 3. 记忆点

    *  只代表"一层"
    ** 代表"零层或多层"

    /depts/*    → 必须有 /depts/ 前缀且后面只跟一层，所以 /depts 本身匹配不上
    /depts/**   → 后面的 /** 可以为零层，所以 /depts 本身能匹配上

推荐写法：addPathPatterns("/**") + excludePathPatterns 列出放行接口，
        比在每个接口里判断 url 更规范，也好维护。

## 易错点
    1. /** 和 /* 差一个星，拦截范围差很多
    2. excludePathPatterns 只对已拦截的路径生效，没配 addPathPatterns 时无从排除
    3. 多个排除路径用逗号隔开：excludePathPatterns("/login", "/emps/upload")


---


# 五、拦截器 - 执行流程

## 1. 完整执行顺序（请求进来 → 响应回去）

    浏览器
      ↓
    Filter 放行前逻辑
      ↓
    Interceptor preHandle
      ↓
    Controller（目标方法）
      ↓
    Interceptor postHandle
      ↓
    Interceptor afterCompletion
      ↓
    Filter 放行后逻辑
      ↓
    浏览器（响应）

顺序记忆：外边一圈是 Filter（进门、出门），里边一圈是 Interceptor（包住 Controller）。
        出去的时候顺序反过来：postHandle → afterCompletion → Filter 放行后。

## 2. 两者所处环境

    Filter       运行在 Web 服务器（Tomcat）里 —— 图中下方"Web 服务器"虚线框
    Interceptor  运行在 Spring 环境里 —— 图中右侧 spring 虚线框
    这决定了拦截范围：Filter 连静态资源都能拦，Interceptor 只能拦 Spring 管的 Controller。

## 3. Filter 与 Interceptor 区别

    区别一：接口规范不同
        Filter       实现 javax.servlet.Filter 接口
        Interceptor  实现 Spring 的 HandlerInterceptor 接口

    区别二：拦截范围不同
        Filter       拦截所有资源（html、css、js、图片、接口全都算）
        Interceptor  只拦截 Spring 环境中的资源（即 Controller 方法）

    补充对比：
        维度         Filter              Interceptor
        归属         Servlet 规范         Spring MVC
        依赖容器     Tomcat              Spring 容器
        能否拦静态   能                  不能
        能否用 Bean  不能（默认）         能，可注入 Mapper、Service

## 4. 记忆点
    - Filter 和 Interceptor 是"包裹"关系，不是并列关系
    - 谁先进谁后出：Filter 先放行，最后收尾
    - 需要操作 Spring Bean（查数据库、读配置）时优先用 Interceptor
