# 什么是AOP

AOP（**Aspect-OrientedProgramming**，面向方面编程），可以说是OOP（**Object-Oriented Programing**，面向对象编程）的补充和完善。

AOP技它利用一种称为“横切”的技术，剖解开封装的对象内部，并将那些影响了多个类的公共行为封装到一个可重用模块，并将其名为“Aspect”，即方面。所谓“方面”，简单地说，就是将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。AOP代表的是一个横向的关系，如果说“对象”是一个空心的圆柱体，其中封装的是对象的属性和行为；那么面向方面编程的方法，就仿佛一把利刃，将这些空心圆柱体剖开，以获得其内部的消息。而剖开的切面，也就是所谓的“方面”了。然后它又以巧夺天功的妙手将这些剖开的切面复原，不留痕迹。

实现AOP的技术，主要分为两大类：**一是采用动态代理技术**（**典型代表为Spring AOP**），利用截取消息的方式（**典型代表为AspectJ-AOP**），对该消息进行装饰，以取代原有对象行为的执行；**二是采用静态织入的方式**，引入特定的语法创建“方面”，从而使得编译器可以在编译期间织入有关“方面”的代码。

# 相关概念

**切面（Aspect）**：一个关注点的模块化，这个关注点实现可能另外横切多个对象。**事务管理**是J2EE应用中一个很好的横切关注点例子。切面用spring的 **Advisor**或**拦截器**实现。
**连接点（Joinpoint）**: **程序执行过程中明确的点**，如方法的调用或特定的异常被抛出。
**通知（Advice）**: 在特定的连接点，AOP框架执行的动作。各种类型的通知包括“around”、“before”和“throws”通知。通知类型将在下面讨论。许多AOP框架包括Spring都是以拦截器做通知模型，维护一个“围绕”连接点的拦截器链。**Spring中定义了四个advice: BeforeAdvice, AfterAdvice, ThrowAdvice和DynamicIntroductionAdvice**
**切入点（Pointcut）**: 指定一个通知将被引发的一系列连接点的集合。AOP框架必须允许开发者指定切入点：例如，使用正则表达式。 Spring定义了**Pointcut**接口，用来组合**MethodMatcher**和**ClassFilter**，可以通过名字很清楚的理解， **MethodMatcher是用来检查目标类的方法是否可以被应用此通知**，而**ClassFilter是用来检查Pointcut是否应该应用到目标类上**
**引入（Introduction）**: 添加方法或字段到被通知的类。 Spring允许引入新的接口到任何被通知的对象。例如，你可以使用一个引入使任何对象实现 IsModified接口，来简化缓存。Spring中要使用Introduction, 可有通过**DelegatingIntroductionInterceptor**来实现通知，通过**DefaultIntroductionAdvisor**来配置**Advice**和代理类要实现的接口（**使用较少**）
**目标对象（Target Object）**: 包含连接点的对象。也被称作被通知或被代理对象。POJO
**AOP代理（AOP Proxy）**: AOP框架创建的对象，包含通知。 在Spring中，AOP代理可以是**JDK动态代理**或者**CGLIB代理**。
**织入（Weaving）**: 组装方面来创建一个被通知对象。这可以在**编译时完成**（例如使用AspectJ编译器），也可以在**运行时完成**。Spring和其他纯Java AOP框架一样，在运行时完成织入。

# AOP概念的通俗理解

