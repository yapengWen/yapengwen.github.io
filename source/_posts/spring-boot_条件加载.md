---
title: springboot根据不同的条件创建bean
date: 2017-10-17 11:37:00
tags: [springboot]
categories: java
---

## springboot根据不同的条件创建bean，动态创建bean，@Conditional注解使用

这个需求应该也比较常见，在不同的条件下创建不同的bean，具体场景很多，能看到这篇的肯定懂我的意思。
倘若不了解spring4.X新加入的@Conditional注解的话，要实现不同条件创建不同的bean还是比较麻烦的，可能需要硬编码一些东西做if判断。那么现在有个@Conditional注解后，事情就简单多了。用法很简单，直接上代码。
新建一个springboot项目，添加一个Configuration标注的类，我们通过不同的条件表达式来创建bean。

```java
package com.tianyalei.condition;

import org.springframework.boot.autoconfigure.condition.ConditionalOnBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnExpression;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Conditional;
import org.springframework.context.annotation.Configuration;

/**
 * Created by wuweifeng on 2017/10/11.
 */
@Configuration
public class Config {

    @Conditional(MyCondition.class)
    @Bean
    public String condition() {
        System.err.println("自定义的condition的match方法返回值为true时，才会进入该方法创建bean");
        return "";
    }

    /**
     * 该Abc class位于类路径上时
     */
    @ConditionalOnClass(Abc.class)
    @Bean
    public String abc() {
        System.err.println("ConditionalOnClass true");
        return "";
    }

//    @ConditionalOnClass(Abc.class)
//    @Bean
//    public Abc newAbc() {
//        System.err.println("ConditionalOnClass true");
//        return new Abc();
//    }

    /**
     * 存在Abc类的实例时
     */
    @ConditionalOnBean(Abc.class)
    @Bean
    public String bean() {
        System.err.println("ConditionalOnBean is exist");
        return "";
    }

    @ConditionalOnMissingBean(Abc.class)
    @Bean
    public String missBean() {
        System.err.println("ConditionalOnBean is missing");
        return "";
    }

    /**
     * 表达式为true时
     */
    @ConditionalOnExpression(value = "true")
    @Bean
    public String expresssion() {
        System.err.println("expresssion is true");
        return "";
    }

    /**
     * 配置文件属性是否为true
     */
    @ConditionalOnProperty(
            value = {"abc.property"},
            matchIfMissing = false)
    @Bean
    public String property() {
        System.err.println("property is true");
        return "";
    }
}
```
这里面有个空类Abc.class，你可以就创建个叫Abc的类，里面是空的就行。
```java
import org.springframework.context.annotation.Condition;
import org.springframework.context.annotation.ConditionContext;
import org.springframework.core.type.AnnotatedTypeMetadata;

/**
 * Created by wuweifeng on 2017/10/11.
 */
public class MyCondition implements Condition {
    @Override
    public boolean matches(ConditionContext conditionContext, AnnotatedTypeMetadata annotatedTypeMetadata) {
        //判断当前系统是Mac，Windows，Linux
        return conditionContext.getEnvironment().getProperty("os.name").contains("Mac");
    }
}
```
**@Conditional(MyCondition.class)**
这句代码可以标注在类上面，表示该类下面的所有@Bean都会启用配置
也可以标注在方法上面，只是对该方法启用配置

除了自己自定义Condition之外，Spring还提供了很多Condition给我们用
**@ConditionalOnBean**（仅仅在当前上下文中存在某个对象时，才会实例化一个Bean）
**@ConditionalOnClass**（某个class位于类路径上，才会实例化一个Bean）
**@ConditionalOnExpression**（当表达式为true的时候，才会实例化一个Bean）
**@ConditionalOnMissingBean**（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
**@ConditionalOnMissingClass**（某个class类路径上不存在的时候，才会实例化一个Bean）
**@ConditionalOnNotWebApplication**（不是web应用）
以上是一些常用的注解，其实就是条件判断，如果为true了就创建Bean，为false就不创建，就这么简单。
这些注解里的条件可以是多个，也可以赋默认值，也可以标注在类上，如果标注在类上，则对类里的所有@Bean方法都生效。
其中@ConditionalOnProperty是指在application.yml里配置的属性是否为true，其他的几个都是对class的判断。
我在配置里加上abc.property = true这个配置就可以测试上面的代码了。
然后再来一个对类进行多个条件标注的例子：
```java
package com.tianyalei.condition;

import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.condition.ConditionalOnProperty;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * Created by wuweifeng on 2017/10/11.
 */
@Configuration
@ConditionalOnProperty(
        value = {"abc.property"},
        matchIfMissing = false
)
@ConditionalOnClass(Abc.class)
public class Multi {
    @Bean
    @ConditionalOnMissingBean({Abc.class})
    public String check() {
        System.err.println("multi check");
        return "check";
    }
}
```
OK，代码很简单，运行看看结果
![](https://yapengwen.github.io/img/20171011114902806.jpg)
可能上面的那些你用的地方不常见，那我来举一个我正在使用的例子。我的应用是基于SpringCloud的，在线上部署时有eureka来做注册中心，而在本地环境下，我的应用是单机的，不需要eureka，但是代码里已经引入了eureka了，每次启动就会自动去连接eureka，然后控制台就开始报错。虽然不影响功能，但是看着一直不停的报错也是不顺眼。
那么我就可以使用Condition注解来解决它。
```java
/**
 * @author wuweifeng wrote on 2017/11/25.
 * 根据部署环境动态决定是否启用eureka
 */
@Component
@ConditionalOnProperty(value = "open.eureka")
@EnableDiscoveryClient
public class JudgeEnableDiscoveryClient {
}
```
我把EnableDiscoveryClient这个注解单独放个类里，里面什么也不写，条件就是application.yml里配置的open.eureka
如果我只想让线上的环境开启eureka，那么我就在application-prod.yml里配上open.eureka=true，其他的yml什么也不写就行了。这样本地启动时就相当于没有开启EnableDiscoveryClient。
