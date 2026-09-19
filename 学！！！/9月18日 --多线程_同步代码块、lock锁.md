# 同步代码块 `synchronized`

| **特点 1** | 锁**默认打开**，有一个线程进去了，**锁自动关闭**   |
| -------- | ------------------------------ |
| **特点 2** | 里面的代码**全部执行完毕**，线程出来，**锁自动打开** |

**synchronized 括号里放锁，进门自动关、出门自动开；**  
**锁对象必须唯一，this 要看清 —— 继承 Thread 时得用 class。**

## 格式

```java
synchronized (锁) {
    操作共享数据的代码
}

public class MyThread extends Thread {
    // ⚠️ 共享数据必须是 static（继承 Thread 时每个线程是不同对象）
    static int ticket = 0;
    @Override
    public void run() {
        while (true) {
            // ✅ 锁对象：MyThread.class（字节码对象，全局唯一）
            synchronized (MyThread.class) {
                if (ticket < 100) {
                    try {
                        Thread.sleep(10);   // 放大问题：让线程在这卡一下
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    ticket++;
                    System.out.println(getName() + "正在卖第" + ticket + "张票");
                } else {
                    break;      // 票卖完了，跳出循环（锁会自动释放）
                }
            }
        }
    }
}


public class ThreadDemo {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
        MyThread t3 = new MyThread();

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}

```


# 同步方法 `synchronized`
## 卖票案例完整实现（同步方法版）

### 需求

> 某电影院目前正在上映国产大片，共有 **100 张票**，而它有 **3 个窗口**卖票。
> 要求：**用同步方法完成**，不能出现超卖、重卖。

---
### 完整代码

### ① 任务类 MyRunnable

```java
public class MyRunnable implements Runnable {
    // ⚠️ 共享数据：不用 static（三个线程共用一个 mr 对象）
    int ticket = 0;

    @Override
    public void run() {
        while (true) {
            // 2. 调用同步方法，返回 true 表示票卖完了
            if (method()) break;
        }
    }

    // 同步方法 → 锁对象是 this（也就是唯一的那个 mr 对象）
    private synchronized boolean method() {
        // 3. 判断共享数据是否到了末尾
        if (ticket == 100) {
            return true;          // 卖完了，通知外面跳出循环
        } else {
            // 4. 没到末尾，正常卖票
            try {
                Thread.sleep(10); // 放大线程安全问题
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            ticket++;
            System.out.println(Thread.currentThread().getName()
                    + "正在卖第" + ticket + "张票");
        }
        return false;             // 还没卖完，继续下一轮
    }
}

public class ThreadDemo {
    public static void main(String[] args) {
        // 只创建一个任务对象（关键！）
        MyRunnable mr = new MyRunnable();

        // 三个线程共用一个任务对象
        Thread t1 = new Thread(mr);
        Thread t2 = new Thread(mr);
        Thread t3 = new Thread(mr);

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
```


# Lock 锁（JDK5 之后的新选择）

## 是什么

| 名称 | 说明 |
|---|---|
| `Lock` | **接口**，不能直接 new |
| `ReentrantLock` | `Lock` 的**实现类**（名字含义：**可重入锁**），平时用这个 |

```java
// 创建锁对象（通常写成 static，让所有线程共用同一把）
static Lock lock = new ReentrantLock();
```

|方法|作用|
|---|---|
|`void lock()`|**上锁**|
|`void unlock()`|**解锁**|

> ⚠️ 和 `synchronized` 最大的区别：**这两个方法都要你自己写**。  
> `synchronized` 是自动挡，`Lock` 是手动挡 —— 灵活，但也容易出事故。


```java
public class MyThread extends Thread {
    // 共享数据
    static int ticket = 0;

    // ⚠️ 锁对象要 static，保证三个线程是同一把锁
    static Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {
            lock.lock();      // ① 手动上锁（⚠️ 写在 try 外面！）
            try {
                // ② 临界区：操作共享数据的代码
                if (ticket == 100) {
                    break;     // break 也会走 finally
                } else {
                    Thread.sleep(10);
                    ticket++;
                    System.out.println(getName() + "正在卖第" + ticket + "张票");
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            } finally {
                lock.unlock();   // ③ 手动解锁（⚠️ 必须放 finally）
            }
        }
    }
}


public class ThreadDemo {
    public static void main(String[] args) {
        MyThread t1 = new MyThread();
        MyThread t2 = new MyThread();
        MyThread t3 = new MyThread();

        t1.setName("窗口1");
        t2.setName("窗口2");
        t3.setName("窗口3");

        t1.start();
        t2.start();
        t3.start();
    }
}
	 
```