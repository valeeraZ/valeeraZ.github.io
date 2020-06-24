---
title: Spring bean debut
layout: post
subtitle: use Spring's bean configuration
date:       2020-06-04
author:     "Zhao"
header-img: "img/Spring.jpg"
tags: 

   - Java
   - Spring
---

# 使用Spring的Bean管理

新建maven工程，引入如下依赖

```xml
<dependencies>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.2.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.2.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
</dependencies>
```

在Java文件夹下，新建我们要使用的Bean的class

```Java
public class Bean {
    public Bean(){
        System.out.println("Bean");
    }
}
```

将我们的bean交由Spring管理：在ressources文件夹下，新建配置文件spring.xml（这一步可以在Intellij IDEA中右键新建XML Configuration file - Spring config）并加入我们的bean

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="bean" class="Bean"></bean>
</beans>
```

测试我们的成果：在test文件夹下，新建一个JUnit测试

```java
import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class test {

    @Test
    public void test(){
        //获取Spring上下文
        ApplicationContext context = new ClassPathXmlApplicationContext("spring.xml");
        //获取bean
        Bean bean = context.getBean("bean",Bean.class);
        System.out.println("bean = " + bean);
    }
}
```



