---
title: SpringBoot或Cloud的普通类如何获取service实现
date: 2019-06-14 12:57:00 +0800 
layout: post
permalink: /blog/2019/06/14/SpringBoot或Cloud的普通类如何获取service实现.html
categories:
  - JAVA
tags:
  - JAVA
  - Spring Boot
---

#### Spring Boot或Cloud的普通类如何获取service实现
我们在便利使用`SpringBoot`或`SpringCloud`的时候,很多时候会遇到一种场景,想在`utils`的工具类中调用被`Spring`托管的`service`.那么我们该如何优雅的实现呢?
##### Num1: 使用PostConstruct
```

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

import javax.annotation.PostConstruct;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;

@Component
public class OrderUtils {

	@Autowired
	private OrderService orderService;

	public static OrderUtils orderUtils;

	public void setService(OrderService orderService) {
		this.orderService = orderService;
	}

	/**
	 * 核心
	 */
	@PostConstruct
	public void init() {
		orderUtils = this;
		orderUtils.orderService = this.orderService;
	}

	public static OrderVo getOrder(String orderId) throws Exception{
		OrderVo order= orderUtils.orderService.getOrderById(orderId);
		return order;
	}

}
```
被`@PostConstruct`修饰的方法会在服务器加载`Servlet`的时候运行，并且只会被服务器执行一次。`PostConstruct`在构造函数之后执行，`init（）`方法之前执行。因为注解`@PostConstruct`的缘故，在类初始化之前会先加载该使用该注解的方法；然后再执行类的初始化。

**注： 构造方法  ——> @Autowired —— > @PostConstruct ——> 静态方法 （按此顺序加载）**

使用此方法实现`service`调用有一个问题,若想对其他`service`进行调用,需将对应的`service`手动注入进来.
##### Num2:使用ApplicationContextAware
```
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;

@Component
public class SpringUtils implements ApplicationContextAware {

    private static ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringUtils.applicationContext = applicationContext;
    }

    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }

    /**
     * 对应的被管理类有别名时使用
     * @param name
     * @param <T>
     * @return
     * @throws BeansException
     */
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String name) throws BeansException {
        return (T) applicationContext.getBean(name);
    }
    
    /**
	 * 在对应的注解内未使用别名时 使用
	 */
	public static <T> T getBean(Class<T> clazz) {
		return applicationContext.getBean(clazz);
	}

}
```
此方法直接实现了`ApplicationContextAware`接口,调用`Spring`提供的`applicationContext`,将所有在`Spring`中有注解的类,均可通过`getBean`方式获取到.
