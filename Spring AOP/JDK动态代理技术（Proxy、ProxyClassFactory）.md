# 前言

java界有个熟语：反射是你通向高级的敲门砖，而**动态代理**是你站稳高级的基础。

动态代理技术，相信我们都并不陌生。特别是在Spring框架内，大量的使用到了反射以及动态代理技术。但是如果我们只是停留在平时的运用阶段，此篇文章你其实是可以跳过的，因为反射、代理技术一般都只有在框架设计中才会使用到，业务开发是不用接触的。

一般而言，动态代理分为两种，**一种是JDK反射机制提供的代理**，**另一种是CGLIB代理**。本文主要介绍JDK动态代理的基本原理，让大家更深刻的理解JDK Proxy,知其然知其所以然。（**CGLIB代理，这里也会给个Demo知道就行**）

> JDK动态代理真正的原理及其生成的过程

# 准备工作

先准备一个接口，一个实现类：

```java
public interface Helloworld {
    void sayHello();
}

public class HelloworldImpl implements Helloworld {

    @Override
    public void sayHello() {
        System.out.print("hello world");
    }
}
```

# JDK动态代理Demo

准备一个必须的`InvocationHandler`：

```java
public class MyInvocationHandler implements InvocationHandler {
    private Object target;

    public MyInvocationHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("method :" + method.getName() + " is invoked!");
        return method.invoke(target, args);
    }
}
```

然后写个Main方法实践一下：

## 方式一：

```java
public static void main(String[] args) throws Exception {
    // 传入三大参数，就能够创建出一个代理对象
    Helloworld helloWorld = (Helloworld) Proxy.newProxyInstance(
        ProxyTest.class.getClassLoader(),
        new Class<?>[]{Helloworld.class},
        new MyInvocationHandler(new HelloworldImpl())); //此处目标实现为HelloworldImpl
    helloWorld.sayHello();
}
输出：
method :sayHello is invoked!
hello world
```

此种方式也是最简便的，也是最被我们熟知的创建代理的方式（**所谓三大参数**）

## 方式二：

接下来介绍另外一种方式，平时我们使用较少，了解一下即可

```java
public static void main(String[] args) throws Exception {
    // 三个步骤：
    // 1、生成代理接口的Class（） class com.sun.proxy.$Proxy0
    // 2、拿到构造器：public com.sun.proxy.$Proxy0(java.lang.reflect.InvocationHandler)
    // 3、new一个InvocationHandler实例~~~
    Class<?> proxyClass = Proxy.getProxyClass(ProxyTest.class.getClassLoader(), Helloworld.class);
    Constructor<?> cons = proxyClass.getConstructor(InvocationHandler.class);
    InvocationHandler ih = new MyInvocationHandler(new HelloworldImpl());

    // 通过构造函数 new出一个实例
    Helloworld helloWorld = (Helloworld) cons.newInstance(ih);

    helloWorld.sayHello();

}
输出：
method :sayHello is invoked!
hello world

```

> 方式一的基本原理是方式二

# CGLIB代理Demo

关于CGLIB的动态代理不是本文的重点，因此此处只给一个Demo：
先写一个增强器（CGLIB内部使用的增强器哦~~~~）：

```java
// 注意：这个是org.springframework.cglib.proxy.MethodInterceptor
// 而不是org.aopalliance.intercept包下的
public class MyMethodInterceptor implements MethodInterceptor {

    @Override
    public Object intercept(Object object, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
        Object intercept = methodProxy.invokeSuper(object, args); // 注意这里调用的是methodProxy.invokeSuper
        System.out.println("中介：该房源已发布！");
        return intercept;
    }
}
```

main方法运行：

```java
// 此处需要说明：Enhancer实际属于CGLIB包的，也就是`net.sf.cglib.proxy.Enhancer`
// 但是Spring把这些类都拷贝到自己这来了，因此我用的Spring的Enhancer，包名为;`org.springframework.cglib.proxy.Enhancer`
public static void main(String[] args) throws Exception {
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(HelloServiceImpl.class); // 注意此处的类型必须是实体类
    enhancer.setCallback(new MyMethodInterceptor());

    HelloServiceImpl helloService = (HelloServiceImpl) enhancer.create();
    helloService.hello();

}
输出;
this is my method~~
中介：该房源已发布！

```

