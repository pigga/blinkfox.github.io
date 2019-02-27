以下配置基于spring 1x 

Xml代码  
```
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE beans PUBLIC "-//SPRING//DTD BEAN//EN" "http://www.springframework.org/dtd/spring-beans.dtd">  
<beans>  
    <bean id="TxProxy"   
        class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean" abstract="true">  
        <property name="transactionManager" ref="txManager"/>  
        <!-- property name="proxyTargetClass" value="true"/-->  
        <property name="transactionAttributes">  
            <props>  
                <prop key="save*">PROPAGATION_REQUIRED,-SQLException</prop>  
            </props>  
        </property>  
    </bean>  
    <bean id="abstractService" class="com.cuishen.testDao.service.AbstractService" init-method="init" destroy-method="destroy" abstract="true">  
        <property name="dao"><ref bean="dao"/></property>  
        <property name="configCacher"><ref bean="configCacher"/></property>  
    </bean>  
    <bean id="sysService" parent="TxProxy">  
        <property name="target">  
                <bean class="com.cuishen.testDao.service.SysService" parent="abstractService">  
                </bean>  
        </property>  
    </bean>  
    <bean id="petService" parent="TxProxy">  
        <property name="target">  
                <bean class="com.cuishen.testDao.service.impl.PetServiceImpl" parent="abstractService"/>  
        </property>  
    </bean>  
</beans>  
```
在实际使用中 parent="abstractService" 这个父类可以没有，在本例中将dao注入这个父类，然后为子类提供模板服务。 上面的petService和sysService分别是基于接口的service实现和普通类的service实现，注意上面配置中注释掉的一句： 

Xml代码  
```
<!-- property name="proxyTargetClass" value="true"/-->  
```
通过测试发现：如果去掉注释的话，spring对于接口和普通类的代理都是通过CGLIB来完成；如果注释掉这句，spring对于接口和普通类的代理将区分对待，即分别用Proxy和CGLIB来代理。 

还有要注意的一点是：spring对于接口的代理可以允许接口的实现类有private的构造方法；而基于CGLIB的对于普通类的代理，构造方法必须是公有的或者是默认的，否则会报错！！ 

如果目标对象没有实现任何接口，Spring使用CGLIB库生成目标对象的子类。在创建这个类的时候，Spring将通知织入，并且将对目标对象的调用委托给这个子类。在类没有实现任何接口，并且没有默认构造函数的情况下，通过构造函数注入时，目前的Spring是无法实现AOP切面拦截的。 

_spring aop 声明式事务的策略有：_ 
###### 1、  PROPAGATION_REQUIRED -- 支持当前的事务，如果不存在就创建一个新的。这是最常用的选择。 
###### 2 、 PROPAGATION_SUPPORTS -- 支持当前的事务，如果不存在就不使用事务。 
###### 3 、 PROPAGATION_MANDATORY -- 支持当前的事务，如果不存在就抛出异常。 
###### 4 、 PROPAGATION_REQUIRES_NEW -- 创建一个新的事务，并暂停当前的事务（如果存在）。 
###### 5 、 PROPAGATION_NOT_SUPPORTED -- 不使用事务，并暂停当前的事务（如果存在）。 
###### 6 、 PROPAGATION_NEVER -- 不使用事务，如果当前存在事务就抛出异常。 
###### 7 、 PROPAGATION_NESTED -- 如果当前存在事务就作为嵌入事务执行，否则与 PROPAGATION_REQUIRED 类似。 