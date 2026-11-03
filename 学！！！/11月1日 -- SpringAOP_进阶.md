# 一、通知类型

## 1. 五种通知（按目标方法执行时机区分）

    @Around          环绕通知      目标方法前、后都执行
    @Before          前置通知      目标方法前执行
    @After           后置通知      目标方法后执行，无论是否有异常都执行
    @AfterReturning  返回后通知    目标方法后执行，有异常则不执行
    @AfterThrowing   异常后通知    目标方法发生异常后才执行

## 2. 各通知的方法签名写法

```java
@Slf4j
@Aspect
@Component
public class RecordTimeAspect {

    @Before("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
    public void before() {
        log.info("before ...");
    }

    @Around("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
    public Object around(ProceedingJoinPoint pjp) throws Throwable {
        log.info("around before ...");
        Object result = pjp.proceed();          // 必须自己调用，目标方法才执行
        log.info("around after ...");
        return result;                          // 必须返回
    }

    @After("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
    public void after() {
        log.info("after ...");
    }

    @AfterReturning("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
    public void afterReturning() {
        log.info("afterReturning ...");
    }

    @AfterThrowing("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
    public void afterThrowing() {
        log.info("afterThrowing ...");
    }
}
```

## 3. 执行顺序（文字版）

    目标方法正常执行：
        @Before → @Around 前半段 → 目标方法 → @Around 后半段
        → @AfterReturning → @After

    目标方法抛出异常：
        @Before → @Around 前半段 → 目标方法异常
        → @AfterThrowing → @After
        （@AfterReturning 不会执行）

    记忆：@After 是"必执行"，@AfterReturning 是"成功才执行"，
         @AfterThrowing 是"失败才执行"。

## 4. 两个必背注意点

    注意1：@Around 必须自己调用 ProceedingJoinPoint.proceed() 让原始方法执行，
           其他四种通知不需要考虑目标方法的执行。

    注意2：@Around 的返回值必须声明为 Object，用来接收原始方法的返回值。
           （返回 void 会导致 Controller 拿到 null）

## 5. 易错点

    1. 只有 @Around 能拿到 ProceedingJoinPoint 的参数，也只有它能控制目标方法是否执行
    2. @After 和 @AfterReturning 容易混：@After 无论成败都跑，@AfterReturning 只在正常返回时跑
    3. @Around 方法签名必须 throws Throwable
    4. 想"改返回值"或"必须拿到返回值"只能用 @Around


---


# 二、@PointCut - 抽取切点表达式

## 1. 作用

    把公共的切点表达式抽取出来，用到时直接引用该表达式即可。
    好处：一个切面里多个通知（@Around / @Before / @After...）共用一个表达式，改一处全部生效。

## 2. 写法

```java
@Pointcut("execution(* com.itheima.service.impl.DeptServiceImpl.*(..))")
public void pt() {}      // 方法体为空，只作为表达式的载体
```

    注意：被 @Pointcut 标记的方法，方法体留空，不需要写任何逻辑。

## 3. 引用方式

```java
@Around("pt()")
public Object recordTime(ProceedingJoinPoint joinPoint) throws Throwable {
    long begin = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    long end = System.currentTimeMillis();
    log.info("执行耗时: {} ms", end - begin);
    return result;
}

@Before("pt()")
public void before() { }

@After("pt()")
public void after() { }
```

## 4. 修饰符决定能不能跨类引用（重点）

    private  void pt() {}   仅能在当前切面类中引用
    public   void pt() {}   在其他外部的切面类中也可以引用

    跨切面类引用的写法：全类名 + 方法名
```java
@Around("com.itheima.aspect.RecordTimeAspect.pt()")
public Object other(ProceedingJoinPoint pjp) throws Throwable {
    return pjp.proceed();
}
```

## 5. 易错点

    1. @Pointcut 方法要写成 void，且方法体为空
    2. 引用时写的是"方法名()"，不要写成 execution(...) 表达式
    3. 别的切面类引用时用全限定名（包名.类名.方法名()），漏包名会解析失败
    4. 想让其他切面复用，修饰符必须是 public


---


# 三、通知顺序

## 1. 前提

    当多个切面的切入点都匹配到了目标方法，
    目标方法运行时，这些切面里的通知方法都会被执行。
    此时就产生了"谁先谁后"的问题。

## 2. 默认规则：按切面类名字母排序

    目标方法**前**的通知：字母排名靠前的先执行
    目标方法**后**的通知：字母排名靠前的后执行

    也就是：前置正序，后置逆序 —— 和 Filter、Interceptor 一样，先进后出。

## 3. 用 @Order(数字) 控制顺序

```java
@Slf4j
@Order(5)
@Aspect
@Component
public class RecordTimeAspect { }
```

    目标方法**前**的通知：数字小的先执行
    目标方法**后**的通知：数字小的后执行

    注意：@Order 的数值只表示相对大小，不要求连续（1、5、100 都行）。

## 4. 举例（图中三个切面）

    切面类              排序依据
    MyAspect2          类名字母序第 1
    MyAspect3          类名字母序第 2
    RecordTimeAspect   @Order(5)，数字最大排最后

    实际执行顺序：
        before 2  →  before 3  →  before 5
                ↓
            目标方法
                ↓
        after 5   →  after 3   →  after 2

    看图更直观：前置是 2、3、5 正着来，后置是 5、3、2 倒着回。

## 5. 记忆方法

    把切面想成一层层包在目标方法外面的壳：
        进入时从最外层往里穿（@Order 小的 / 字母靠前的在外层）
        出来时从最里层往外退，顺序自然反过来

## 6. 易错点

    1. @Order 加在切面类上，不是加在通知方法上
    2. 后置通知是"逆序"，不是和前置一样的顺序，这点最容易记错
    3. 类名排序是字符串比较，Aspect10 会排在 Aspect2 前面（字典序），所以规范做法是用 @Order


