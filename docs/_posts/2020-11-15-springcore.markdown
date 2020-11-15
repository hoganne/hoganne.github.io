---
layout: post
title:  "spring Core"
date:   2020-11-15 12:39:53 +0800
categories: hoganne update
---
####  Understanding AOP Proxies

Spring AOP is proxy-based. It is vitally important that you grasp the semantics of what that last statement actually means before you write your own aspects or use any of the Spring AOP-based aspects supplied with the Spring Framework.

Spring AOP是基于代理的。在编写自己的方面或使用Spring框架随附的任何基于Spring AOP的方面之前，掌握最后一条语句实际含义的语义至关重要。

Consider first the scenario where you have a plain-vanilla, un-proxied, nothing-special-about-it, straight object reference, as the following code snippet shows:

首先考虑您有一个普通的，未经代理的，没有什么特别的，直接的对象引用的情况，如以下代码片段所示：

```java
public class SimplePojo implements Pojo {
    public void foo() {
        // this next method invocation is a direct call on the 'this' reference
        //此下一个方法调用是对“ this”引用的直接调用
        this.bar();
    }
    public void bar() {
        // some logic...
    }
}
```

If you invoke a method on an object reference, the method is invoked directly on that object reference, as the following image and listing show:

如果在对象引用上调用方法，则直接在该对象引用上调用该方法，如下图和清单所示：

![aop proxy plain pojo call](https://docs.spring.io/spring-framework/docs/current/reference/html/images/aop-proxy-plain-pojo-call.png)

```java
public class Main {

    public static void main(String[] args) {
        Pojo pojo = new SimplePojo();
        // this is a direct method call on the 'pojo' reference
        //这是对'pojo'引用的直接方法调用
        pojo.foo();
    }
}
```

Things change slightly when the reference that client code has is a proxy. Consider the following diagram and code snippet:

当客户端代码具有的引用是代理时，情况会稍有变化。考虑以下图表和代码片段：

![aop proxy call](https://docs.spring.io/spring-framework/docs/current/reference/html/images/aop-proxy-call.png)



```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

The key thing to understand here is that the client code inside the `main(..)` method of the `Main` class has a reference to the proxy. This means that method calls on that object reference are calls on the proxy. As a result, the proxy can delegate to all of the interceptors (advice) that are relevant to that particular method call. However, once the call has finally reached the target object (the `SimplePojo` reference in this case), any method calls that it may make on itself, such as `this.bar()` or `this.foo()`, are going to be invoked against the `this` reference, and not the proxy. This has important implications. It means that self-invocation is not going to result in the advice associated with a method invocation getting a chance to run.

此处要理解的关键是Main类的main（..）方法中的客户端代码具有对代理的引用。这意味着该对象引用上的方法调用是代理上的调用。结果，代理可以委派给与该特定方法调用相关的所有拦截器（建议）。但是，一旦调用最终到达目标对象（在本例中为SimplePojo引用），则可能会针对它调用可能对其自身进行的任何方法调用，例如this.bar（）或this.foo（）。此参考，而不是代理。这具有重要意义。这意味着自调用不会导致与方法调用相关的advice（增强）得到运行的机会。

Okay, so what is to be done about this? The best approach (the term "best" is used loosely here) is to refactor your code such that the self-invocation does not happen. This does entail some work on your part, but it is the best, least-invasive approach. The next approach is absolutely horrendous, and we hesitate to point it out, precisely because it is so horrendous. You can (painful as it is to us) totally tie the logic within your class to Spring AOP, as the following example shows:

好吧，那么该怎么办？最佳方法（此处宽松地使用术语“最佳”）是重构代码，以免发生自调用。这确实需要您做一些工作，但这是最好的，侵入性最小的方法。下一种方法绝对可怕，我们正要指出这一点，恰恰是因为它是如此可怕。您可以（对我们来说是痛苦的）完全将类中的逻辑与Spring AOP绑定在一起，如以下示例所示：

```java
public class SimplePojo implements Pojo {

    public void foo() {
        // this works, but... gah!
        ((Pojo) AopContext.currentProxy()).bar();
    }

    public void bar() {
        // some logic...
    }
}
```

This totally couples your code to Spring AOP, and it makes the class itself aware of the fact that it is being used in an AOP context, which flies in the face of AOP. It also requires some additional configuration when the proxy is being created, as the following example shows:

这将您的代码完全耦合到Spring AOP，并且使类本身意识到在AOP上下文中使用它这一事实，而AOP却是事实。创建代理时，还需要一些其他配置，如以下示例所示：

```java
public class Main {

    public static void main(String[] args) {
        ProxyFactory factory = new ProxyFactory(new SimplePojo());
        factory.addInterface(Pojo.class);
        factory.addAdvice(new RetryAdvice());
        factory.setExposeProxy(true);

        Pojo pojo = (Pojo) factory.getProxy();
        // this is a method call on the proxy!
        pojo.foo();
    }
}
```

Finally, it must be noted that AspectJ does not have this self-invocation issue because it is not a proxy-based AOP framework.

最后，必须指出，AspectJ没有此自调用问题，因为它不是基于代理的AOP框架。
