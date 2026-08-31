# Stream 流的中间方法（6 个）

| 方法 | 作用 | 关键点 |
|------|------|--------|
| `filter(条件)` | 过滤 | 参数是 Predicate（返回 boolean 的 lambda） |
| `limit(n)` | 取前 n 个 | long |
| `skip(n)` | 跳前 n 个 | long |
| `distinct()` | 去重 | 无参，依赖 `hashCode + equals` |
| `concat(a, b)` | 合并两个流 | 静态方法，参数是两个 Stream |
| `map(转换函数)` | 转换数据类型 | 参数是 `Function<T, R>` |

## ⚠️ 两个重要注意

### ① Stream 流只能使用一次

```java
Stream<String> s1 = list.stream();
s1.filter(x -> x.length() > 3);
s1.filter(x -> x.startsWith("张"));  // ❌ 报错！流已被消费
```

所以**强烈建议链式编程**：

```java
list.stream()
 .filter(x -> x.length() > 3)
    .filter(x -> x.startsWith("张"))
    .forEach(System.out::println);
```

### ② 不影响原数据

Stream 上的一切操作都在**副本**上进行，原集合 /数组**纹丝不动**。


## 一、filter：过滤（留下你想要的）
没简化的：
![376](../图片/Stream%20filter：过滤.png)

```java
list.stream()
 .filter(s -> s.startsWith("张"))    // 留下"张"开头的
    .forEach(s -> System.out.println(s));
```
### 关键点
- `filter` 的参数是 `Predicate<T>` 函数式接口
- 返回 **true** → 当前数据**留下**
- 返回 **false** → 当前数据**丢弃**
- Lambda 简化成：`s -> 条件`

## 二、limit & skip（一对兄弟）

```java
list.stream().limit(3).forEach(s -> System.out.println(s));  // 取前 3 个
list.stream().skip(4).forEach(s -> System.out.println(s));   // 跳过前 4 个
```

- `limit(n)`：**取**前 n 个
- `skip(n)`：**跳**过前 n 个

## 三、distinct & concat

### ① distinct：元素去重

```java
list1.stream().distinct().forEach(s -> System.out.println(s));
```

> ⚠️ distinct 依赖 **`hashCode + equals`** 判断"是不是同一个"，所以自定义对象要去重，得重写这俩方法。

### ② concat：合并两个流

```java
Stream.concat(list1.stream(), list2.stream()).forEach(s -> System.out.println(s));
```

合并顺序：**a 在前、b 在后**。注意 `concat` 是 `Stream` 接口的**静态方法**，所以 `Stream.xxx` 调用。


## 四、map：转换流中的数据类型

### 经典场景

数据是 `"张无忌-15"` 这种"姓名-年龄"格式，要**只取出年龄**打印。
```java
list.stream().map(new Function<String, Integer>() {
    @Override
    public Integer apply(String s) {
    String[] arr = s.split(regex: "-"); //第一个数据split切割成["张无忌", "15"]
    String ageString = arr[1];//取年龄的索引
    int age = Integer.parseInt(ageString);//转换成int类型
    return age;
    }
}).forEach((s -> System.out.println(s));
```

```java
// Lambda 简化版（一行搞定）
list.stream()
    .map(s -> Integer.parseInt(s.split("-")[1]))   // String → Integer
    .forEach(s -> System.out.println(s));
```

**输出**：15 14 13 20 100 40 35 37（全部是整数）



