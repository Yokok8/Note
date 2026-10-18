# 员工模块 · 准备工作

## 一、三步准备（课件原文）
1. 准备**数据库表**：emp、emp_expr
2. 准备**实体类**：Emp、EmpExpr
3. 准备**三层架构**的基本代码结构：EmpController、EmpService / EmpServiceImpl、EmpMapper

三件事一一对应：

| 层 | 内容 | 对应关系 |
|---|---|---|
| 数据层 | emp、emp_expr 两张表 | ↓ |
| 实体层 | Emp、EmpExpr 两个类 | 表和类**一一对应** |
| 代码层 | Controller / Service / ServiceImpl / Mapper | 一套完整链路 |

## 二、两张表的分工
| 表 | 存什么 | 关系 |
|---|---|---|
| **emp** | 员工基本信息（姓名、性别、头像、职位、入职日期、dept_id…） | 与 dept 是**一对多**（dept_id 外键） |
| **emp_expr** | 员工的**工作经历**（一段经历一行） | 与 emp 是**一对多**（一个员工多段经历） |

所以员工模块天然是**三表关联**：emp ↔ dept、emp ↔ emp_expr。

## 三、⭐ 原型上那个绿框在提示什么
课件把「**所属部门**」这一列单独用绿框高亮了，意思很明确：
- **这一列的值不是 emp 表里存的**
- emp 表里只有 `dept_id`，页面上要显示"学工部""教研部"这些**中文名**
- → 必须 **join dept 表** 查出来

同理，页面上"职位"列显示"班主任/讲师"，但库里存的是数字 job（1/2/3/4），**是后端翻译成文字给前端的**。
**规律：原型上出现"人类可读的名字"，背后往往藏着 id + 关联表。**&#8203;

## 四、实体类对应
    public class Emp {
        private Integer id;
        private String username;
        private String password;
        private String name;
        private Short gender;
        private String image;
        private Short job;
        private LocalDate entryDate;
        private Integer deptId;      // 对应表里的 dept_id
        private LocalDateTime createTime;
        private LocalDateTime updateTime;

        // private String deptName;  // 多表查询出来的部门名（不在表里）
        // private List<EmpExpr> exprList;  // 一对多的工作经历（后面会加）
    }

    public class EmpExpr {
        private Integer id;
        private Integer empId;       // 外键，关联 emp.id
        private LocalDate begin;
        private LocalDate end;
        private String company;
        private String job;
    }

**表和类一一对应，但类里可以多出"表里没有的字段"**&#8203;（比如 deptName、exprList），用来装 join 查出来的数据。

## 五、三层代码骨架
    EmpController      → 接请求，返回 Result
    EmpService（接口）  → 定方法规范
    EmpServiceImpl     → 业务实现（补时间、组装数据）
    EmpMapper          → 写 SQL（多表查询就在这层）

## 六、易错点
| 易错 | 说明 |
|---|---|
| 只建 emp 表漏了 emp_expr | 工作经历查不出来，新增员工时也存不了 |
| 实体类字段名和表字段对不上 | 要么开驼峰映射，要么字段名写对 |
| 以为部门名存在 emp 表 | 存的是 dept_id，名字靠 join 或翻译 |
| 一层代码只建了 Controller | 三层要全建，否则调不通 |
| 忘了性别/职位是数字 | 实体类用 Short/Integer，不要用 String |

## 七、口诀
- **准备三件套：表 → 实体类 → 三层代码**
- **emp 关联 dept，emp 关联 emp_expr，员工模块是三表联动**
- **原型上的中文名（部门名/职位名），背后都是 id + 关联表**

---

# 分页查询 · 分析

## 一、页数是怎么算的（课件红字）
每页显示记录数 10：
- 共 **500** 条：500 / 10 = **50 页**
- 共 **501** 条：501 / 10 = 50.1 → **51 页**（**向上取整**）

所以前端要显示"共 51 页"，必须知道两个数：**每页几条** + **总共几条**。

## 二、⭐ 前端传给后端什么（只有 2 个参数）
| 参数 | 含义 | 例 |
|---|---|---|
| **page** | 页码（第几页） | 1、2、3… |
| **pageSize** | 每页显示记录数 | 10 |

## 三、⭐ 后端给前端返回什么（只有 2 个字段）
| 字段 | 含义 | 从哪来 |
|---|---|---|
| **total** | **总记录数**（Long） | `select count(*) from emp e ...;` |
| **rows** | **当前页数据**（List） | `select * from emp e ... limit ?,?;` |

**一个分页接口 = 两条 SQL + 一个封装类。**&#8203;

