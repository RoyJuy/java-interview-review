### java 关键字 

### - synchronized



由于同一进程多个线程共享一片内存空间，在带来方便的同时，也带来了访问冲突的问题。java提供了专门的机制来解决这一问题，有效避免了同一数据对象被多个线程同时访问。

- synchronized关键字可以修饰函数，也可作为函数内的语句，就是平时所说的同步方法和同步语句块。再细分，synchronized可作用于instance变量，object reference（对象引用）、static 函数和class literals（类名称字面常量）身上。
- 无论synchronized关键字加在方法上还是对象上，它取得的锁都是对象，而不是把一段代码或者函数当做锁，而且同步方法很可能还可能会被其他线程的对象访问

```java
public class SynchronizedTest implements Runnable {

    private static int count = 0;

    public void incrementCount() {
        for (int i = 0; i < 100000; i++) {
            count++;
        }
    }

    @Override
    public void run() {
        incrementCount();
    }

    public static void main(String[] args) throws InterruptedException {
        SynchronizedTest runnableTest = new SynchronizedTest();
        Thread t1 = new Thread(runnableTest);
        Thread t2 = new Thread(runnableTest);

        // t1 执行incrementCount
        t1.start();
        // t2 执行incrementCount
        t2.start();
        t1.join();
        t2.join();
        // 主线程waiting ， 等待t1， t2 
        System.out.println(count);

    }
}
```

输出结果 ：<= 200000

#### 确保输出结果 == 200000,  **synchronized** 控制

1. 修饰代码块：

   ```java
   
       // 新增objMonitor
       private Object objMonitor = new Object();
   
       public void incrementCount() {
           // 代码块修饰， 确保这一块代码每一次只能有一个线程进行调用
           synchronized (objMonitor) {
               for (int i = 0; i < 100000; i++) {
                   count++;
               }
           }
       }
   
   ```

   **<u>*count ++*</u>**  操作的代码块使用synchronized修饰确保每次只有一个线程进行调用

2. 修饰普通方法：

   ```java
   
       // 普通方法修饰
       public synchronized void incrementCount() {
   
           for (int i = 0; i < 100000; i++) {
               count++;
           }
       }
   
   ```

   <u>***incrementCount***</u> 普通方法上修饰

3. 修饰静态方法：

   ```java
     // 静态方法修饰
       public synchronized static void incrementCount() {
   
           for (int i = 0; i < 100000; i++) {
               count++;
           }
       }
   ```

   <u>***incrementCount***</u>  静态方法修饰

#### 原子性，可见性，有序性

通过 <u>***synchronized***</u> 修饰的方法或者方法块，能够同时保证这段代码的原子性、可见性和有序性。线程安全

原子性：同一时间单线程操作

可见性：对一个变量进行unlock之前，必须先把此变量同步回主内存中执行（store/write）

有序性：在一个线程内，按照控制流顺序，书写在前面的操作先行发生于书写在后面的操作（<u>***Program Order Rule***</u>）

#### 锁对象

在修饰方法块的时候使用了一个Object对象，进行获取锁对象

```java
private Object objMonitor = new Object(); 

synchronized (objMonitor){ 
  //执行方法
}
```

在修饰普通方法和静态方法时，只是在方法前修饰了，没有明确的对象。其实三者都有各自的锁对象，只有获取了锁对象，线程才能进入执行里面的代码

- 修饰代码块：锁定锁是synchronized括号里面的对象
- 修饰普通方法：锁定调用当前方法的this对象
- 修饰静态方法：锁定当前类的class对象

多个线程之间，如果要通过synchronized保证线程安全，获取的要是同一把锁。如果多线程获取多把锁就会出现线程安全问题

#### 可重入

在一个线程获取到这个锁了，在未释放这把锁之前，还能进入获取锁

```java
 // 新增objMonitor
    private Object objMonitor = new Object();
    
    public void incrementCount() {
        // 第一次获取锁
        synchronized (objMonitor) {
            for (int i = 0; i < 100000; i++) {
                count++;
            }
        }
        // 此时线程未terminate，锁未释放
        getObjMonitorAgain();
    }

    private void getObjMonitorAgain() {
        // 再次获取锁，和第一次是同一把
        synchronized (objMonitor) {

        }
    }
```

上面代码出两次调用<u>***synchronized(objMonitor)***</u>获取的是同一把锁