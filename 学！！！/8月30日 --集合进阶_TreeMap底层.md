# TreeMap 源码解读笔记

> **核心一句话**：TreeMap 底层是**红黑树**，依靠键的大小关系来决定节点位置，既能去重、又能自动排序。

---

## 一、基础认知：节点与类结构

### 1. 每个节点（Entry）的内部属性

| 属性 | 说明 |
| --- | --- |
| `K key` | 键 |
| `V value` | 值 |
| `Entry left` | 左子节点 |
| `Entry right` | 右子节点 |
| `Entry parent` | 父节点 |
| `boolean color` | 节点颜色（红/黑） |

> 比普通二叉树多了 `parent` 和 `color`，这是**红黑树**的标志。

```java
// TreeMap 中每一个节点的内部属性
K key;               // 键
V value;             // 值
Entry<K,V> left;     // 左子节点
Entry<K,V> right;    // 右子节点
Entry<K,V> parent;   // 父节点
boolean color;       // 节点的颜色
```

---

### 2. TreeMap 类的关键成员变量

| 变量 | 类型 | 说明 |
| --- | --- | --- |
| `comparator` | `Comparator` | 比较器对象，决定排序规则 |
| `root` | `Entry` | 根节点 |
| `size` | `int` | 集合元素个数 |

- `comparator` 为 `null` 时 → 使用**自然排序**（键需实现 `Comparable`）
- `comparator` 不为 `null` 时 → 使用**比较器排序**（自定义规则）

```java
public class TreeMap<K,V> {

    // 比较器对象
    private final Comparator<? super K> comparator;

    // 根节点
    private transient Entry<K,V> root;

    // 集合的长度
    private transient int size = 0;
}
```

---

### 3. 两种构造方法

| 构造方法 | comparator 值 | 排序方式 |
| --- | --- | --- |
| `new TreeMap()` | `null` | 自然排序 |
| `new TreeMap(Comparator)` | 传入的比较器 | 比较器排序 |

```java
// 3. 空参构造：没有传递比较器对象，使用自然排序
public TreeMap() {
    comparator = null;
}

// 4. 带参构造：传递了比较器对象，使用比较器排序
public TreeMap(Comparator<? super K> comparator) {
    this.comparator = comparator;
}
```

---

## 二、核心流程：添加元素 put

### 调用链路

```
put(key, value)
  └── put(key, value, replaceOld=true)   // replaceOld: 键重复时是否覆盖
        ├── 第一次添加 → addEntryToEmptyMap()  → 作为根节点
        └── 非首次添加 → 找位置 + addEntry() + fixAfterInsertion()
```

```java
// 5. 添加元素
public V put(K key, V value) {
    return put(key, value, true);
}

// 参数一：键
// 参数二：值
// 参数三：当键重复的时候，是否需要覆盖值
//         true：覆盖    false：不覆盖
```

---

### 第一步：判断根节点

- `root == null` → 第一次添加，当前元素作为根节点，直接返回
- `root != null` → 非首次添加，进入查找插入位置的流程

```java
private V put(K key, V value, boolean replaceOld) {
    // 获取根节点的地址值，赋值给局部变量 t
    Entry<K,V> t = root;

    // 判断根节点是否为 null
    // 如果为 null，表示当前是第一次添加，会把当前元素当做根节点
    // 如果不为 null，表示当前不是第一次添加，继续执行下面的代码
    if (t == null) {
        // 方法的底层，会创建一个 Entry 对象，把他当做根节点
        addEntryToEmptyMap(key, value);
        // 表示此时没有覆盖任何的元素
        return null;
    }

    // 表示两个元素的键比较之后的结果
    int cmp;
    // 表示当前要添加节点的父节点
    Entry<K,V> parent;
    // ...后续逻辑见下文
}
```

---

### 第二步：找到插入位置（二叉查找树规则）

根据是否有比较器，分两条路径，但**比较逻辑完全一样**：

```
循环（从根节点开始往下找）：
  cmp = 当前键 与 节点键 比较结果
  cmp < 0  → 往左子节点走
  cmp > 0  → 往右子节点走
  cmp == 0 → 键重复！
            若 replaceOld 为 true → 覆盖旧值，返回旧值
```