## 四、两条 SQL 长什么样
    -- ① 查总记录数（前端要算总页数）
    select count(*) from emp;

    -- ② 查当前页的数据（只取这一页那 10 条）
    select * from emp limit 0, 10;    -- 第 1 页：跳过 0 条，取 10 条
    select * from emp limit 10, 10;   -- 第 2 页：跳过 10 条，取 10 条

limit 的公式：**起始索引 = (page - 1) × pageSize**

## 五、PageResult 封装类（⭐ 泛型）
    @Data
    public class PageResult<T> {
        private Long total;   // 总记录数
        private List<T> rows; // 当前页数据
    }

**为什么要用泛型 `<T>`**：
- 部门分页 → `PageResult<Dept>`
- 员工分页 → `PageResult<Emp>`
- 班级分页 → `PageResult<Clazz>`

一个类就能复用所有模块，不用为每张表写一版。

## 六、完整流程串一遍
1. 前端传 `page=1&pageSize=10`
2. 后端算起始索引：`(1-1) × 10 = 0`
3. 执行 `select count(*)` → 得 total=500
4. 执行 `select ... limit 0, 10` → 得 10 条员工
5. 封装 `new PageResult(total=500, rows=这10条)`
6. 包进 Result 返回：`{"code":1,"data":{"total":500,"rows":[...]}}`
7. 前端算出总页数 50，渲染分页条

## 七、易错点
| 易错 | 说明 |
|---|---|
| 只写了 limit，忘了 count | 前端算不出总页数，分页条显示不出来 |
| 起始索引忘了 -1 | `page × pageSize` 会**跳过第一页数据** |
| total 用 int | 数据量大时会溢出，用 **Long** |
| PageResult 写死成 Dept | 用泛型，全模块复用 |
| 两条 SQL 查的条数对不上 | 条件必须一致（where 条件两条都要带） |

## 八、口诀
- **前端给 page + pageSize，后端还 total + rows**
- **两条 SQL：count 查总数，limit 查当前页**
- **起始索引 = (page - 1) × pageSize**
- **PageResult 加泛型，一个类管所有模块**

---

# 分页查询 · 原始方式

## 一、三层各自的职责（课件图）
| 层 | 职责 |
|---|---|
| **Controller** | ① 接收参数（分页）② 调用 Service 进行分页查询，获取 PageResult ③ 响应结果 |
| **Service** | ① 调用 Mapper 查**总记录数** ② 调用 Mapper 查**结果列表** ③ **封装 PageResult 对象**返回 |
| **Mapper** | 两条 SQL：① select count(*) ... ② select ... limit ... |

请求 `/emps?page=1&pageSize=10` 从浏览器一路走到数据库，再一路返回 —— 和之前部门管理完全一样的链路，只是 Service 里多了"两条 SQL + 封装"。

## 二、EmpController
    @GetMapping
    public Result page(@RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(defaultValue = "10") Integer pageSize) {
        log.info("分页请求参数： {}, {}", page, pageSize);
        PageResult<Emp> pageResult = empService.page(page, pageSize);
        return Result.success(pageResult);
    }

⭐ **`@RequestParam(defaultValue = "1")`**：前端不传这个参数时用默认值。
- page 默认 1（第一页）
- pageSize 默认 10（每页 10 条）

这样前端即使什么参数都不传，接口也不会报错。

## 三、EmpServiceImpl（核心逻辑）
    public PageResult<Emp> page(Integer page, Integer pageSize) {
        // 1. 获取总记录数
        Long count = empMapper.count();

        // 2. 获取每一页的数据列表
        Integer start = (page - 1) * pageSize;
        List<Emp> empList = empMapper.list(start, pageSize);

        // 3. 封装分页结果
        return new PageResult<Emp>(count, empList);
    }

三步必须按顺序：
1. **先查总数**（前端要算总页数）
2. **再算起始索引** `(page - 1) * pageSize`，查当前页数据
3. **最后封装** PageResult

## 四、EmpMapper
    @Mapper
    public interface EmpMapper {

        @Select("select count(*) from emp e left join dept d on e.dept_id = d.id")
        public Long count(); // 查询总记录数

        @Select("select e.*, d.name deptName from emp e left join dept d on e.dept_id = d.id limit #{start},#{pageSize}")
        public List<Emp> list(Integer start, Integer pageSize); // 查询结果列表
    }

两个关键细节：

### 1. `d.name deptName` —— 起别名
- dept 表里字段叫 `name`，Emp 实体类里叫 `deptName`
- 直接 `select d.name` 映射不到 deptName，**必须起别名 `d.name deptName`**
- 这样查询结果才能自动封装进 `Emp.deptName`

