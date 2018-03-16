# 多线程



## 基本使用

### `Thread`

方法一，继承`Thread`类

```java
# Worker
public class Worker extends Thread {
    public void run() {
    	// 线程主体声明在重写的run方法内
        for (int i = 0; i < 100; i++) {
            System.out.println(i);
        }
    }
}

# Main
public class Main {
    public static void main(String[] args)  {
    	// 执行时，调用线程类的start方法开启一个线程。此处同时开启三个线程
        new Worker().start();
        new Worker().start();
        new Worker().start();
    }
}
```

> 注意：开启线程时，调用`start`方法，如果调用`run`方法，只是普通方法调用，并不会开启线程。

### `Runnable`

方法二，实现`Runnable`接口。

```java
# Worker
public class Worker implements Runnable {
    public void run() {
    	// 线程主体声明在重写的run方法内
        for (int i = 0; i < 100; i++) {
            System.out.println(i);
        }
    }
}

# Main
public class Main {
    public static void main(String[] args)  {
        Worker worker = new Worker();
        new Thread(worker).start();
        new Thread(worker).start();
        new Thread(worker).start();
    }
}
```

> 此种方法可以用于同时开启同一个对象的多个线程。

### 匿名类

```java
new Thread(() -> {
    for (int i = 0; i < 10; i++) {
        System.out.println(i);
    }
}).start();
```



## 线程池

异步计算案例：

```java
/**
 * Worker实现Callable接口
 */
public class Worker implements Callable<Integer> {

    private int end;

    /**
     * 构造方法，传递变量
     *
     * @param end 计算多少数值之和
     */
    Worker(int end) {
        this.end = end;
    }

    /**
     * 重写call方法，计算求和
     *
     * @return 返回求和结果
     */
    @Override
    public Integer call() {
        int total = 0;
        for (int i = 1; i <= end; i++) {
            total += i;
        }
        return total;
    }

}

/**
 * 主线程
 */
public class Index {

    public static void main(String[] args) {
        // 初始化线程池
        ExecutorService es = Executors.newFixedThreadPool(2);
        Worker task1 = new Worker(100);
        Worker task2 = new Worker(200);

        // 提交任务至线程池，返回Future对象结果
        Future<Integer> s1 = es.submit(task1);
        Future<Integer> s2 = es.submit(task2);
        try {
            // 通过Future对象，获取返回的结果
            System.out.println(s1.get());
            System.out.println(s2.get());
        } catch (InterruptedException | ExecutionException e) {
            e.printStackTrace();
        }
        // 关闭线程池
        es.shutdown();
    }

}
```



## 同步锁

线程同步锁使线程数据安全，但是会造成线程等待，导致速度降低。

所以在多线程中，只把需要更新的数据放在同步块里面，并且赋值给其他变量。

同步块外面不能再使用更新数据的变量，避免其他线程已经更新此变量。

同步块里面，尽量只包裹更新的数据，不要处理其他流程，避免影响性能。

###`synchronized` 关键字

例如：

```java
# Tickets 票务类
public class Tickets implements Runnable {

    private int m;
    private int n = 0;
    private final Object obj = new Object();

    Tickets(int m) {
        this.m = m;
    }

    @Override
    public void run() {
        while (true) {
            int t;
            synchronized (obj) {
                // 此处计算n，并且取出值赋给t，同步块外面不再使用n变量，因为n变量在使用时，可能被其他线程更改，所以使用t变量。
                t = ++n;
            }
            if (t > m) {
                break;
            }
            // 处理第now张票
            System.out.println(t);
        }
    }
}

# Main
public static void main(String[] args) {
    Tickets tk = new Tickets(10);
    new Thread(tk).start();
    new Thread(tk).start();
    new Thread(tk).start();
}
```

同步锁的另外一种写法（推荐）：

```java
public class Tickets implements Runnable {

    private int m;
    private int n = 0;

    Tickets(int m) {
        this.m = m;
    }

    @Override
    public void run() {
        int t;
        while ((t = pay()) <= m) {
            System.out.println(t);
        }
    }

    private synchronized int pay() {
        return ++n;
    }
}
```

> **注意**：`synchronized`声明的同步方法默认同步锁是`this`，静态同步方法的同步锁是`类名.class`。

### `Lock`接口

`Lock`接口比`synchronized`关键字更灵活。在有异常处理时，推荐使用`lock`接口，反正，可以使用`synchronized`关键字。

示例：

```java
public class Tickets implements Runnable {

    private int m;
    private int n = 0;
    private Lock l = new ReentrantLock();

    Tickets(int m) {
        this.m = m;
    }

    @Override
    public void run() {
        while (true) {
            int t = 0;
            l.lock();
            try {
                Thread.sleep(100);
                t = ++n;
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                l.unlock();
            }
            if (t > m) {
                break;
            }
            System.out.println(t);
        }
    }
}
```