```java
    // 表示当前的比较规则
    // 自然排序：comparator 为 null，cpr 也为 null
    // 比较器排序：comparator 记录的就是比较器
    Comparator<? super K> cpr = comparator;

    // 判断当前是否有比较器对象
    // 传递了比较器 → 执行 if，以比较器规则为准
    // 没传递比较器 → 执行 else，以自然排序规则为准
    if (cpr != null) {
        // ===== 比较器排序 =====
        do {
            parent = t;
            cmp = cpr.compare(key, t.key);
            if (cmp < 0)
                t = t.left;       // 往左找
            else if (cmp > 0)
                t = t.right;      // 往右找
            else {
                // cmp == 0，键重复，覆盖逻辑
                V oldValue = t.value;
                if (replaceOld || oldValue == null) {
                    t.value = value;
                }
                return oldValue;
            }
        } while (t != null);
    } else {
        // ===== 自然排序 =====
        // 把键强转成 Comparable 类型
        // 要求：键必须实现 Comparable 接口，否则强转报错
        Comparable<? super K> k = (Comparable<? super K>) key;
        do {
            // 把根节点当做当前节点的父节点
            parent = t;
            // 调用 compareTo 方法比较大小
            cmp = k.compareTo(t.key);

            if (cmp < 0)
                t = t.left;       // 比较结果为负数，往左找
            else if (cmp > 0)
                t = t.right;      // 比较结果为正数，往右找
            else {
                // 比较结果为 0，键重复，覆盖
                V oldValue = t.value;
                if (replaceOld || oldValue == null) {
                    t.value = value;
                }
                return oldValue;
            }
        } while (t != null);
    }

    // 把当前节点按照指定的规则进行添加
    addEntry(key, value, parent, cmp < 0);
    return null;
}
```

---

### 第三步：创建新节点并挂载

```java
private void addEntry(K key, V value, Entry<K, V> parent, boolean addToLeft) {
    Entry<K,V> e = new Entry<>(key, value, parent);
    if (addToLeft)
        parent.left = e;        // 挂在左边
    else
        parent.right = e;       // 挂在右边

    // 添加完毕之后，需要按照红黑树的规则进行调整
    fixAfterInsertion(e);
    size++;
    modCount++;
}
```

---

## 三、红黑树调整：fixAfterInsertion（重点）

### 前置：为什么要调整？

新节点默认是**红色**。如果父节点也是红色，就违反了红黑树规则（不能有两个连续红色节点），必须调整。

### 调整的三大手段

| 手段 | 作用 |
| --- | --- |
| **变色** | 父、叔变黑，爷变红 |
| **左旋** | 调整结构平衡 |
| **右旋** | 调整结构平衡 |

### 调整逻辑（按情况分类）

先判断：**父节点是爷爷的左孩子还是右孩子？**（目的是找到叔叔节点）

```
while (x 非根 且 父节点为红色):

  情况A：父节点是爷爷的左孩子 → 叔叔 = 爷爷的右孩子
    ├── 叔叔为红色 → 变色：父黑、叔黑、爷红，x 跳到爷爷继续
    └── 叔叔为黑色（或null）
        ├── x 是父的右孩子 → 先左旋父节点，再继续
        └── x 是父的左孩子 → 父黑、爷红，右旋爷爷

  情况B：父节点是爷爷的右孩子 → 叔叔 = 爷爷的左孩子（左右镜像）
    ├── 叔叔为红色 → 变色：父黑、叔黑、爷红，x 跳到爷爷继续
    └── 叔叔为黑色（或null）
        ├── x 是父的左孩子 → 先右旋父节点，再继续
        └── x 是父的右孩子 → 父黑、爷红，左旋爷爷

最后：root.color = BLACK  // 根永远是黑色
```

### 变色与旋转记忆口诀

- **叔叔红** → 只变色，不旋转，把问题向上传递
- **叔叔黑** → 先判断 x 在父的哪边，决定是否先转父，再「父黑爷红 + 反方向旋爷爷」
- **左左** → 右旋爷爷
- **右右** → 左旋爷爷
- **左右** → 先左旋父，变左左，再右旋爷爷
- **右左** → 先右旋父，变右右，再左旋爷爷

### 完整源码

```java
private void fixAfterInsertion(Entry<K,V> x) {
    // 因为红黑树的节点默认就是红色的
    x.color = RED;

    // 按照红黑规则进行调整
    // parentOf(x): 获取 x 的父节点
    // parentOf(parentOf(x)): 获取 x 的爷爷节点
    // leftOf: 获取左子节点
    while (x != null && x != root && x.parent.color == RED) {

        // 判断当前节点的父节点是爷爷节点的左子节点还是右子节点
        // 目的：为了获取当前节点的叔叔节点
        if (parentOf(x) == leftOf(parentOf(parentOf(x)))) {

            // ===== 情况A：父节点是爷爷的左子节点 =====
            // 叔叔 = 爷爷的右子节点
            Entry<K,V> y = rightOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                // 叔叔节点为红色的处理方案
                setColor(parentOf(x), BLACK);               // 父节点设为黑色
                setColor(y, BLACK);                          // 叔叔节点设为黑色
                setColor(parentOf(parentOf(x)), RED);       // 爷爷节点设为红色
                x = parentOf(parentOf(x));                   // 把爷爷节点设置为当前节点
            } else {
                // 叔叔节点为黑色的处理方案
                // 判断当前节点是否为父节点的右子节点
                if (x == rightOf(parentOf(x))) {
                    // 当前节点是父节点的右子节点（左右情况）
                    x = parentOf(x);
                    rotateLeft(x);                          // 左旋
                }
                setColor(parentOf(x), BLACK);               // 父节点设为黑色
                setColor(parentOf(parentOf(x)), RED);        // 爷爷节点设为红色
                rotateRight(parentOf(parentOf(x)));          // 右旋爷爷
            }
        } else {

            // ===== 情况B：父节点是爷爷的右子节点（左右镜像） =====
            // 叔叔 = 爷爷的左子节点
            Entry<K,V> y = leftOf(parentOf(parentOf(x)));
            if (colorOf(y) == RED) {
                setColor(parentOf(x), BLACK);               // 父节点设为黑色
                setColor(y, BLACK);                          // 叔叔节点设为黑色
                setColor(parentOf(parentOf(x)), RED);        // 爷爷节点设为红色
                x = parentOf(parentOf(x));                   // 把爷爷节点设置为当前节点
            } else {
                // 叔叔节点为黑色的处理方案
                if (x == leftOf(parentOf(x))) {
                    // 当前节点是父节点的左子节点（右左情况）
                    x = parentOf(x);
                    rotateRight(x);                         // 右旋
                }
                setColor(parentOf(x), BLACK);               // 父节点设为黑色
                setColor(parentOf(parentOf(x)), RED);        // 爷爷节点设为红色
                rotateLeft(parentOf(parentOf(x)));           // 左旋爷爷
            }
        }
    }

    // 把根节点设置为黑色
    root.color = BLACK;
}
```

