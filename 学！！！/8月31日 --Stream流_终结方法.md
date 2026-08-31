# Stream 流的终结方法（4 个）

| 方法                   | 作用    | 返回值                |
| -------------------- | ----- | ------------------ |
| `forEach(Consumer)`  | 遍历    | void               |
| `count()`            | 统计数量  | long               |
| `toArray()`          | 收集到数组 | `Object[]`         |
| `collect(Collector)` | 收集到集合 | List / Set / Map 等 |
## 一、forEach：遍历

```java
//完整写法
list.stream().forEach(new Consumer<String>() {
    @Override
    public void accept(String s) {
    System.out.println(s);
    }
});

// Lambda 简化版（日常写法）
list.stream().forEach(s -> System.out.println(s));

// 方法引用（再简化一点点）
list.stream().forEach(System.out::println);
```

### 关键点
- 参数是 `Consumer<T>` 函数式接口
- 泛型 `T` 表示**流中数据的类型**（这里是 `String`）
- `accept` 方法的形参 `s` 依次代表"流里的每一个数据"
- 方法体里写对每个数据的处理逻辑（这里是打印）

## 二、count：统计元素数量

```java
long count = list.stream().count();
System.out.println(count);  // 9
```

### 关键点
- 返回值是 **`long`**，不是 `int`
- 无参数，直接调
- 统计的是**当前流里还剩多少元素**（前面如果有 `filter` / `limit`，就会少）


## 三、toArray：收集到数组

### 两种写法

```java
// ① 无参版 → 返回 Object[]（不常用）
Object[] arr1 = list.stream().toArray();

// ② 有参版 → 返回指定类型数组（推荐）
String[] arr = list.stream().toArray(new IntFunction<String[]>() {
    @Override
    public String[] apply(int value) {
    return new String[value];
    }
});;

// Lambda 简化版（日常写法）
String[] arr2 = list.stream().toArray(value -> new String[value]);

// 方法引用版（最简洁，推荐）
String[] arr3 = list.stream().toArray(String[]::new);
```

### IntFunction 关键点
- **泛型**：具体类型的数组（这里是 `String[]`）
- **apply 的形参 `value`**：流里元素的个数，要跟返回数组的长度一致
- **apply 的返回值**：那个具体类型的数组
- **方法体**：`return new String[value];` ← 你要做的就是创建一个数组

## 四、collect：收集到集合（最常用的终结方法）

### ① 收集到 List

需求：把所有男性收起来

```java
List<String> newList1 = list.stream()
    .filter(s -> "男".equals(s.split("-")[1]))   // 留下"男"的
    .collect(Collectors.toList());                // 收成 List
System.out.println(newList1);
```

输出：[张无忌-男-15, 张无忌-男-15, 赵敏-女-13... ← 只要男的]

---

### ② 收集到 Set（自动去重）

需求：把所有男性收起来（不要重复的）

```java
Set<String> newList2 = list.stream()
    .filter(s -> "男".equals(s.split("-")[1]))
    .collect(Collectors.toSet());
System.out.println(newList2);
```

输出：[张无忌-男-15, 张强-男-20, 张三丰-男-100, 张翠山-男-40, 张良-男-35]
       （"张无忌"那俩只剩1个 ← Set 自动去重）

### ③ toMap：收集到 Map（重点）
![652](../图片/Function%20函数的拆解.png)
泛型二是key或value的数据类型


```java
.collect(Collectors.toMap(
    new Function<String, String>() {      // 参数一：key 怎么来
        @Override
        public String apply(String s) {   // 张无忌-男-15
            return s.split("-")[0];       // → "张无忌"（key）
        }
    },
    new Function<String, Integer>() {    // 参数二：value 怎么来
        @Override
        public Integer apply(String s) {
            return Integer.parseInt(s.split("-")[2]); // → 15（value）
        }
    }
));

// Lambda 简化版
Map<String, Integer> map = list.stream()
    .collect(Collectors.toMap(
        s -> s.split("-")[0],                    // key：姓名
        s -> Integer.parseInt(s.split("-")[2])   // value：年龄
    ));
```

输出：`{张无忌=15, 周芷若=14, 赵敏=13, 张强=20, 张三丰=100, ...}`


# Stream 流 · 总结

## ① Stream 流的作用
**结合 Lambda 表达式，简化集合、数组的操作**

## ② Stream 的使用步骤（3 步）
1. 获取 Stream 流对象
2. 使用中间方法处理数据
3. 使用终结方法处理数据

## ③ 如何获取 Stream 流对象（4 种）

| 数据源 | 写法 |
|--------|------|
| 单列集合 | `Collection` 中的默认方法 `.stream()` |
| 双列集合（Map） | ❌ 不能直接获取（要 keySet/entrySet/values 转一下） |
| 数组 | `Arrays` 工具类中的静态方法 `Arrays.stream(数组)` |
| 一堆零散数据 | `Stream` 接口中的静态方法 `Stream.of(数据...)` |

## ④ 常见方法

| 分类 | 方法 |
|------|------|
| **中间方法**（链式调用、不执行） | `filter` / `limit` / `skip` / `distinct` / `concat` / `map` |
| **终结方法**（最后一步、真正执行） | `forEach` / `count` / `collect` |