### 2. 两条 SQL 的 where 条件必须一致
count 用 `left join dept`，list 也必须用 `left join dept`。
否则总数和当前页数据**对不上**（比如按部门筛了，总数却没筛）。

### 3. `limit #{start},#{pageSize}` —— 两个参数
- 第一个是**起始索引**（跳过多少条）
- 第二个是**取多少条**
- MyBatis 多参数要一一对应，所以 Service 传了 `(start, pageSize)`

## 五、数据流完整串一遍
    GET /emps?page=1&pageSize=10
      ↓ Controller 接参（defaultValue 兜底）
      ↓ Service：count() → 500
      ↓ Service：start = (1-1)*10 = 0，list(0, 10) → 10 条
      ↓ new PageResult(500, 10条)
      ↓ Controller：Result.success(pageResult)
    → {"code":1,"data":{"total":500,"rows":[...10条...]}}

## 六、易错点
| 易错 | 说明 |
|---|---|
| 忘了 defaultValue | 前端不传参数直接 400 报错 |
| `d.name` 没起别名 | deptName 一直是 null（**静默失败**） |
| start 算成 `page * pageSize` | 第一页数据被跳过 |
| 两条 SQL 条件不一致 | total 和 rows 数量对不上，分页条乱 |
| `limit #{start},#{pageSize}` 顺序颠倒 | 变成取第 pageSize 页的 start 条，全乱 |
| EmpMapper 漏了 @Mapper | 启动报找不到 Bean |

## 七、口诀
- **Controller 接参（给默认值）→ Service 查两次 → Mapper 两条 SQL**
- **Service 三步：count、list(start, pageSize)、封装 PageResult**
- **多表查询给字段起别名，别名要和实体类属性名一致**
- **两条 SQL 的条件必须一模一样**

---

# 员工管理 - 分页查询（PageHelper 插件）

## 一、PageHelper 是什么

- 定义：**第三方提供的、在 MyBatis 框架中用来实现分页的插件**
- 作用：**简化分页操作，提高开发效率**
- 官网：https://pagehelper.github.io/
- 对比上一节的"原始方式"：原来要自己写 count 查总数、自己算 `start = (page-1)*pageSize`、自己拼 `limit`，用了插件后这些全都不用写

## 二、使用三步（核心）

### 第①步：pom.xml 引入依赖

    <dependency>
        <groupId>com.github.pagehelper</groupId>
        <artifactId>pagehelper-spring-boot-starter</artifactId>
        <version>1.4.7</version>
    </dependency>

- 注意是 `pagehelper-spring-boot-starter`（SpringBoot 的 starter 版本），**不是** 原生的 `pagehelper`
- starter 版本会自动完成配置（拦截器注册），引入即用，**不需要额外写任何配置**

### 第②步：Mapper 方法无需考虑分页

    @Select("select e.*, d.name deptName from emp e left join dept d on e.dept_id = d.id order by e.update_time desc")
    public List<Emp> list();

- **Mapper 的 SQL 里绝对不要写 `limit`**，就当查全部数据来写
- 返回类型还是 `List<Emp>`，不用改
- 原来那条 `count(*)` 的方法**可以删掉了**，插件会自动帮你查总数

### 第③步：Service 中调用 PageHelper 完成分页

    @Override
    public PageResult<Emp> page(Integer page, Integer pageSize) {
        // 1. 设置分页参数（必须在查询方法之前调用）
        PageHelper.startPage(page, pageSize);

        // 2. 调用 Mapper 方法查询（紧跟在 startPage 后面）
        List<Emp> empList = empMapper.list();

        // 3. 强转成 Page 类型，取出总记录数和当前页数据
        Page<Emp> p = (Page<Emp>) empList;

        // 4. 封装返回
        return new PageResult<>(p.getTotal(), p.getResult());
    }

- 需要导包：`import com.github.pagehelper.Page;`、`import com.github.pagehelper.PageHelper;`
- `PageHelper.startPage(page, pageSize)` 的参数顺序：**第一个是页码，第二个是每页条数**，不要写反

## 三、核心原理（重点理解）

- `PageHelper.startPage(page, pageSize)` 会**拦截紧随其后的第一条 SQL**
- 拦截后插件自动做两件事：
    1. 在 SQL 末尾**自动拼接 `limit ?, ?`**（参数根据 page / pageSize 算出来）
    2. **额外自动执行一次 `select count(*)`** 查总记录数
- 所以你写一条 SQL，插件帮你执行了两条（分页查询 + 总数查询）

### 为什么能强转成 Page？

