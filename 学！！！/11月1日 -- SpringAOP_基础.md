# 一、什么是 AOP

## 1. 概念

    AOP：Aspect Oriented Programming，面向切面编程 / 面向方面编程
    简单理解：面向特定方法编程。

    更直白一点：在不修改原方法代码的前提下，给一批方法统一加上额外功能。
    例如一批方法都需要记录日志、统计耗时，代码只写一份，框架负责"织"进去。

## 2. 为什么需要 AOP（课程场景）

    需求：案例中部分业务方法运行较慢，要定位执行耗时较长的接口，
         因此需要统计每一个业务方法的执行耗时。

    笨办法：每个方法开头记开始时间、结尾记结束时间、相减、打印日志。
            几十个方法就要写几十遍，重复且难维护。

    AOP 办法：写一份"统计耗时"的逻辑，声明它作用在哪些方法上，
              Spring 自动把它织入到这些方法的前后。

## 3. 优势（四条，背下来）

    1. 减少重复代码      公共逻辑只写一份
    2. 代码无侵入        原业务方法一行都不用改
    3. 提高开发效率      加功能不用逐个方法改
    4. 维护方便          改一处，所有被织入的方法同步生效

## 4. 关键提示

    AOP 是一种编程思想（思想本身不属于 Spring）；
    Spring AOP 是这种思想在 Spring 框架中的具体实现。
    本课程要学的是 Spring AOP。

    补充：Spring AOP 底层靠动态代理实现，这也是它能在不改源码的情况下增强方法的原因。

## 5. 典型应用场景

    日志记录、执行耗时统计、事务管理、权限校验、缓存处理
    共同特点：都是"和业务无关、但多个方法都要用"的横切逻辑。


---


# 二、AOP 快速入门 - 统计业务方法执行耗时

## 1. 需求

统计所有业务层方法的执行耗时，找出慢接口。

## 2. 原始方式 vs AOP 方式

原始方式（每个方法自己写，代码侵入）：
```java
public List<Dept> list() {
    long beginTime = System.currentTimeMillis();      // 开始时间
    List<Dept> deptList = deptMapper.list();          // 原始业务
    long endTime = System.currentTimeMillis();        // 结束时间
    log.info("执行耗时: {} ms", endTime - beginTime);  // 计算耗时
    return deptList;
}

public void delete(Integer id) {
    long beginTime = System.currentTimeMillis();
    deptMapper.delete(id);
    long endTime = System.currentTimeMillis();
    log.info("执行耗时: {} ms", endTime - beginTime);
}
// 其他方法省略...  每个方法都要重复这三段
```

AOP 方式（业务方法保持干净，一行不改）：
```java
public List<Dept> list() {
    List<Dept> deptList = deptMapper.list();
    return deptList;
}

public void delete(Integer id) {
    deptMapper.delete(id);
}
// 其他方法省略...
```

对比结论：原始方式改动面大、重复代码多；AOP 把"计时"这段横切逻辑抽出来写一次。

## 3. 实现两步

第一步：导入依赖（pom.xml）
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

第二步：编写 AOP 程序
```java
@Aspect
@Component
public class RecordTimeAspect {

    @Around("execution(* com.itheima.service.impl.*.*(..))")
    public Object recordTime(ProceedingJoinPoint pjp) throws Throwable {

        long beginTime = System.currentTimeMillis();   // 目标方法执行前

        Object result = pjp.proceed();                // 调用原始方法

        long endTime = System.currentTimeMillis();    // 目标方法执行后
        log.info("执行耗时: {} ms", endTime - beginTime);

        return result;                                // 必须把结果返回
    }
}
```

## 4. 注解与表达式拆解

    @Aspect        标识这是一个切面类
    @Component     交给 Spring 容器管理（IOC）
    @Around        环绕通知：目标方法前后都能写逻辑
    ProceedingJoinPoint.proceed()  执行原始的目标方法

    execution(* com.itheima.service.impl.*.*(..))
        返回值任意   包名（固定）              类名任意 方法名任意 参数任意

## 5. 易错点

    1. pjp.proceed() 抛的是 Throwable，方法必须声明 throws Throwable
    2. 忘记 return result，Controller 收到的就是 null
    3. @Aspect 和 @Component 缺一不可：少 @Component 切面不生效
    4. 表达式包名写错（如写成 service 而非 service.impl），切面静默失效，不会报错
    5. 别忘了 spring-boot-starter-aop 依赖


