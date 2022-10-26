# Java代理的几种实现方式

## 1.分类

第一种：静态代理，只能静态的代理某些类或者某些方法，不推荐使用，功能弱，编码简单

第二种：动态代理，包含Proxy代理和CGLIB动态代理

### 1.2. Proxy代理，JDK内置的动态代理

特点：面向接口的，不需要导入三方依赖的动态代理，可以对多个不同的接口进行增强，通过反射读取注解时，只能读取到接口上的注解

原理：面向接口，只能对实现类在实现接口中定义的方法进行增强

### 1.3.CGLIB动态代理

特点：面向父类的动态代理，需要导入第三方依赖

原理：面向父类，底层通过子类继承父类并重写方法的形式实现增强

Proxy和CGLIB是非常重要的代理模式，是SpringAOP底层实现的主要两种方式

CGLIB的核心类：

`net.sf.cglib.proxy.Enhancer` - 主要的增强类

`net.sf.cglib.proxy.MethodInterceptor` - 主要的方法拦截类，它是Callback接口的子接口，需要用户实现

`net.sf.cglib.proxy.MethodProxy` - JDK的`java.lang.reflect.Method`类的代理类，可以方便的实现对源对象方法的调用，如使用：

`Obejct o = methodProxy.invokeSuper(proxy.args);//虽然第一个参数是被代理对象，也不会出现死循环的问题`

`net.sf.cglib.proxy.MethodInterceptor`接口是最通用的回调类型，它经常被基于代理的AOP用来实现拦截方法的调用。这个接口只定义了一个方法

```java
public Object intercept(Object object, java.lang.reflect.Method method, Object[] args, MethodProxy proxy) throws Throwable;
```

第一个参数是代理对象，第二和第三个参数分别是拦截的方法和方法的参数。原来的方法可能通过使用`java.lang.reflect.Method`对象的一般反射调用，或者使用`net.sf.cglib.proxy.MethodProxy`对象调用。`net.sf.cglib.proxy.MethodProxy`通常被首选使用，因为它更快。