# JDK代理生成过程（原理）

我们之所以天天叫JDK动态代理，是因为这个代理class是由JDK在运行时动态帮我们生成。

为了更好的讲述，请在JVM的启动参数上加上如下启动参数：

```properties
-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true
```

这个参数的作用：帮我们把JDK动态生成的proxy class 的字节码保存到硬盘中，然后我们方便查看了

> 可议看到生成的代理类的所在包为：`com.sun.proxy` ，它是默认包名，类名是$Proxy0.class

贴出它生成出来的源码如下：

```java
// 1、所有JDK动态代理  都是Proxy的子类  且自己是final类
// 2、实现了你所需要代理得接口
// 3、代理类整体看起来都是非常简单的  我们发现不管调用哪个方法，最终都是交给了InvocationHandler.invoke()方法  这也就是为什么需要我们提供这个接口的实现类的原因吧
public final class $Proxy0 extends Proxy implements Helloworld {

    private static Method m1;
    private static Method m3;
    private static Method m2;
    private static Method m0;
    	
	// 通过反射给Method赋值   这里我们得出结论
	// Object的三个方法equals/toString/hashCode最终都是会被代理的
	// m3是我们HelloService自己的业务方法
    static {
        try {
            m1 = Class.forName("java.lang.Object").getMethod("equals", Class.forName("java.lang.Object"));
            m3 = Class.forName("com.proxy.Helloworld").getMethod("sayHello");
            m2 = Class.forName("java.lang.Object").getMethod("toString");
            m0 = Class.forName("java.lang.Object").getMethod("hashCode");
        } catch (NoSuchMethodException var2) {
            throw new NoSuchMethodError(var2.getMessage());
        } catch (ClassNotFoundException var3) {
            throw new NoClassDefFoundError(var3.getMessage());
        }
    }

	// 构造函数  
    public $Proxy0(InvocationHandler var1) throws  {
        super(var1);
    }

    public final boolean equals(Object var1) throws  {
        try {
            return (Boolean)super.h.invoke(this, m1, new Object[]{var1});
        } catch (RuntimeException | Error var3) {
            throw var3;
        } catch (Throwable var4) {
            throw new UndeclaredThrowableException(var4);
        }
    }

    public final void sayHello() throws  {
        try {
            super.h.invoke(this, m3, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final String toString() throws  {
        try {
            return (String)super.h.invoke(this, m2, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }

    public final int hashCode() throws  {
        try {
            return (Integer)super.h.invoke(this, m0, (Object[])null);
        } catch (RuntimeException | Error var2) {
            throw var2;
        } catch (Throwable var3) {
            throw new UndeclaredThrowableException(var3);
        }
    }
}

```

从代理出来的类我们可以总结如下（逻辑很简单）：

1. 静态字段：被代理的接口所有方法都有一个对应的静态方法变量；
2. 静态块：主要是通过反射初始化静态方法变量；
3. 具体每个代理方法：逻辑都差不多就是 h.invoke，主要是调用我们定义好的invocatinoHandler逻辑,触发目标对象target上对应的方法;
4. 构造函数：从这里传入我们InvocationHandler逻辑；
   

# Proxy类

一切源于Proxy这个类。我们可以看看

