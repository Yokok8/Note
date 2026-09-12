# Java 中的编码 / 解码（String 类方法）

## 编码（String → byte[]）—— 写出去

| 方法 | 说明 |
|------|------|
| `public byte[] getBytes()` | 使用**默认方式**进行编码 |
| `public byte[] getBytes(String charsetName)` | 使用**指定方式**进行编码 |

## 解码（byte[] → String）—— 读回来

| 方法 | 说明 |
|------|------|
| `String(byte[] bytes)` | 使用**默认方式**进行解码 |
| `String(byte[] bytes, String charsetName)` | 使用**指定方式**进行解码 |

```java
String s = "黑马";

// ① 编码：String → byte[]
byte[] bytes1 = s.getBytes();           // 默认字符集（JDK 9+ 是 UTF-8）
byte[] bytes2 = s.getBytes("GBK");      // 指定 GBK 编码
System.out.println(bytes2.length);       // "黑马" 在 GBK = 4 字节

byte[] bytes3 = s.getBytes("UTF-8");
System.out.println(bytes3.length);       // "黑马" 在 UTF-8 = 6 字节

// ② 解码：byte[] → String
String s1 = new String(bytes2);         // ❌ 默认 UTF-8 解 GBK 字节 → 乱码
String s2 = new String(bytes2, "GBK");  // ✅ GBK 解 GBK 字节 → "黑马"
```


# 字符流家族图

```
       字符流
   /          \
 Reader      Writer（两个抽象基类）
    |            |
 FileReader   FileWriter（两个具体类）

FileReader：操作本地文件的**字符输入流**
FileWriter：操作本地文件的**字符输出流**
```


# FileReader（字符输入流）

## ① 创建字符输入流对象

| 构造方法 | 说明 |
|---------|------|
| `public FileReader(File file)` | 创建字符输入流关联本地文件 |
| `public FileReader(String pathname)` | 创建字符输入流关联本地文件 |

⚠️ 细节 1：**如果文件不存在，就直接报错**（跟 FileInputStream 一样）

## ② 读取数据

### 两种 read 方法

| 方法 | 说明 |
|------|------|
| `public int read()` | 读取**一个**字符，读到末尾返回 -1 |
| `public int read(char[] buffer)` | 读取**多个**字符，读到末尾返回 -1 |
### read() 空参 vs read(char[]) 有参的区别

#### 核心区别一张表

| 维度 | `read()`（空参） | `read(char[] buffer)`（有参） |
|------|----------------|------------------------------|
| 一次读多少 | **1 个字符** | **一堆字符**（看数组多大） |
| 返回值含义 ⚠️ | 读到的**字符的编码值** | 读到的**字符个数**（不是内容！） |
| 返回类型 | int | int |
| 读到末尾 | -1 | -1 |
| 性能 | 慢 | 快（推荐） |


---
### 🔑 关键细节（黑马标红）

| 细节 | 内容 |
|------|------|
| **细节1** | **按字节进行读取** → 遇到中文，**一次读多个字节**，读取后**解码**，返回一个整数 |
| **细节2** | 读到文件末尾了，**read 方法返回 -1** |

## ③ 释放资源

| 成员方法 | 说明 |
|---------|------|
| `public int close()` | **释放资源 / 关流** |



# FileWriter（字符输出流）
## FileWriter 构造方法（4 个）

| 构造方法 | 说明 |
|---------|------|
| `public FileWriter(File file)` | 创建字符输出流关联本地文件 |
| `public FileWriter(String pathname)` | 创建字符输出流关联本地文件 |
| `public FileWriter(File file, boolean append)` | 创建字符输出流关联本地文件，**续写** |
| `public FileWriter(String pathname, boolean append)` | 创建字符输出流关联本地文件，**续写** |

```java
// ① 覆盖写（默认）
FileWriter fw1 = new FileWriter("D:\\a.txt");

// ② 续写
FileWriter fw2 = new FileWriter("D:\\a.txt", true);

// ③ 用 File 对象
File f = new File("D:\\a.txt");
FileWriter fw3 = new FileWriter(f);
FileWriter fw4 = new FileWriter(f, true); // 续写版
```

## FileWriter 成员方法（5 种）

| 方法 | 说明 |
|------|------|
| `void write(int c)` | 写出**一个字符** |
| `void write(String str)` | 写出**一个字符串** |
| `void write(String str, int off, int len)` | 写出**字符串的一部分** |
| `void write(char[] cbuf)` | 写出**一个字符数组** |
| `void write(char[] cbuf, int off, int len)` | 写出**字符数组的一部分** |

---


# FileWriter 书写细节

## ① 创建字符输出流对象

- **细节1**：参数是字符串表示的路径 **或者** File 对象都是可以的
- **细节2**：如果文件不存在会创建一个新的文件，**但是要保证父级路径是存在的**
- **细节3**：如果文件已经存在，则**会清空文件**，如果不想清空可以打开**续写开关**

## ② 写数据

- **细节**：如果 `write` 方法的参数是整数，但是实际上写到本地文件中的是整数在**字符集**上对应的字符

## ③ 释放资源

- **细节**：每次使用完流之后都要释放资源


# 🔥 FileWriter 比 FileOutputStream 多了什么？

字符流**多了 String 版本**（可以直接写字符串），字节流要先 `getBytes()` 转一下。

| 字节流 FileOutputStream | 字符流 FileWriter                    |
| -------------------- | --------------------------------- |
| `write(byte[])`      | `write(String)` / `write(char[])` |
| 写字符串要先转字节            | **直接写字符串**，更方便 ✅                  |


