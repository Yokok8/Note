# FileInputStream（文件字节输入流）

**作用**：操作本地文件的字节输入流，可以把本地文件中的数据读取到程序中来。

---

## 📌 读数据的 3 步流程

| 步骤 | 说明 |
|------|------|
| ① **创建字节输入流对象** | `new FileInputStream(文件路径)` |
| ② **读数据** | `fis.read(...)` |
| ③ **释放资源** | `fis.close()`（关流）|

```java
import java.io.FileInputStream;
import java.io.IOException;

public class Test {
    public static void main(String[] args) throws IOException {
        // ① 创建字节输入流对象
        FileInputStream fis = new FileInputStream("D:\\a.txt");

        // ② 读数据（一次读一个字节）
        int b = fis.read();
        System.out.println((char) b); // 转成字符打印

        // ③ 释放资源
        fis.close();
    }
}
```


# 字节输入流的细节

## 1. 创建字节输入流对象

| 细节 | 说明 |
|------|------|
| **细节1** | 如果文件**不存在，就直接报错**（FileNotFoundException）|

## 2. 读数据

| 细节 | 说明 |
|------|------|
| **细节1** | 一次读一个字节，**读出来的是数据在 ASCII 上对应的数字** |
| **细节2** ⚠️ | 读到文件末尾了，**read 方法返回 -1**（黑马循环的核心）|

## 3. 释放资源

| 细节 | 说明 |
|------|------|
| 必关 | 每次用完都要 close |

# 字节输入流循环读取

```java
// 1. 创建对象
FileInputStream fis = new FileInputStream("myio\\a.txt");

// 2. 循环读取
int b;
while ((b = fis.read()) != -1) {
    System.out.println((char) b);
}

// 3. 释放资源
fis.close();
```
