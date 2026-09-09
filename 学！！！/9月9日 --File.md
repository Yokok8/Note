# 路径

## ① 相对路径

相对当前工作目录的路径

| 示例 | 含义 |
|------|------|
| `"a.txt"` | 当前目录下找 a.txt |
| `"abc\\a.txt"` | 当前目录下的 abc 文件夹里找 a.txt |

## ② 绝对路径

从盘符（根目录）开始的完整路径

| 示例 | 含义 |
|------|------|
| `"C:\\a.txt"` | C 盘根目录下的 a.txt |
| `"C:\\abc\\a.txt"` | C 盘 abc 文件夹下的 a.txt |


# File 对象

- File 对象就表示一个路径，可以是**文件的路径**、也可以是**文件夹的路径**
- 这个路径可以是**存在的**，也**允许是不存在的**

---
## 🔥 这两句话的重点（黑马反复考）

| 重点 | 含义 |
|------|------|
|  **“文件 + 文件夹”** &#8203; | File 一个类**两种身份**，用 `isFile()` 和 `isDirectory()` 区分 |
|  **“允许不存在”** &#8203; | `new File("xxx.txt")` **不会真创建**文件，只是造个"代表对象" |

# File类的构造方法

| 方法名                                        | 说明                                         |
| ------------------------------------------ | ------------------------------------------ |
| `public File(String pathname)`             | 根据**文件路径**创建文件对象（**单参**）                   |
| `public File(String parent, String child)` | 根据**父路径字符串 + 子路径字符串**创建文件对象（**双参 1**）      |
| `public File(File parent, String child)`   | 根据**父路径 File 对象 + 子路径字符串**创建文件对象（**双参 2**） |

---
### 💡 三种写法对照

```java
// ① 单参：最常用，直接传完整路径
File f1 = new File("D:\\aaa\\a.txt");

// ② 双参 1：父路径是字符串（适合"父目录固定，子路径会变"的场景）
File f2 = new File("D:\\aaa", "a.txt");

// ③ 双参 2：父路径是 File 对象（适合"父目录自己也是动态生成的"场景）
File parentDir = new File("D:\\aaa");
File f3 = new File(parentDir, "a.txt");
```
> 三种写法**效果完全一样**，都是创建 `D:\aaa\a.txt` 这个路径的 File 对象。


# File 的常见成员方法（判断、获取）

## ① 判断类（3 个）

| 方法                             | 说明                       |
| ------------------------------ | ------------------------ |
| `public boolean isDirectory()` | 判断此路径表示的 File **是否为文件夹** |
| `public boolean isFile()`      | 判断此路径表示的 File **是否为文件**  |
| `public boolean exists()`      | 判断此路径表示的 File **是否存在**   |
判断存在，是否是文件、文件夹
## ② 获取类（5 个）

| 方法                                | 说明                          |
| --------------------------------- | --------------------------- |
| `public long length()`            | 返回文件的**大小（字节）**&#8203;      |
| `public String getAbsolutePath()` | 返回文件的**绝对路径**               |
| `public String getPath()`         | 返回**定义文件时使用的路径**            |
| `public String getName()`         | 返回文件的**名称，带后缀**             |
| `public long lastModified()`      | 返回文件的**最后修改时间（毫秒值）**&#8203; |
获取大小、绝对路径、定义文件时使用的路径、文件的名称、最后修改时间
获取大小只能获取文件的大小，想获取文件夹的大小就获取里面文件的大小总和


# File 的常见成员方法（创建、删除）

| 方法                               | 说明                          |
| -------------------------------- | --------------------------- |
| `public boolean createNewFile()` | 创建新的空文件（创建的一定是文件，会创建没后缀的文件） |
| `public boolean mkdir()`         | 创建**单级**文件夹                 |
| `public boolean mkdirs()`        | 创建**多级**文件夹                 |
| `public boolean delete()`        | 删除**文件、空文件夹**               |

---

## 🔥 重点提醒（黑马单独标红的）

> **`delete()` 默认只能删除文件 / 空文件夹，且直接删除不走回收站！**
> 意味着**删了就没了**，不可恢复（Windows 回收站都进不去）。
> 有文件的文件夹不能删

---

### 💡 实战代码（黑马这页没给，我帮你补）

```java
// ① createNewFile：创建文件
File f1 = new File("D:\\aaa.txt");
System.out.println(f1.createNewFile()); // true（首次创建成功）
System.out.println(f1.createNewFile()); // false（文件已存在）

// ② mkdir：创建单级文件夹
File f2 = new File("D:\\aaa");
System.out.println(f2.mkdir()); // true（前提：aaa不存在）

// ③ mkdirs：创建多级文件夹（连父目录一起建）
File f3 = new File("D:\\aaa\\bbb\\ccc");
System.out.println(f3.mkdirs()); // true（aaa/ bbb/ ccc 全建好）

// ④ delete：删除
System.out.println(f1.delete()); // true（删除文件）
System.out.println(f3.delete()); // ❌ false（ccc 是非空文件夹，删不动）
System.out.println(f2.delete()); // true（删空文件夹）
```

# File 的常见成员方法（获取并遍历）

| 方法 | 说明 |
|------|------|
| `public File[] listFiles()` | 获取当前该路径下所有内容 |

---
## 💡 实战代码（黑马只给方法名，我补完整示例）

```java
File dir = new File("D:\\aaa");
File[] files = dir.listFiles(); // 拿到 aaa 下所有内容

for (File f : files) {
    System.out.println(f.getName() + " | " 
 + (f.isFile() ? "文件" : "文件夹"));
}

//假设 D:\\aaa 下有：
//   hello.txt
//   data.json
//   images 文件夹
// 输出：
//   hello.txt | 文件
//   data.json | 文件
//   images | 文件夹
```
# listFiles() 返回值的 6 种情况

| # | 调用场景 | 返回值 |
|---|---------|--------|
| 1 | 路径**不存在** | ❌ 返回 **null** |
| 2 | 路径是**文件**（不是文件夹）| ❌ 返回 **null** |
| 3 | 路径是**空文件夹** | ✅ 返回 **长度为0 的数组**（不是 null） |
| 4 | 路径是**有内容的文件夹** | ✅ 返回 **File[]**，含里面所有文件和文件夹 |
| 5 | 路径是**有隐藏文件**的文件夹 | ✅ 返回 **File[]**，**包含隐藏文件** |
| 6 | 路径是**需要权限**才能访问的文件夹 | ❌ 返回 **null** |

---
### 🔍 一眼对照（黑马最爱考）
返回值是 null 的3 种： 不存在、是文件、没权限  
返回值是数组的 3 种： 空文件夹、有内容文件夹、有隐藏文件

| 方法                                               | 说明                               |
| ------------------------------------------------ | -------------------------------- |
| `public static File[] listRoots()`               | 列出可用的**文件系统根**（**静态方法**）         |
| `public String[] list()`                         | 获取当前该路径下所有内容（**名字** String[]）    |
| `public String[] list(FilenameFilter filter)`    | 利用文件名过滤器获取                       |
| `public File[] listFiles()`                      | 获取当前该路径下所有内容（**File 对象** File[]） |
| `public File[] listFiles(FileFilter filter)`     | 利用文件过滤器获取                        |
| `public File[] listFiles(FilenameFilter filter)` | 利用文件名过滤器获取                       |
