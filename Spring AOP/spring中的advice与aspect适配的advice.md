

# Aspect中的执行顺序

```java
@Before
try{
    目标方法
    @AfterReturning
} catch(Throwable ex) {
    @AfterThrowing
}finally{
    @After
}
```



# 整个interceptor调用顺序

1. ExposeInvocationInterceptor(暴露到当前线程中)
2. ThrowsAdviceInterceptor(**AspectJAfterThrowingAdvice**)(异常通知)
3. AfterReturningAdviceInterceptor(AspectJAfterReturningAdvice )(最终通知)
4. AspectJAfterAdvice (返回通知)
5. methodBeforeAdviceInterceptor(前置通知)

使用Acpect注解时的调用顺序

1. ExposeInvocationInterceptor(暴露到当前线程中)
2. AspectJAfterThrowingAdvice (异常通知)
3. AfterReturningAdviceInterceptor(最终通知)
4. AspectJAfterAdvice (返回通知)
5. methodBeforeAdviceInterceptor(前置通知)

# Spring中的Advice接口

> 其中只有**MethodBeforeAdvice**和**AfterReturningAdvice**是有接口方法的
>
> 所有对Aspect的支持只有**AspectJMethodBeforeAdvice** 和**AspectJAfterReturningAdvice** 实现了xxxAdvice接口，其他的都实现的**MethodInterceptor**接口(父接口Interceptor,Advice,都是标记接口)

```java
public interface BeforeAdvice extends Advice {
}
public interface MethodBeforeAdvice extends BeforeAdvice {
	void before(Method method, Object[] args, @Nullable Object target) throws Throwable;
}

public interface AfterAdvice extends Advice {
}
public interface AfterReturningAdvice extends AfterAdvice {
	void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable;
}

public interface ThrowsAdvice extends AfterAdvice {
}
```



# Spring中的MethodInterceptor

## 说明

> 都实现了MethodInterceptor接口，其中每个invoke的方法中都调用了对应的xxxAdvice接口的方法

## MethodBeforeAdviceInterceptor

```java
private final MethodBeforeAdvice advice;
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    this.advice.before(mi.getMethod(), mi.getArguments(), mi.getThis());
    return mi.proceed();
}
```

## AfterReturningAdviceInterceptor

```java
private final AfterReturningAdvice advice;
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    Object retVal = mi.proceed();
    this.advice.afterReturning(retVal, mi.getMethod(), mi.getArguments(), mi.getThis());
    return retVal;
}
```

## ThrowsAdviceInterceptor

```java
private final Object throwsAdvice;
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
    try {
        return mi.proceed();
    }
    catch (Throwable ex) {
        Method handlerMethod = getExceptionHandler(ex);
        if (handlerMethod != null) {
            invokeHandlerMethod(mi, ex, handlerMethod);
        }
        throw ex;
    }
}
```

# 对Aspect的支持

# 说明

> 对Aspect支持的类都实现了Spring中特定的Advice接口，用于通过反射调用使用注解标注的用户自定义的方法，但是**AspectJAfterThrowingAdvice** (异常通知)没有实现对应的**ThrowsAdvice**(标记接口)，但实现了**MethodInterceptor**接口

## AspectJMethodBeforeAdvice implements MethodBeforeAdvice

```java
@Override
public void before(Method method, Object[] args, @Nullable Object target) throws Throwable {
   invokeAdviceMethod(getJoinPointMatch(), null, null);
}
```

## AspectJAfterAdvice implements MethodInterceptor, AfterAdvice

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
   try {
      return mi.proceed();
   }
   finally {
      invokeAdviceMethod(getJoinPointMatch(), null, null);
   }
}
```

## AspectJAfterReturningAdvice implements AfterReturningAdvice, AfterAdvice

> AfterAdvice是个标记接口

```java
@Override
public void afterReturning(@Nullable Object returnValue, Method method, Object[] args, @Nullable Object target) throws Throwable {
   if (shouldInvokeOnReturnValueOf(method, returnValue)) {
      invokeAdviceMethod(getJoinPointMatch(), returnValue, null);
   }
}
```

## AspectJAfterThrowingAdvice implements MethodInterceptor, AfterAdvice

```java
@Override
public Object invoke(MethodInvocation mi) throws Throwable {
   try {
      return mi.proceed();
   }
   catch (Throwable ex) {
      if (shouldInvokeOnThrowing(ex)) {
         invokeAdviceMethod(getJoinPointMatch(), null, ex);
      }
      throw ex;
   }
}
```