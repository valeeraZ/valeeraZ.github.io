---
title: Spring-Bean Scope: Singleton & Prototype
layout: post
subtitle: Singleton & Prototype
date:       2020-06-30
author:     "Zhao"
header-img: "img/Spring.jpg"
tags: 
   - Java
   - Design Pattern
   - Spring
---

When you create a bean definition, you create a recipe for creating actual instances of the class defined by that bean definition. The idea that a bean definition is a recipe is important, because it means that, as with a class, you can create many object instances from a single recipe.

You can control not only the various dependencies and configuration values that are to be plugged into an object that is created from a particular bean definition but also control the scope of the objects created from a particular bean definition. This approach is powerful and flexible, because you can choose the scope of the objects you create through configuration instead of having to bake in the scope of an object at the Java class level. Beans can be defined to be deployed in one of a number of scopes. The Spring Framework supports six scopes, four of which are available only if you use a web-aware `ApplicationContext`. You can also create [a custom scope.](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-custom)

| Scope                                                        | Description                                                  |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [singleton](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-singleton) | (Default) Scopes a single bean definition to a single object instance for each Spring IoC container. |
| [prototype](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-prototype) | Scopes a single bean definition to any number of object instances. |
| [request](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-request) | Scopes a single bean definition to the lifecycle of a single HTTP request. That is, each HTTP request has its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [session](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-session) | Scopes a single bean definition to the lifecycle of an HTTP `Session`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [application](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-scopes-application) | Scopes a single bean definition to the lifecycle of a `ServletContext`. Only valid in the context of a web-aware Spring `ApplicationContext`. |
| [websocket](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/web.html#websocket-stomp-websocket-scope) | Scopes a single bean definition to the lifecycle of a `WebSocket`. Only valid in the context of a web-aware Spring `ApplicationContext`. |

本文章介绍Spring的bean作用域的两种：Singleton单例模式和Prototype多例模式

# Singleton单例模式

一张图来看一下什么是单例模式。![](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/images/singleton.png)

当Bean的作用域为singleton时，Spring容器就只会存在一个共享的Bean实例，并且所有对Bean的请求，只要id相同就会返回同一个Bean实例。singleton作用域对于无会话的Bean是最合适的选择。

首先不要忘记引入maven依赖：`spring-core`, `spring-context`和单元测试`junit`。

准备两个Bean：Bean1和Bean2，其中Bean1依赖于Bean2。

```java
public class Bean1 {
    private Bean2 bean2;

    public Bean2 getBean2() {
        return bean2;
    }

    public void setBean2(Bean2 bean2) {
        this.bean2 = bean2;
    }

    @Override
    public String toString() {
        return "Bean1{" +
                "bean2=" + bean2 +
                '}';
    }
}
```

```java
public class Bean2 {}
```

用上一篇文章介绍过的setter注入模式来注入bean2：在spring.xml中插入如下配置

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="Bean2" id="bean2" scope="singleton"/>

    <bean class="Bean1" id="bean1">
        <property name="bean2" ref="bean2"/>
    </bean>
</beans>
```

可以看到，在Bean2的配置中，加入了`scope="singleton"`这一配置。单例模式对于Spring是缺省的。

写测试代码，来验证一下我们的理论。

```java
@Test
public void test(){
  ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
  Bean2 bean2_1 = context.getBean("bean2",Bean2.class);
  System.out.println("bean2_1 = " + bean2_1);
  Bean2 bean2_2 = context.getBean("bean2", Bean2.class);
  System.out.println("bean2_2 = " + bean2_2);
  Bean1 bean1 = context.getBean("bean1",Bean1.class);
  System.out.println("bean1 = " + bean1);
}
```

这里定义了两个Bean2和一个Bean1，其中Bean1中含有一个Bean2。按照我们之前的文章中的代码测试，三个Bean2应该是相同的。

```
bean2_1 = Bean2@87f383f
bean2_2 = Bean2@87f383f
bean1 = Bean1{bean2=Bean2@87f383f}
```

## 不同的context上下文

迄今为止，我们都是在同一个`ApplicationContext`上下文环境中获取Bean，那么如果运用单例模式，在多个上下文环境中尝试获取同一个Bean会怎样呢？

```java
@Test
    public void test(){
        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
        Bean2 bean2_1 = context.getBean("bean2",Bean2.class);
        System.out.println("bean2_1 = " + bean2_1);
        Bean2 bean2_2 = context.getBean("bean2", Bean2.class);
        System.out.println("bean2_2 = " + bean2_2);
        Bean1 bean1 = context.getBean("bean1",Bean1.class);
        System.out.println("bean1 = " + bean1);
      
        System.out.println("------");
      
        ApplicationContext context1 = new ClassPathXmlApplicationContext("spring.xml");
        Bean2 bean2_3 = context1.getBean("bean2",Bean2.class);
        System.out.println("bean2_3 = " + bean2_3);
        Bean2 bean2_4 = context1.getBean("bean2", Bean2.class);
        System.out.println("bean2_4 = " + bean2_4);
        Bean1 bean1_2 = context1.getBean("bean1",Bean1.class);
        System.out.println("bean1_2 = " + bean1_2);
    }
```

可以看到，我新建了一个`ApplicationContext`，让我们看一下输出结果：

```
bean2_1 = Bean2@87f383f
bean2_2 = Bean2@87f383f
bean1 = Bean1{bean2=Bean2@87f383f}
------
bean2_3 = Bean2@4671e53b
bean2_4 = Bean2@4671e53b
bean1_2 = Bean1{bean2=Bean2@4671e53b}
```

同一个上下文环境中的输出是一样的，不同的上下文环境获取的Bean就不一样。

The scope of the Spring singleton is best described as being per-container and per-bean. This means that, if you define one bean for a particular class in a single Spring container, the Spring container creates one and only one instance of the class defined by that bean definition. 

# Prototype 多例模式

The non-singleton prototype scope of bean deployment results in the creation of a new bean instance every time a request for that specific bean is made. That is, the bean is injected into another bean or you request it through a `getBean()` method call on the container. As a rule, you should use the prototype scope for all stateful beans and the singleton scope for stateless beans.

![](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/images/prototype.png)

(A data access object (DAO) is not typically configured as a prototype, because a typical DAO does not hold any conversational state. It was easier for us to reuse the core of the singleton diagram.)

每次通过容器的getBean方法获取prototype定义的Bean时，都将产生一个新的Bean实例。
对于singleton作用域的 Bean，每次请求该Bean都将获得相同的实例。容器负责跟踪Bean实例的状态，负责维护Bean实例的生命周期行为；如果一个Bean被设置成 prototype作用域，程序每次请求该id的Bean，Spring都会新建一个Bean实例，然后返回给程序。在这种情况下，Spring容器仅仅 使用new 关键字创建Bean实例，一旦创建成功，容器不在跟踪实例，也不会维护Bean实例的状态。
如果不指定Bean的作用域，Spring默认使用singleton作用域。Java在创建Java实例时，需要进行内存申请；销毁实例时，需要完成垃圾回收，这些工作都会导致系统开销的增加。因此，prototype作用域Bean的创建、销毁代价比较大。而singleton作用域的Bean实例一旦创建成功，可以重复使用。因此，除非必要，否则尽量避免将Bean被设置成prototype作用域。

要想试验我们的理论，只需要设置Bean2的`scope = prototype`就可以了。看一下测试结果：

```
bean2_1 = Bean2@87f383f
bean2_2 = Bean2@4eb7f003
bean1 = Bean1{bean2=Bean2@eafc191}
```

可以看到，3次调用Bean2的结果都不一样。每次向Spring上下文请求实例的时候，我们获得的都是一个新的实例。

# 单例模式Bean依赖于多例模式Bean

在我们的例子中，Bean1中有类型为Bean2的属性，也就是Bean1依赖于Bean2。  

如果Bean1是singleton，Bean2是prototype，那么每次请求Bean1时所包含的Bean2还相同吗？也就是如下这种情况：

```java
<bean class="Bean2" id="bean2" scope="prototype"/>

<bean class="Bean1" id="bean1" scope="singleton">
  <property name="bean2" ref="bean2"/>
</bean>
```

写一下测试代码，测试下多次请求Bean1的情况。

```java
@Test
public void test(){
  ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
  Bean1 bean1_1 = context.getBean("bean1",Bean1.class);
  System.out.println("bean1_1 = " + bean1_1);
  Bean1 bean1_2 = context.getBean("bean1",Bean1.class);
  System.out.println("bean1_2 = " + bean1_2);
  System.out.println("(bean1_1 == bean1_2) = " + (bean1_1 == bean1_2));

}
```

```
bean1_1 = Bean1{bean2=Bean2@87f383f}
bean1_2 = Bean1{bean2=Bean2@87f383f}
(bean1_1 == bean1_2) = true
```

可以看到，两次Bean2是同一个实例，两个Bean1也是相同的。Bean1是单例模式，因此当需要多次实例化Bean1时，Spring也只实例化唯一的Bean2来确保每个Bean1是相同的。

When you use singleton-scoped beans with dependencies on prototype beans, be aware that dependencies are resolved at instantiation time. Thus, if you dependency-inject a prototype-scoped bean into a singleton-scoped bean, a new prototype bean is instantiated and then dependency-injected into the singleton bean. The prototype instance is the sole instance that is ever supplied to the singleton-scoped bean.

如果我想让每次调用Bean1的*某个方法*拿到的Bean2都不同，该怎么办？

However, suppose you want the singleton-scoped bean to acquire a new instance of the prototype-scoped bean repeatedly at runtime. You cannot dependency-inject a prototype-scoped bean into your singleton bean, because that injection occurs only once, when the Spring container instantiates the singleton bean and resolves and injects its dependencies. If you need a new instance of a prototype bean at runtime more than once, see [Method Injection](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection)

## 方法注入之查找注入

A problem arises when the bean lifecycles are different. Suppose singleton bean A needs to use non-singleton (prototype) bean B, perhaps on each method invocation on A. The container creates the singleton bean A only once, and thus only gets one opportunity to set the properties. The container cannot provide bean A with a new instance of bean B every time one is needed.

Lookup method injection is the ability of the container to override methods on container-managed beans and return the lookup result for another named bean in the container. The lookup typically involves a prototype bean, as in the scenario described in [the preceding section](https://docs.spring.io/spring/docs/5.2.7.RELEASE/spring-framework-reference/core.html#beans-factory-method-injection). 

修改我们的Bean1代码，去掉对Bean2的依赖，并新加两个方法：

```java
public abstract class Bean1 {

    public void printBean2(){
        System.out.println("Bean2 = " + createBean2());
    }

    protected abstract Bean2 createBean2();
}
```

在spring.xml中修改：

```xml
<bean class="Bean2" id="bean2" scope="prototype"/>

<bean class="Bean1" id="bean1" scope="singleton">
  <lookup-method name="createBean2" bean="bean2" />
</bean>
```

测试一下，是否单例的Bean1，每次产生的多例Bean2，是不同的：

```java
@Test
public void test(){
  ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
  Bean1 bean1 = context.getBean("bean1",Bean1.class);
  bean1.printBean2();
  bean1.printBean2();
  Bean1 bean1_1 = context.getBean("bean1",Bean1.class);
  bean1_1.printBean2();
  bean1_1.printBean2();
  System.out.println("(bean1 == bean1_1) = " + (bean1 == bean1_1));
}
```

```
Bean2 = Bean2@4148db48
Bean2 = Bean2@282003e1
Bean2 = Bean2@7fad8c79
Bean2 = Bean2@71a794e5
(bean1 == bean1_1) = true
```

可以看到，每次产生的Bean2都是不同的，而两个Bean1是相同的，遵守了singleton模式。





