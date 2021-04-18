#### 单例模式

> 单例模式：全局实例只有一个。
> 单例模式，如线程池，数据库线程池，日志文件，缓存等，单例模式可以避免生产过多的对象，以及不同实例下导致程序异常或者数据不一致的情况
> 在java中需要保证线程安全

如：
##### 饿汉模式
> 先创建好全局静态对象，这是最经典的单例，缺点就是创建的对象不一定使用，先创建好会导致资源浪费
```java
public class StaticSingletone {

    private static final StaticSingletone instance = new StaticSingletone();

    private StaticSingletone() {

    }

    public static StaticSingletone getInstance() {
        return instance;
    }
}
```

##### 懒汉模式
> 只有使用的时候才实例化对象，但为保证线程安全，直接使用同步锁锁住创建方法，缺点就是效率慢

```java

public class SyncSingletone {

    private static SyncSingletone instance;

    private SyncSingletone() {

    }

    public static synchronized SyncSingletone getInstance() {
        if (instance == null) {
            instance = new SyncSingletone();
        }
        return instance;
    }
}
```
##### 双重检查加锁
> 当需要创建的时候才加锁，为保证线程安全，还需要将对象实例设置为volatile，即线程共享，保证获取到的实例是共享的数据，
> 再判断一次是多重保障，相对来说，这个资源消耗比较合理，但实现比较复杂，可根据自身情况选择不同的单例实现

```java
public class DoubleSyncSingletone {

    private volatile static DoubleSyncSingletone instance;

    private DoubleSyncSingletone() {

    }

    public static DoubleSyncSingletone getInstance() {
        if (instance == null) {
            synchronized (DoubleSyncSingletone.class) {
                if (instance == null) {
                    instance = new DoubleSyncSingletone();
                }
            }
        }
        return instance;
    }
}
```
 