1. **通知(Advice)**: 通知定义了切面是什么以及何时使用。描述了切面要完成的工作和何时需要执行这个工作。
2. **连接点(Joinpoint)**: 程序能够应用通知的一个“时机”，这些“时机”就是连接点，例如方法被调用时、异常被抛出时等等。
3. **切入点(Pointcut)** ：通知定义了切面要发生的“故事”和时间，那么切入点就定义了“故事”发生的地点，例如某个类或方法的名称，Spring中允许我们方便的用正则表达式来指定
4. **切面(Aspect)** ：通知和切入点共同组成了切面：时间、地点和要发生的“故事”
5. **引入(Introduction)** ：引入允许我们向现有的类添加新的方法和属性(Spring提供了一个方法注入的功能）
6. **目标(Target)** ：即被通知的对象，如果没有AOP,那么它的逻辑将要交叉别的事务逻辑，有了AOP之后它可以只关注自己要做的事（AOP让他做爱做的事）
7. **代理(proxy)** ：应用通知的对象，详细内容参见设计模式里面的代理模式
8. **织入(Weaving)** ：把切面应用到目标对象来创建新的代理对象的过程，织入一般发生在如下几个时机:
   ---- (1)编译时：当一个类文件被编译时进行织入，这需要特殊的编译器才可以做的到，例如AspectJ的织入编译器
   ---- (2)类加载时：使用特殊的ClassLoader在目标类被加载到程序之前增强类的字节代码
   ----(3)运行时：切面在运行的某个时刻被织入,SpringAOP就是以这种方式织入切面的，原理应该是使用了JDK的动态代理技术

# Spring AOP的三种实现方式（基于Spring Boot）

## 一、基于XML配置的Spring AOP

现在都是spring boot的时代了，因此基于xml配置的例子，本文不做介绍了，有需要的可以自己去找其余博文阅读

## 二、基于ProxyFactoryBean，编码的方式来实现

导包

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

这个starter引入了如下依赖

```xml
  <dependencies>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter</artifactId>
      <version>2.1.9.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-aop</artifactId>
      <version>5.1.10.RELEASE</version>
      <scope>compile</scope>
    </dependency>
    <dependency>
      <groupId>org.aspectj</groupId>
      <artifactId>aspectjweaver</artifactId>
      <version>1.9.4</version>
      <scope>compile</scope>
    </dependency>
  </dependencies>
```

有如下包即可正常工作了

> 备注：aopalliance.jar这个jar包已经不用单独引入了，因为Spring-AOP包已经把这个jar包内全部的类都已经放进来了，因此无需重复引入 只需要引入依赖包：`org.aspectj:aspectjweaver`即可

**为何Spring自己实现了AOP，还需要导入org.aspectj:aspectjweaver的包呢？**

官网解释的原因如下：

- 原因一：spring确实有自己的AOP。功能已经基本够用了，除非你的要在接口上动态代理或者方法拦截精确到getter和setter。这些都是写奇葩的需求Spring做不到，但一般不使用。
- 原因二：**1、如果使用xml方式，不需要任何额外的jar包**。**2、如果使用@Aspect的注解方式**。你就可以在类上直接一个@Aspect就搞定，不用费事在xml里配了。但是这需要额外的jar包（ aspectjweaver.jar）。因为spring直接使用AspectJ的注解功能，**注意只是使用了它 的注解功能而已。并不是核心功能 ！！！**

> 注意到文档上还有一句很有意思的话：文档说到 是选择spring AOP还是使用full aspectJ？什么是full aspectJ？如果你使用"full aspectJ"。就是说你可以实现基于接口的动态代理，等等强大的功能。**而不仅仅是aspectj的 注-解-功-能 ！！！**

```java
// A类：
@Service
public class AServiceImpl implements AService {
    @Override
    public void sayHelloA() {
        System.out.println("hello A");
    }
}
// B类：
@Service
public class BServiceImpl implements BService {
    @Override
    public void sayHelloB() {
        System.out.println("hello B");
    }
}
// C类：
@Service
public class CServiceImpl implements CService {
    @Override
    public void sayHelloC() {
        System.out.println("hello C");
    }
}

```

**通过实现接口的方式编写的通知类**

```java
/**
 * 在方法之前、之后 打印输出日志
 *
 * @author fangshixiang@vipkid.com.cn
 * @description
 * @date 2018-10-29 17:42
 */
@Component //通知组件交给容器管理
public class LogAdvice implements MethodBeforeAdvice, AfterReturningAdvice, MethodInterceptor {

    //MethodBeforeAdvice的方法
    @Override
    public void before(Method method, Object[] args, Object target) throws Throwable {
        System.out.println("MethodBeforeAdvice...before...");
    }

    //AfterReturningAdvice的方法
    @Override
    public void afterReturning(Object returnValue, Method method, Object[] args, Object target) throws Throwable {
        System.out.println("AfterReturningAdvice...afterReturning...");
    }


    //MethodInterceptor的方法
    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        System.out.println("MethodInterceptor...invoke...start");
        Object proceed = invocation.proceed();
        System.out.println("MethodInterceptor...invoke...end");
        return proceed;
    }
}
```

> 实现接口 MethodBeforeAdvice该拦截器会在调用方法前执行
> 实现接口 AfterReturningAdvice该拦截器会在调用方法后执行
> 实现接口 MethodInterceptor该拦截器会在调用方法前后都执行，实现环绕效果

然后就是配置了，其中最重要的类为**ProxyFactoryBean、BeanNameAutoProxyCreator、AspectJExpressionPointcutAdvisor**等等代理类，能达到强大的效果。这种一般都是spring时代基于xml的书写方式，因此这里不做详细讲解，SpringBoot时代，建议使用优雅的注解的风格编写，但本文提供一个参考博文：
[Spring AOP之ProxyFactoryBean与BeanNameAutoProxyCreator](https://blog.csdn.net/zhengwei223/article/details/77985407)

## 三、基于注解方式@AspectJ实现AOP

PS：其实springboot此配置是默认开启的，所以根本可以不用管了，在Springboot中使用过注解配置方式的人会问是否需要在程序主类中增加@EnableAspectJAutoProxy来启用，实际并不需要。看下面关于AOP的默认配置属性，其中spring.aop.auto属性默认是开启的，也就是说只要引入了AOP依赖后，其实默认已经增加了@EnableAspectJAutoProxy。
截图看看Boot的一些默认配置：

additional-spring-configuration-metadata.json

```json
{
    "name": "spring.aop.auto",
    "type": "java.lang.Boolean",
    "description": "Add @EnableAspectJAutoProxy.",
    "defaultValue": true
}
```



不过后面会有点奇怪的问题，springboot中，不管这个项是否设置为true或者false，都不会跟以前spring项目中，如果没有设置为true，当代理类没有继承接口，启动项目的时候会报错。而springboot项目中，会自动转换成使用CGLIB进行动态代理，其中原理是怎么实现，就没有去看底层代码了，估计底层进行了改造吧！
切面：

```java
/**
 * @author fangshixiang@vipkid.com.cn
 * @description
 * @date 2018-10-30 11:32
 */
@Component //这个组件一定得加入到容器才行
@Aspect
public class SpringLogAspect {

    //定义一个切入点：指定哪些方法可以被切入（如果是别的类需要使用 请用该方法的全类名）
    @Pointcut("execution(* com.fsx.run.service..*.*(..))")
    public void pointCut() {
    }

    @Before("pointCut()")
    public void doBefore(JoinPoint joinPoint) {
        System.out.println("AOP Before Advice...");
    }

    @After("pointCut()")
    public void doAfter(JoinPoint joinPoint) {
        System.out.println("AOP After Advice...");
    }

    @AfterReturning(pointcut = "pointCut()", returning = "returnVal")
    public void afterReturn(JoinPoint joinPoint, Object returnVal) {
        System.out.println("AOP AfterReturning Advice:" + returnVal);
    }

    @AfterThrowing(pointcut = "pointCut()", throwing = "error")
    public void afterThrowing(JoinPoint joinPoint, Throwable error) {
        System.out.println("AOP AfterThrowing Advice..." + error);
        System.out.println("AfterThrowing...");
    }

    // 环绕通知：此处有一个坑，当AfterReturning和Around共存时，AfterReturning是获取不到返回值的
    //@Around("pointCut()")
    //public void around(ProceedingJoinPoint pjp) {
    //    System.out.println("AOP Aronud before...");
    //    try {
    //        pjp.proceed();
    //    } catch (Throwable e) {
    //        e.printStackTrace();
    //    }
    //    System.out.println("AOP Aronud after...");
    //}

}
```

idea很智能：如果切中了，会有这个小图标的

![](images/SpringAop/idea_aop提示.png)

controller层这么测试：

```java
@Autowired
AService aService;
@Autowired
BService bService;

@Override
public Object testDemo(String str) {
    aService.sayHelloA();
    bService.sayHelloB();
    return str + "succuss~";
}
```

访问，控制台打印结果如下：

```java
AOP Before Advice...
hello A
AOP After Advice...
AOP AfterReturning Advice:AServiceImpl
AOP Before Advice...
hello B
AOP After Advice...
AOP AfterReturning Advice:BServiceImpl

```

> 此处有一个坑，当**AfterReturning**和**Around**共存时，**AfterReturning**是获取不到返回值的。当然，如果你的方法本来就没有返回值，那肯定也是null咯

关于@Pointcut切点表达式的书写，请参见：
[小家Spring】Spring AOP中@Pointcut切入点表达式最全面使用介绍](https://blog.csdn.net/f641385712/article/details/83543270)

# 神一样的AspectJ --> AOP的领跑者

AspectJ 可以成为是AOP的鼻祖，规范的制定者。

因此本文先进行一个简单案例的演示，然后引出AOP中一些晦涩难懂的抽象概念，放心，通过本篇博客，我们将会非常轻松地理解并掌握它们。编写一个HelloWord的类，然后利用**AspectJ**技术切入该类的执行过程。
Hello类：

```java
public class HelloWord {

    public void sayHello(){
        System.out.println("hello world !");
    }
    public static void main(String args[]){
        HelloWord helloWord =new HelloWord();
        helloWord.sayHello();
    }
}
```

编写**AspectJ类**，注意关键字为**aspect**(MyAspectJDemo.aj,**其中aj为AspectJ的后缀**)，含义与class相同，即定义一个AspectJ的类（**注意：后缀名是.aj**）

```java
public aspect MyAspectJDemo {
    /**
     * 定义切点,日志记录切点
     */
    pointcut recordLog():call(* HelloWord.sayHello(..));

    /**
     * 定义切点,权限验证(实际开发中日志和权限一般会放在不同的切面中,这里仅为方便演示)
     */
    pointcut authCheck():call(* HelloWord.sayHello(..));

    /**
     * 定义前置通知!
     */
    before():authCheck(){
        System.out.println("sayHello方法执行前验证权限");
    }

    /**
     * 定义后置通知
     */
    after():recordLog(){
        System.out.println("sayHello方法执行后记录日志");
    }
}
```

这样直接运行HelloWorld的main，是不会有AOP效果的。因为我们还需要配置：改变编译器重新编译后再运行，具体参考：
[AspectJ——简介以及在IntelliJ IDEA下的配置](https://blog.csdn.net/gavin_john/article/details/80156963)
配置好后运行结果如下：

![](images/SpringAop/aspectj运行结果.png)

AspectJ是一个java实现的AOP框架，它能够对java代码进行AOP编译（**一般在编译期进行**），让java代码具有AspectJ的AOP功能（当然需要特殊的编译器），可以这样说AspectJ是目前实现AOP框架中最成熟，功能最丰富的语言，更幸运的是，AspectJ与java程序完全兼容，几乎是无缝关联，因此对于有java编程基础的工程师，上手和使用都非常容易。

# AspectJ的织入方式及其原理概要

对于织入这个概念，可以简单理解为aspect(切面)应用到目标函数(类)的过程。对于这个过程，**一般分为动态织入和静态织入**，动态织入的方式是在运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的，如**Java JDK的动态代理(**Proxy，底层通过**反射实现**)或者**CGLIB的动态代理**(底层通过**继承实现**)，Spring AOP采用的就是基于运行时增强的代理技术，这点另一篇博文已经有分析了，这里主要重点分析一下静态织入。**ApectJ采用的就是静态织入的方式。**ApectJ主要采用的是编译期织入，在这个期间使用AspectJ的acj编译器(类似javac)把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。

![](images/SpringAop/ajc编译过程.png)

关于ajc编译器，是一种能够识别aspect语法的编译器，它是采用java语言编写的，由于javac并不能识别aspect语法，便有了ajc编译器，注意ajc编译器也可编译java文件。为了更直观了解aspect的织入方式，我们打开前面案例中已编译完成的HelloWord.class文件，反编译后的java代码如下：

```java
//编译后织入aspect类的HelloWord字节码反编译类
public class HelloWord {
    public HelloWord() {
    }

    public void sayHello() {
        System.out.println("hello world !");
    }

    public static void main(String[] args) {
        HelloWord helloWord = new HelloWord();
        HelloWord var10000 = helloWord;

   try {
        //MyAspectJDemo 切面类的前置通知织入
        MyAspectJDemo.aspectOf().ajc$before$com_zejian_demo_MyAspectJDemo$1$22c5541();
        //目标类函数的调用
           var10000.sayHello();
        } catch (Throwable var3) {
        MyAspectJDemo.aspectOf().ajc$after$com_zejian_demo_MyAspectJDemo$2$4d789574();
            throw var3;
        }

        //MyAspectJDemo 切面类的后置通知织入 
        MyAspectJDemo.aspectOf().ajc$after$com_zejian_demo_MyAspectJDemo$2$4d789574();
    }
}
```

显然AspectJ的织入原理已很明朗了，当然除了编译期织入，还存在链接期(编译后)织入，即将aspect类和java目标类同时编译成字节码文件后，再进行织入处理，这种方式比较有助于已编译好的第三方jar和Class文件进行织入操作，由于这不是本篇的重点，暂且不过多分析，掌握以上AspectJ知识点就足以协助理解Spring AOP了

# Spring AOP的优点

Spring AOP 与ApectJ 的目的一致，都是为了**统一处理横切业务**，但与AspectJ不同的是，**Spring AOP 并不尝试提供完整的AOP功能(即使它完全可以实现)**，Spring AOP 更注重的是与Spring IOC容器的结合，并结合该优势来解决横切业务的问题，因此在AOP的功能完善方面，相对来说AspectJ具有更大的优势

同时,Spring注意到AspectJ在AOP的实现方式上依赖于特殊编译器(ajc编译器)，因此Spring很机智回避了这点，转向采用**动态代理技术的实现原理**来构建Spring AOP的内部机制（动态织入），这是与AspectJ（静态织入）最根本的区别。

在AspectJ 1.5后，引入@Aspect形式的注解风格的开发，Spring也非常快地跟进了这种方式，因此Spring 2.0后便使用了与AspectJ一样的注解。请注意，**Spring 只是使用了与 AspectJ 5 一样的注解**，但仍然没有使用 AspectJ 的编译器，底层依是动态代理技术的实现，因此并不依赖于 AspectJ 的编译器

# 再说区别和联系

## 区别

## AspectJ

AspectJ是一个面向切面的框架，**它扩展了Java语言**。AspectJ定义了AOP语法，所以它有一个专门的编译器用来生成遵守Java字节编码规范的Class文件（**只是我们Spring一般只用到它的注解，其余都是Spring自己实现的**）。

## Spring AOP

Spring提供了四种类型(说三种也对)的Aop支持：

1. 基于经典的SpringAOP

   1. 使用ProxyFactoryBean创建Bean。显然已经过时了，因为每个Bean你都使用它包装一下，麻烦
   2. 引入自动代理创建器。如：**AbstractAutoProxyCreator**的所有子类，实现有多种。(**EnableAspectJAutoProxy**注解驱动原理就是它)
      1. 前置通知：org.springframework.aop.MethodBeforeAdvice
      2. 后置通知：org.springframework.aop.AfterReturningAdvice
      3. 异常通知：org.springframework.aop.ThrowsAdvice
      4. 环绕通知：org.aopalliance.intercept.MethodInterceptor （注意这个是aopalliance包下的接口，Spring提供实现类非常之多，比如：**MethodBeforeAdviceInterceptor**(包装了MethodBeforeAdvice)，CacheInterceptor等等非常多）

2. 纯POJO切面
   这个方式也就比较古老了，纯基于xml，简单例子如下：

   ```xml
   <-- Spring Aop的命名空间可以将纯POJO转换为切面，实际上这些POJO只是提供了满足切点的条件时所需要调用的方法，但是，这种技术需要XML进行配置，不能支持注解  -->
   <bean id="pojoAdvice" class="com.njust.learning.spring.pojoaop.PojoAdvice"></bean>
   <aop:config>
     <aop:pointcut id="p" expression="execution (* *.add(..))"/>
     <aop:aspect ref="pojoAdvice">
       <aop:before method="before" pointcut-ref="p"/>
       <!--通过设置returning来将返回值传递给通知-->
       <aop:after-returning method="after" pointcut-ref="p" returning="returnval"/>
       <aop:around method="around" pointcut-ref="p"/>
       <!--通过设置returning来将异常对象传递给通知-->
       <aop:after-throwing method="afterThrowing" pointcut-ref="p" throwing="e"/>
     </aop:aspect>
   </aop:config>
   ```

   3. @ASpectJ注解驱动的切面

   1. 需要注意的是，Spring只使用到了它的注解功能，并不使用它的核心功能。核心解析功能由Spring自己实现的

```java
@Aspect
public class HelloAspect {
    @Pointcut("execution(* com.fsx.service.*.*(..)) ")
    public void point() {
    }
    @Before("point()")
    public void before() {
        System.out.println("this is from HelloAspect#before...");
    }
    @After("point()")
    public void after() {
        System.out.println("this is from HelloAspect#after...");
    }
    @AfterReturning("point()")
    public void afterReturning() {
        System.out.println("this is from HelloAspect#afterReturning...");
    }
    @AfterThrowing("point()")
    public void afterThrowing() {
        System.out.println("this is from HelloAspect#afterThrowing...");
    }
    // 此处需要注意：若写了@Around方法，那么最后只会执行@Around和@AfterReturning 其它的都不会执行
    //@Around("point()")
    //public void around() {
    //    System.out.println("this is from HelloAspect#around...");
    //}
}
```

4. 注入式AspectJ切面（其实与Spring并无多大的关系，这个就是使用AspectJ这个框架实现Aop编程） 需用用到`AspectJ`自己的编译器，因此其实也可以说这个不属于Spring的

1. 这种方式上面已经做了案例了，此处忽略（使用较少，但Spring也给与了支持）

## 联系

我们借助于Spring Aop的命名空间可以将纯POJO转换为切面，实际上这些POJO只是提供了满足切点的条件时所需要调用的方法，但是，这种技术需要XML进行配置，不能支持注解
**所以spring借鉴了AspectJ的切面，以提供注解驱动的AOP，本质上它依然是Spring基于代理的AOP，只是编程模型与AspectJ完全一致，这种风格的好处就是不需要使用XML进行配置**（XML方式已经淘汰嘛，哈哈）

# 最后

博主希望通过本文，让读者能够了解到Spring AOP和AspectJ的区别与联系。