---


# 三、AOP 核心概念

## 1. 五个概念（原文定义，背下来）

    连接点 JoinPoint    可以被 AOP 控制的方法（暗含方法执行时的相关信息）
    通知 Advice         那些重复的逻辑，也就是共性功能（最终体现为一个方法）
    切入点 PointCut     匹配连接点的条件，通知仅在切入点方法执行时被应用
    切面 Aspect         描述通知与切入点的对应关系（通知 + 切入点）
    目标对象 Target     通知所应用的对象

## 2. 落到代码上对应谁（对照图右侧 DeptServiceImpl）

    Target      →  DeptServiceImpl（被增强的类）
    连接点      →  list()、delete()、save()、getById() 这四个方法
    切入点      →  execution(* com.itheima.service.impl.*.*(..))
    通知        →  recordTime() 方法体里那段计时代码
    切面        →  @Around + 切入点表达式 + 通知方法，三者合起来
    切面类      →  RecordTimeAspect

## 3. 关键区分（考点，别混）

    连接点 vs 切入点
        连接点：所有"可以被 AOP 控制"的方法，范围大
        切入点：用表达式筛选后真正被增强的方法，范围小
        关系：切入点 ⊆ 连接点

    通知 vs 切面
        通知：只是一个方法（干什么）
        切面：通知 + 切入点（干什么 + 在哪干）

## 4. 一个类比记住全部

    目标对象   要监控的那栋楼
    连接点     楼里所有能装摄像头的位置（每个方法都是一个位置）
    切入点     最终选定要装的几个位置（表达式筛出来的）
    通知       摄像头本身，负责录像那段逻辑
    切面       "在哪些位置装摄像头"的完整方案

## 5. 易错点

    1. 连接点不是切入点，别把"所有方法"当成"被增强的方法"
    2. 通知最终必须是一个方法，@Around 只是标记它的类型
    3. 切面 = 通知 + 切入点，缺一个都不成立


---

# 四、AOP 执行流程 - 动态代理

## 1. 结论先摆出来

    运行时，Spring 会为目标对象动态生成一个"代理对象"，
    代理对象和目标对象实现同一个接口，
    然后把代理对象放进容器。Controller 拿到的一直是代理对象，
    所以我们调用业务方法时，实际先进入代理，再进目标方法。

## 2. 图里三个角色

    目标对象 Target     DeptServiceImpl       原始业务代码，干干净净（只查数据库）
    代理对象 Proxy      DeptServiceImplProxy  运行时生成，方法体 = 通知 + 调目标对象
    Controller          注入 DeptService       实际注入的是代理对象，不是原始实现

## 3. 一次调用的完整链路

    Controller 调 deptService.list()
        ↓
    进入代理对象 list()          ← 因为注入的是代理
        ↓
    记录 beginTime               ← 通知的前置逻辑
        ↓
    调用目标对象.list()           ← 原始业务逻辑（deptMapper.list()）
        ↓
    记录 endTime、打印耗时         ← 通知的后置逻辑
        ↓
    return 结果 给 Controller

## 4. 代理对象内部大概长这样

```java
public class DeptServiceImplProxy implements DeptService {

    private DeptServiceImpl target;   // 持有目标对象

    @Override
    public List<Dept> list() {
        long begin = System.currentTimeMillis();        // 通知前
        List<Dept> deptList = target.list();            // 调用原始方法
        long end = System.currentTimeMillis();          // 通知后
        log.info("执行耗时: {} ms", end - begin);
        return deptList;
    }
}
```

    这也解释了为什么业务代码一行没改，耗时却被打出来了 ——
    真正被调用的是代理，业务代码只是被代理"包"了一层。

## 5. 必须理解的三个点

    1. 代理对象与目标对象实现同一接口，所以 Controller 按接口注入感知不到差别
    2. 没有接口时，Spring 改用 CGLIB 生成目标对象的子类作为代理
    3. AOP 生效的前提是"调用经过代理"：
       同类内部方法直接互调（this.xxx()）不经过代理，通知不会执行

## 6. 易错点

    1. 别以为 AOP 是"自动改写了源码"，源码没变，变的是容器里的对象
    2. Controller 注入的类型写接口(DeptService)，写实现类(DeptServiceImpl)可能报错
    3. 自调用失效是高频面试题，记住：代理只拦"从外部进来的调用"
