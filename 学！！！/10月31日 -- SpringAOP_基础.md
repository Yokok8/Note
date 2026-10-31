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
