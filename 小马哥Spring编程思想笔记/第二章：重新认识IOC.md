# 控制翻转

[维基百科](https://wiki.hk.wjbk.site/baike-%E6%8E%A7%E5%88%B6%E5%8F%8D%E8%BD%AC)

**控制反转**（Inversion of Control，缩写为**IoC**），是[面向对象编程](https://wiki.hk.wjbk.site/baike-面向对象编程)中的一种设计原则，可以用来减低计算机代码之间的[耦合度](https://wiki.hk.wjbk.site/baike-耦合度_(計算機科學))。其中最常见的方式叫做**依赖注入**（Dependency Injection，简称**DI**），还有一种方式叫“依赖查找”（Dependency Lookup）。通过控制反转，对象在被创建的时候，由一个调控系统内所有对象的外界实体，将其所依赖的对象的引用传递(注入)给它。

## 起源

早在2004年，[Martin Fowler](https://wiki.hk.wjbk.site/baike-Martin_Fowler)就提出了“哪些方面的控制被反转了？”这个问题。他总结出是依赖对象的获得被反转了，因为大多数应用程序都是由两个或是更多的类通过彼此的合作来实现业务逻辑，这使得每个对象都需要获取与其合作的对象（也就是它所依赖的对象）的引用。如果这个获取过程要靠自身实现，那么这将导致代码高度[耦合](https://wiki.hk.wjbk.site/baike-耦合力_(計算機科學))并且难以维护和调试。

## 技术描述

Class A中用到了Class B的对象b，一般情况下，需要在A的代码中显式的new一个B的对象。

采用依赖注入技术之后，A的代码只需要定义一个私有的B对象，不需要直接new来获得这个对象，而是通过相关的容器控制程序来将B对象在外部new出来并注入到A类里的引用中。而具体获取的方法、对象被获取时的状态由配置文件（如XML）来指定。

## 实现方法

实现控制反转主要有两种方式：**依赖注入和依赖查找**。两者的区别在于，**前者是被动的接收对象**，在类A的实例创建过程中即创建了依赖的B对象，通过类型或名称来判断将不同的对象注入到不同的属性中，而**后者是主动索取相应类型的对象**，获得依赖对象的时间也可以在代码中自由控制。

## 实现技术

- Using a [service locator pattern](https://en.wikipedia.org/wiki/Service_locator_pattern)(使用服务定位器模式)
- Using dependency injection, for example(依赖注入)
  - Constructor injection(构造器注入)
  - Parameter injection(参数注入)
  - Setter injection(setter方法注入)
  - Interface injection(接口注入)
- Using a contextualized lookup(使用上下文查找，如Java Beans技术)
- Using [template method design pattern](https://en.wikipedia.org/wiki/Template_method_design_pattern)(使用模板方法设计模式，JdbcTemplate中的callback)
- Using [strategy design pattern](https://en.wikipedia.org/wiki/Strategy_design_pattern)(使用策略设计模式)

# Java Beans

## Introspector 

提供了一种标准的工具来了解目标Java Bean支持的属性，事件和方法

全是静态方法

主要方法：

- public static BeanInfo getBeanInfo(Class<?> beanClass) ：获取beanClass的BeanInfo
- public static BeanInfo getBeanInfo(Class<?> beanClass, Class<?> stopClass) ：获取beanClass的BeanInfo，在分析中将忽略stopClass或其基类中的任何方法/属性/事件。 

## BeanInfo 

**接口**

使用BeanInfo接口创建一个BeanInfo类，并提供关于bean的方法，属性，事件和其他功能的显式信息

主要方法：

- BeanDescriptor getBeanDescriptor()
  - 返回提供关于该bean的整体信息的bean描述符，例如其显示名称或其定制器。

- BeanDescriptor[] getBeanDescriptors()：
  - 返回bean的所有属性的描述符。  

## PropertyDescriptor

描述了Java Bean通过一对访问器方法导出的一个属性。

主要方法：

- public Class<?> getPropertyType()
  - 返回属性的Java类型信息。

- public synchronized Method getReadMethod()
  - 获取应该用于读取属性值的方法

- public synchronized Method getWriteMethod()
  - 获取应用于写入属性值的方法。

# 依赖查找VS依赖注入

| 类型     | 依赖处理 | 实现便利性 | 代码入侵性   | API依赖性     | 可读性 |
| -------- | -------- | ---------- | ------------ | ------------- | ------ |
| 依赖查找 | 主动获取 | 相对繁琐   | 入侵业务逻辑 | 依赖容器API   | 良好   |
| 依赖注入 | 被动获取 | 相对便利   | 入侵性低     | 不依赖容器API | 一般   |

# setter注入VS构造器注入 

下面观点来源于《Expert One-on-One™ J2EE™ Development without EJB™》

## setter注入

### 优点

- JavaBean属性在ide中得到了很好的支持。
- JavaBean属性是自文档化的。
- JavaBean属性由子类继承，不需要任何代码。
- 如果需要，可以使用标准JavaBeans属性编辑器机制进行类型转换。
- 许多现有的JavaBean可以在一个面向JavaBean的IoC容器中使用，而无需修改。
- 如果每个setter都有相应的getter(使属性可读，也可写)，可以请求
- 当前配置状态的组件。
- 如果我们想要保持这种状态，这是特别有用的:例如，
  在XML表单或数据库中。使用构造函数注入，无法找到当前状态。
- Setter注入对于具有默认值的对象工作得很好，这意味着不是所有的属性都需要在
  运行时。

### 缺点

- 在调用顺序上没有对应的契约，但是Spring提供了`org.springframework.beans.factory.InitializingBean`接口来解决，可以在所有的属性设置完成之后，调用我们自定义的init方法

## 构造器

### 优点

- 可以同时对所有属性进行配置，无需担心顺序问题，在参数列表上已经指明顺序
- 无需调用init方法
- 代码量相对setter较少

### 缺点

- 容易产生**循环依赖**

- Java构造函数参数没有自省可见的名称。(java8之后得到改善)
- ide对构造函数参数列表的支持不如JavaBean setter方法好。
- 长构造函数参数列表和大型构造函数主体可能变得笨拙。
- 具体的继承可能会有问题。
- 与javabean相比，对可选属性的支持较差
- 单元测试可能稍微有些困难
- 当合作者被传递到对象构造时，就不可能更改保存在对象。

