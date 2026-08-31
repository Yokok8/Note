## Stream 流的作用
结合 Lambda 表达式，**简化集合、数组的操作**

## Stream 流的使用步骤（3 步）

1. **先得到一条 Stream 流（流水线）**，把数据放上去
2. **利用 Stream 中的 API** 进行各种操作
3. （终结后，输出结果）

## 中间方法 vs 终结方法（关键区分）

| 类型 | 操作 | 关键特性 |
|------|------|---------|
| **中间方法** | 过滤、转换（filter / map） | 方法调用完毕**还可以继续调**其他方法 |
| **终结方法** | 统计、打印（count / forEach） | **最后一步**，调用完毕**就不能再调**其他方法了 |

## 获取 Stream 流（4 种方式）

| 数据源                  | 方法                  | 来源                   |
| -------------------- | ------------------- | -------------------- |
| **单列集合**（List / Set） | `集合.stream()`       | `Collection` 接口的默认方法 |
| **双列集合**（Map）        | ❌ 无法直接用             | 要先转成单列集合             |
| **数组**               | `Arrays.stream(数组)` | `Arrays` 工具类的静态方法    |
| **一堆零散数据**           | `Stream.of(数据...)`  | `Stream` 接口的静态方法     |
### 一、 单列集合获取 Stream 流

### 两步走
 
1. `集合.stream()` → 拿到一条 Stream 流
2. `.forEach(...)` → 终结方法，把每个元素跑一遍

### 写法对比（Lambda 简化）

**传统（匿名内部类）**：
```java
stream1.forEach(new Consumer<String>() {
    @Override
    public void accept(String s) {
        System.out.println(s);
    }
});
```

**Lambda 简化**：
```java
list.stream().forEach(s -> System.out.println(s));
```

### 二、双列集合获取 Stream 流（间接拿）

Map **不能直接 .stream()**，要先转成单列集合：

```java
HashMap<String, Integer> hm = new HashMap<>();
hm.put("aaa", 111);
hm.put("bbb", 222);
hm.put("ccc", 333);
hm.put("ddd", 444);

// 第一种：流里是 key
hm.keySet().stream().forEach(s -> System.out.println(s));

// 第二种：流里是 Entry（键值对）—— 最常用
hm.entrySet().stream().forEach(s -> System.out.println(s));
```

### 三、数组获取 Stream 流

```java
int[] arr1 = {1,2,3,4,5,6,7,8,9,10};
String[] arr2 = {"a", "b", "c"};

Arrays.stream(arr1).forEach(s -> System.out.println(s));
Arrays.stream(arr2).forEach(s -> System.out.println(s));
```

### 四、零散数据获取 Stream 流

```java
Stream.of(1,2,3,4,5).forEach(s -> System.out.println(s));
Stream.of("a","b","c","d","e").forEach(s -> System.out.println(s));
```

### ⚠️ Stream.of 的坑（重要）

```java
int[] arr1 = {1, 2, 3, 4, 5};
Stream.of(arr1).forEach(s -> System.out.println(s));
// 输出：[@41629346] ← 整个数组被当作一个元素了！
```

#### 为什么会这样？

`Stream.of(T... values)` 的 T 是**泛型**，而泛型**不能存基本数据类型**。

所以传 `int[]` 时，JDK 把整个 `int[]`数组当成 **一个 Object 元素**塞进了 Stream，流里就1 个东西（数组本身）。