---


# 四、切入点表达式

## 1. 是什么

    描述切入点方法的一种表达式。
    作用：决定项目中哪些方法需要加入通知。

    常见两种形式：
        execution(...)      根据方法签名匹配
        @annotation(...)    根据注解匹配

## 2. execution 完整语法

```java
execution(访问修饰符?  返回值  包名.类名.?方法名(方法参数)  throws 异常?)
```

    带 ? 的三部分都可以省略：
        1. 访问修饰符   public、protected 等，可省略
        2. 包名.类名    可省略
        3. throws 异常  可省略（指方法上声明抛出的异常，不是实际抛出的异常）

    实际开发基本都写省略版：
```java
@Before("execution(void com.itheima.service.impl.DeptServiceImpl.delete(java.lang.Integer))")
public void before(JoinPoint joinPoint) { }
```

## 3. 两个通配符（重点区分）

    *    单个独立的任意符号
         可通配：任意返回值、任意包名、任意类名、任意方法名、
                 任意类型的一个参数，也可作为包名/类名/方法名的一部分

         execution(* com.*.service.*.update*(*))
                返回值任意  包名一部分  类名任意  方法名前缀update  一个任意参数

    ..   多个连续的任意符号
         可通配：任意层级的包、任意类型任意个数的参数

         execution(* com.itheima..DeptService.*(..))
                返回值任意  itheima下任意层级包  DeptService类  任意方法  任意参数

    一句话：* 顶一个，.. 顶一串。

## 4. 逻辑运算符

    && 与    || 或    ! 非
    用来组合复杂表达式，例：
        execution(* com.itheima.service.*.*(..)) && !execution(* com.itheima.service.*.get*(..))

## 5. @annotation 写法

    作用：匹配标识了特定注解的方法。

```java
@Around("@annotation(com.itheima.anno.LogOperation)")
public Object logAround(ProceedingJoinPoint joinPoint) throws Throwable {
    long startTime = System.currentTimeMillis();
    Object result = joinPoint.proceed();
    return result;
}
```

    配套：在需要被增强的方法上加该注解
```java
@LogOperation
@DeleteMapping
public Result delete(Integer id) {
    deptService.delete(id);
    return Result.success();
}

@LogOperation
@PostMapping
public Result save(@RequestBody Dept dept) {
    deptService.save(dept);
    return Result.success();
}
```

## 6. 两种方式怎么选

    execution      按包/类/方法名批量匹配，适合"整个业务层统一加日志"这种规则场景
    @annotation    按注解精确匹配，想加哪个方法就打哪个注解，灵活、粒度细

## 7. 易错点

    1. * 和 .. 别混：* 只代表一个，.. 代表零到多个
    2. 写 .. 时前后都要有点：com.itheima..DeptService
    3. @annotation 括号里写的是注解的**全类名**，不是注解名
    4. 自定义注解要有 @Target(ElementType.METHOD)，否则加不到方法上
    5. @annotation 匹配的是"方法上有没有这个注解"，注解没加的地方不会生效



---


# 五、连接点 JoinPoint

## 1. 概念

    Spring 中用 JoinPoint 抽象了连接点，
    通过它可以在通知里获得方法执行时的相关信息：
    目标类名、方法名、方法参数、返回值等。

## 2. 用哪个类型（必考）

    @Around 通知           只能用 ProceedingJoinPoint
    其它四种通知           只能用 JoinPoint（它是 ProceedingJoinPoint 的父类型）

    原因：只有 @Around 需要"控制目标方法是否执行"，
         所以要用子接口 ProceedingJoinPoint 拿到 proceed() 方法。

    写反了会编译/运行出错：@Before 里写 ProceedingJoinPoint 不合法。

## 3. 常用 API

    joinPoint.getTarget()                    获取目标对象
    joinPoint.getTarget().getClass().getName()   获取目标类名
    joinPoint.getSignature()                 获取目标方法签名（Signature）
    joinPoint.getSignature().getName()       获取目标方法名
    joinPoint.getArgs()                      获取目标方法的运行参数（Object[]）
    joinPoint.proceed()                      执行目标方法（仅 ProceedingJoinPoint 有）

## 4. 示例代码

@Around 环绕通知：
```java
@Around("execution(* com.itheima.service.DeptService.*(..))")
public Object around(ProceedingJoinPoint joinPoint) throws Throwable {

    String className = joinPoint.getTarget().getClass().getName();  // 目标类名
    Signature signature = joinPoint.getSignature();                 // 目标方法签名
    String methodName = joinPoint.getSignature().getName();         // 目标方法名
    Object[] args = joinPoint.getArgs();                            // 目标方法运行参数

    Object res = joinPoint.proceed();                               // 放行，执行目标方法
    return res;
}
```

其它四种通知（以 @Before 为例）：
```java
@Before("execution(* com.itheima.service.DeptService.*(..))")
public void before(JoinPoint joinPoint) {

    String className = joinPoint.getTarget().getClass().getName();  // 目标类名
    Signature signature = joinPoint.getSignature();                 // 目标方法签名
    String methodName = joinPoint.getSignature().getName();         // 目标方法名
    Object[] args = joinPoint.getArgs();                            // 目标方法运行参数
}
```

## 5. 易错点

    1. @Around 用 ProceedingJoinPoint，其它通知用 JoinPoint，不要混
    2. getArgs() 返回 Object[]，取具体值要按顺序取并强转类型
    3. getSignature() 拿到的是签名对象，方法名要再 .getName()
    4. 想拿方法上的注解：((MethodSignature) joinPoint.getSignature()).getMethod().getAnnotation(Xxx.class)