---

## 四、课堂思考题总结

### Q1：TreeMap 添加元素，键需要重写 hashCode 和 equals 吗？

**不需要。** TreeMap 靠比较器/Comparable 判断键重复，不依赖 hashCode 和 equals。

### Q2：HashMap 有红黑树，键需要实现 Comparable 或传比较器吗？

**不需要。** HashMap 底层用**哈希值**的大小关系构建红黑树，与比较器无关。

```
// 6.2 HashMap 是哈希表结构的，JDK8 开始由数组、链表、红黑树组成
// 既然有红黑树，HashMap 的键是否需要实现 Comparable 接口或者传递比较器对象呢？
// 不需要。因为 HashMap 底层默认是利用哈希值的大小关系来创建红黑树的
```

### Q3：TreeMap 和 HashMap 谁效率更高？

| 场景 | 结论 |
| --- | --- |
| 极端情况（形成链表） | TreeMap 更高 |
| 一般情况 | **HashMap 更高**（O(1) 平均访问） |

```
// 6.3 如果是最坏情况，添加了 8 个元素，这 8 个元素形成了链表，此时 TreeMap 的效率要更高
// 但是这种情况出现的几率非常少
// 一般而言，还是 HashMap 的效率要更高
```

### Q4：Java 会提供"键重复不覆盖"的 put 方法吗？

会，即 `putIfAbsent`。

**核心思想（代码两面性）**：
- 看到一个逻辑的 A 面，且发现有变量能控制它 → 一定有 B 面
- `boolean` 变量控制 → 一般只有 A/B 两面
- `int` 变量控制 → 至少三面

```
// 6.4 传递一个思想：
//  代码中的逻辑都有两面性，如果我们只知道了其中的 A 面，
//  而且代码中还发现了有变量可以控制两面性的发生，
//  那么该逻辑一定会有 B 面。
//
//  习惯：
//    boolean 类型的变量控制，一般只有 AB 两面，因为 boolean 只有两个值
//    int 类型的变量控制，一般至少有三面，因为 int 可以取多个值
```

### Q5：三种双列集合如何选择？

| 需求 | 选择 |
| --- | --- |
| 默认（效率最高） | **HashMap** |
| 保证存取有序 | **LinkedHashMap** |
| 需要排序 | **TreeMap** |

```
// 6.5 三种双列集合如何选择？
//  HashMap LinkedHashMap TreeMap
//
//  默认：HashMap（效率最高）
//  如果要保证存取有序：LinkedHashMap
//  如果要进行排序：TreeMap
```

---

## 五、一张图速记整体流程

```
put(key, value)
   │
   ▼
 root 为 null？──是──► 作为根节点，返回
   │否
   ▼
 有比较器？──是──► 用 comparator.compare 查找位置
   │否
   ▼
 用 Comparable.compareTo 查找位置
   │
   ▼
 键重复(cmp==0)？──是──► replaceOld? 覆盖 : 不覆盖，返回旧值
   │否
   ▼
 addEntry：创建节点挂到 parent 左/右
   │
   ▼
 fixAfterInsertion：红黑树调整（变色 + 旋转）
   │
   ▼
 root.color = BLACK，结束
```

---

## 六、关键记忆点速览

1. TreeMap = **红黑树**，节点有 `parent` 和 `color`
2. 两种排序：**自然排序**（Comparable）vs **比较器排序**（Comparator）
3. 添加流程：**找位置 → 挂节点 → 红黑树调整**
4. 红黑树调整三招：**变色、左旋、右旋**
5. 根节点永远是**黑色**
6. TreeMap 键**不需要**重写 hashCode/equals
7. 集合选择：默认 HashMap，有序 LinkedHashMap，排序 TreeMap
