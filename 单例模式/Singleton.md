# 应用场景

只需要一个实例，比如各种Manager、Factory

# 写法

## 饿汉式

类加载到内存后，就实例化一个单例，JVM保证线程安全

简单实用，推荐实用

唯一缺点：不管用到与否，类装载时就完成实例化

构造方法私有

```java
public class Mgr01 {
    private static final Mgr01 INSTANCE = new Mgr01();
    private Mgr01(){};
    public static Mgr01 getInstance() {
        return INSTANCE;
    }
}
public class Mgr02 {
    private static final Mgr02 INSTANCE;
    static {
        INSTANCE = new Mgr02();
    }
    private Mgr02(){};
    public static Mgr02 getInstance() {
        return INSTANCE;
    }
}
```
## 懒汉式 

lazy loading 懒加载，虽然达到了按需初始化的目的，但却带来线程不安全的问题
```java
public class Mgr03 {
    private static Mgr03 INSTANCE;
    private Mgr03(){};
    public static Mgr03 getInstance() {
    	if (INSTANCE == null) {
            try {
                Thread.sleep(1);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
    		INSTANCE = new Mgr03();
    	}
        return INSTANCE;
    }
    public static void main(Stirng[] args) {
    	for (int i = 0; i < 100; i ++) {
    		new Thread(() -> System.out.println(Mgr03.getInstance().hashCode())).start();
    	}
    }
}
```

通过synchronized解决线程安全问题，但也带来效率下降

```java
public class Mgr04 {
    private static Mgr04 INSTANCE;
    private Mgr04(){};
    public static synchronized Mgr04 getInstance() {
    	if (INSTANCE == null) {
    		INSTANCE = new Mgr04();
    	}
        return INSTANCE;
    }
}
```

妄图通过减小同步代码块的方式提高效率，然后*不可行*

```java
public class Mgr05 {
    private static Mgr05 INSTANCE;
    private Mgr06(){};
    public static Mgr05 getInstance() {
    	if (INSTANCE == null) {
            synchronized (Mgr05.class){
            	INSTANCE = new Mgr05();
            }
    	}
        return INSTANCE;
    }
}
```

双重检查

```java
public class Mgr06 {
    private static volatile Mgr06 INSTANCE; //JIT，防止指令重排
    private Mgr06(){};
    public static Mgr06 getInstance() {
    	if (INSTANCE == null) {
            synchronized (Mgr06.class){
                if (INSTANCE == null) {
                    INSTANCE = new Mgr06();
                }
            }
    	}
        return INSTANCE;
    }
}
```

## 静态内部类方式

JVM保证单例，每个类仅加载一次

加载外部类时不会加载内部类，这样可以实现懒加载

```java
public class Mgr07 {
    private Mgr07(){};
    private static class Mgr07Holder {
        private final static Mgr07 INSTANCE = new Mgr07();
    }
    public static Mgr07 getInstance() {
        return Mgr07Holder.INSTANCE;
    }
}
```

## 枚举方式

不仅可以解决线程同步，还可以防止反序列化，因为枚举没有构造方法，用Class文件无法构造对象。

```java
public enum Mgr08 {
    INSTANCE;
    
    public static void main(Stirng[] args) {
    	for (int i = 0; i < 100; i ++) {
    		new Thread(() -> System.out.println(Mgr08.INSTANCE.hashCode())).start();
    	}
    }
}
```