```java
public class Proxy implements java.io.Serializable {
	,,,
    /**
     * a cache of proxy classes
     */
     //KeyFactory和ProxyClassFactory都是内部类
    private static final WeakCache<ClassLoader, Class<?>[], Class<?>>
        proxyClassCache = new WeakCache<>(new KeyFactory(), new ProxyClassFactory());
	
	// 这是我们自己实现的执行器~~~
    protected InvocationHandler h;
	
	// 显然，我们并不能直接new它的对象
	// 它里面所有的方法都是静态方法
    private Proxy() {
    }
    // 这个构造函数由子类调用
    protected Proxy(InvocationHandler h) {
        Objects.requireNonNull(h);
        this.h = h;
    }
    
	// 判断是否为Proxy的子类（也就是否为JDK动态代理生成的类）
	// 可以对比一下ClassUtils.isCglibProxyClass(object.getClass())  判断是否是CGLIB对象(其实就是看名字里是否含有 "$$"这个)
    public static boolean isProxyClass(Class<?> cl) {
        return Proxy.class.isAssignableFrom(cl) && proxyClassCache.containsValue(cl);
    }

	// 这个不用说了，最重要的一个方法：生成代理对象
	public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                          InvocationHandler h) {
		...
	}

	...下面的方法再单独分析
}
```

我们从上面的`方式二`分析：`getProxyClass`方法是入口：需要传入类加载器和interface

```java
@CallerSensitive
public static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces) throws IllegalArgumentException {
    // 克隆一份
    final Class<?>[] intfs = interfaces.clone();
    final SecurityManager sm = System.getSecurityManager();
    if (sm != null) {
        checkProxyAccess(Reflection.getCallerClass(), loader, intfs);
    }
    // 真正做事的是这里
    return getProxyClass0(loader, intfs);
}
```

然后调用`getProxyClass0`方法，方法的JavaDoc其实说得比较清楚了：如果实现当前接口的代理类存在，直接从缓存中返回，如果不存在，则通过`ProxyClassFactory`来创建。

```java
private static Class<?> getProxyClass0(ClassLoader loader, Class<?>... interfaces) {
    // 这里注意：我们发现允许实现接口的上线是65535个（哇，这也太多了吧  哈哈）
    if (interfaces.length > 65535) {
        throw new IllegalArgumentException("interface limit exceeded");
    }

    // 这个JavaDoc说得很清楚，先从缓存拿，否则用`ProxyClassFactory`创建
    // If the proxy class defined by the given loader implementing
    // the given interfaces exists, this will simply return the cached copy;
    // otherwise, it will create the proxy class via the ProxyClassFactory
    return proxyClassCache.get(loader, interfaces);
}
```

# ProxyClassFactory

**ProxyClassFactory是Proxy的一个静态内部类**。它的逻辑包括了下面三步：

1. 包名的创建逻辑
   包名生成逻辑默认是com.sun.proxy，如果被代理类是**non-public proxy interface**（也就说实现的接口若不是public的，包名处理方式不太一样），则用和被代理类接口一样的包名，**类名默认是$Proxy 加上一个自增的整数值**（$Proxy0.class）
2. 调用`ProxyGenerator. generateProxyClass`生成代理类字节码
   `-Dsun.misc.ProxyGenerator.saveGeneratedFiles=true` 这个参数就是在该方法起到作用，如果为true则保存字节码到磁盘。代理类中，所有的代理方法逻辑都一样都是调用invocationHander的invoke方法（上面源码我们也能看出来）
3. 把代理类字节码加载到JVM。
   把字节码通过传入的类加载器加载到JVM中:`defineClass0(loader, proxyName,proxyClassFile, 0, proxyClassFile.length);`


# 总结

通过这一分析，发现JDK的动态代理也不过如此嘛。

由下面几点是需要注意的：

- **toString() hashCode() equal()方法 调用逻辑**：这个三个Object上的方法，如果被调用将和其他接口方法方法处理逻辑一样，都会经过invocationHandler逻辑，从上面的字节码结果就可以明显看出。其他Object上的方法将不会走代理处理逻辑，直接走Proxy继承的Object上方法逻辑。
- interface 含有equals,toString hashCode方法时，和处理普通接口方法一样，都会走invocation handler逻辑，以目标对象重写的逻辑为准去触发方法逻辑
- interface含有重复的方法签名,**以接口传入顺序为准**，**谁在前面就用谁的方法**，**代理类中只会保留一个**，**不会有重复的方法签名**；