- `Page<E> extends ArrayList<E>`，而 `ArrayList<E> implements List<E>`
- 所以 `Page<E>` 也是一个 `List<E>`，插件就是把返回的 List 偷偷换成了 Page 对象
- 即：`empMapper.list()` 表面返回 `List<Emp>`，**实际类型是 `Page<Emp>`**
- 强转后 Page 提供了两个额外方法：
    - `getTotal()`：总记录数（插件额外查出来的）
    - `getResult()`：当前页的数据列表

## 四、原始方式 vs PageHelper（对比速查）

| 对比项 | 原始方式 | PageHelper |
|---|---|---|
| 依赖 | 无（自己写） | 加 pagehelper-spring-boot-starter |
| Mapper 方法数 | 2 个（`count()` + `list()`） | **1 个**（只写 `list()`） |
| SQL 里写 limit | 要自己写 `limit #{start},#{pageSize}` | **不写**，插件自动拼 |
| 计算起始索引 | 自己算 `start = (page-1)*pageSize` | **不算**，插件自动算 |
| 取总记录数 | `count()` 查出来的 | `(Page)list.getTotal()` |
| Service 代码量 | 多 | 少，就 4 行 |

## 五、完整三层代码（PageHelper 版）

### EmpController

    @GetMapping
    public Result page(@RequestParam(defaultValue = "1") Integer page,
                       @RequestParam(defaultValue = "10") Integer pageSize) {
        PageResult<Emp> pageResult = empService.page(page, pageSize);
        return Result.success(pageResult);
    }

### EmpService / EmpServiceImpl

    public PageResult<Emp> page(Integer page, Integer pageSize);

    @Override
    public PageResult<Emp> page(Integer page, Integer pageSize) {
        PageHelper.startPage(page, pageSize);
        List<Emp> empList = empMapper.list();
        Page<Emp> p = (Page<Emp>) empList;
        return new PageResult<>(p.getTotal(), p.getResult());
    }

### EmpMapper

    @Select("select e.*, d.name deptName from emp e left join dept d on e.dept_id = d.id order by e.update_time desc")
    public List<Emp> list();

### PageResult（复用上一节封装的泛型类）

    @Data
    @NoArgsConstructor
    @AllArgsConstructor
    public class PageResult<T> {
        private Long total;      // 总记录数
        private List<T> rows;    // 当前页数据列表
    }

## 六、易错点（必看）

- ❌ `PageHelper.startPage()` 和 Mapper 查询之间**不要插入别的查询**，否则分页会作用在错误的 SQL 上（只对紧随其后的**第一条**生效）
- ❌ 忘了强转 `Page<Emp>` → 拿不到 `getTotal()`，只能用 `empList.size()` 得到**当前页条数**（不是总数）
- ❌ Mapper 的 SQL 里自己又写了 `limit` → 和插件拼的 limit 冲突重复
- ❌ `startPage(page, pageSize)` 参数写反 → 页码和每页条数颠倒，结果全乱
- ❌ 用了 PageHelper 但 Mapper 还是留着 `count()` 方法 → 多余，删掉即可
- ⚠️ **分页查询必须保证线程安全**：`startPage` 用的是 ThreadLocal，所以只能在当前线程内紧跟调用，不能跨线程/异步使用

## 七、口诀

> **一引依赖、二删 limit、三 startPage**
> **紧跟第一条，强转 Page 取 total**

---

# PageHelper 总结页 

## 1. 实现机制（两条 SQL）

| | SQL 形态 | 作用 |
|---|---|---|
| a | `select count(0) from emp e ...` | 查总记录数 total |
| b | `select ... from emp e ... limit ?,?` | 查当前页数据 rows |

- 我代码里只写了一条 b 的原文，a 是插件自动生成的
- 拼接顺序：**先 count，后 limit**

## 2. 注意事项（重点）

| 序号 | 注意点 | 踩坑后果 |
|---|---|---|
| a | **SQL 语句结尾不要加分号 `;`** | 插件在末尾拼 limit 会变成 `...desc; limit ?,?` → SQL 语法错误 |
| b | **只对紧跟在其后的第一条 SQL 分页** | 中间插了别的查询 → 分页作用到错误的 SQL 上，原查询没有 limit |

## 3. 规避写法

    // ✅ 正确：无分号，Mapper 调用紧跟 startPage
    PageHelper.startPage(page, pageSize);
    List<Emp> empList = empMapper.list();

    // ❌ 错误示例1：SQL 结尾带分号
    @Select("select e.* from emp e ... order by e.update_time desc;")

    // ❌ 错误示例2：中间插了别的查询
    PageHelper.startPage(page, pageSize);
    Long count = empMapper.count();   // ← 分页被它吃掉了
    List<Emp> empList = empMapper.list();

## 4. 口诀
> **count 打头、limit 收尾；结尾无分号，紧跟第一条**
