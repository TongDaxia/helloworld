## 并发Bug产生的原因



为了平衡CPU，内存和IO的性能差异，计算机进行了一些优化：

1. CPU 增加了缓存，以均衡与内存的速度差异；
2. 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；
3. 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。

### Bug源头1：缓存导致的可见性（CPU缓存）

一个线程对共享变量的修改，另外一个线程能够立刻看到，我们称为**可见性**。

CPU缓存导致无法实现可见性。

### Bug源头2：线程切换带来的原子性问题

程序中的一个操作对应CPU多条指令，CPU执行多线程任务时的线程切换导致无法实现原子性。

### Bug源头之3：编译优化带来的有序性问题

编译优化时会打乱顺序。

比如双重检查 创建单例对象还会有问题：

```
public class Singleton {
  static Singleton instance;
  static Singleton getInstance(){
    if (instance == null) {
      synchronized(Singleton.class) {
        if (instance == null)
          instance = new Singleton();
        }
    }
    return instance;
  }
```

因为优化后的 new 对象 执行逻辑被打乱了：

1. 分配一块内存 M；
2. 将 M 的地址赋值给 instance 变量；
3. 最后在内存 M 上初始化 Singleton 对象。

如果CPU在2和3之间有切换，就会导致后面的线程出现空指针。

现在比较通用的是**静态内部类**实现单例：

```
public class MySingleton {
    //内部类
    private static class MySingletonHandler{
        private static MySingleton instance = new MySingleton();
    } 
    private MySingleton(){}
    public static MySingleton getInstance() { 
        return MySingletonHandler.instance;
    }
}
```



## JAVA内存模型

解决以上问题最直接的方法是**禁用缓存和编译优化** ；

最合理的方法是 **按需禁用缓存以及编译优化**

具体的实现是 **volatile**、**synchronized** 和 **final** 三个关键字，以及六项 **Happens-Before 规则**内容。

**volatile** :不能使用 CPU 缓存

**Happens-Before  前面一个操作的结果对后续操作是可见的**











































