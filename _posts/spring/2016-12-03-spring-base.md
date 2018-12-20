---
layout: post
title: Spring 简介
categories: Spring
description: Spring 简介
keywords: Spring, java
---

# 特色

- J2EE without EJB
- IOC/DI & AOP(aspect oriented programming)
- light weight
- container,容器,包含并管理对象声明周期
- framework,框架
- 轻量级，非侵入性
- 丰富模块，整理如下：

![image](https://github.com/stdupanda/stdupanda.github.io/raw/master/images/posts/springmodules.png)

# IOC基础

## Bean 配置形式

xml annotation

## Bean 配置方法

全类名（反射）、通过工厂方法（静态工厂方法 & 实例工厂方法）、FactoryBean

### 注意点

```xml

<?xml version="1.0" encoding="UTF-8"?>
   <beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xmlns:p="http://www.springframework.org/schema/p"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
           http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-4.1.xsd">
<!-- 配置bean
    class：bean的全类名，通过反射的方式在IOC容器中创建Bean，所以要求Bean中必须有无参数的构造器
    id：标识容器中的bean，id唯一,id 可以指定多个名字，名字之间可用逗号、分号、或空格分隔
。
-->
<bean id="helloSpring" class="com.yl.HelloSpring">
    <property name="name" value="Spring"></property>
</bean>
<!-- 使用构造器注入属性值可以指定参数的位置和参数的类型，以区分重载的构造器 -->
<bean id="car" class="com.yl.Car">
    <constructor-arg value="Audi" index="0"></constructor-arg>
    <constructor-arg value="Shanghai" index="1"></constructor-arg>
    <constructor-arg value="300000" type="double"></constructor-arg>
</bean>

<bean id="car2" class="com.yl.Car">
    <constructor-arg value="BMW" type="java.lang.String"></constructor-arg>
    <!-- 如果字面值包含特殊字符，可以使用<![CDATA[]]> 包裹起来-->
    <!-- 属性值还可以使用value子节点进行配置 -->
    <constructor-arg type="java.lang.String">
        <value><![CDATA[<ShangHai~>]]></value>
    </constructor-arg>
    <constructor-arg value="200" type="int"></constructor-arg>
</bean>


<bean id="person" class="com.yl.Person">
    <property name="name" value="Tom"></property>
    <property name="age" value="24"></property>
    <!-- 可以使用property的ref属性建立bean之间的引用关系 -->
    <!-- <property name="car" ref="car2"></property> -->
    <!-- <property name="car">
        <ref bean="car2"/>
    </property> -->
    <!-- 内部Bean，不能被外部引用，只能在内部使用  -->
    <property name="car">
        <bean class="com.yl.Car">
            <constructor-arg value="Ford"></constructor-arg>
            <constructor-arg value="BeiJing"></constructor-arg>
            <constructor-arg value="100000"></constructor-arg>
        </bean>
    </property>
</bean>


<bean id="person2" class="com.yl.Person">
    <constructor-arg value="Jerry"></constructor-arg>
    <constructor-arg value="25"></constructor-arg>
    <!-- <constructor-arg ref="car"></constructor-arg> -->
    <!-- 测试null值 -->
    <!-- <constructor-arg><null/></constructor-arg> -->
    <!-- 为级联属性赋值。注意：属性需要先初始化后才可以为级联属性赋值，否则会有异常 -->
    <constructor-arg ref="car"></constructor-arg>
    <property name="car.speed" value="260"></property>
</bean>


<!-- 测试集合属性 -->
<bean id="person3" class="com.yl.collections.Person">
    <property name="name" value="Mike"></property>
    <property name="age" value="30"></property>
    <property name="cars">
        <!-- 使用list节点为List类型的属性赋值 -->
        <list>
            <ref bean="car"/>
            <ref bean="car2"/>
            <bean class="com.yl.Car">
                <constructor-arg value="Ford"></constructor-arg>
                <constructor-arg value="BeiJing"></constructor-arg>
                <constructor-arg value="100000"></constructor-arg>
            </bean>
        </list>
    </property>
</bean>

<!-- 配置Map属性值 -->
<bean id="newPerson" class="com.yl.collections.NewPerson">
    <property name="name" value="Rose"></property>
    <property name="age" value="24"></property>
    <property name="cars">
        <!-- 使用map节点机map的entry子节点配置Map类型的成员变量 -->
        <map>
            <entry key="AA" value-ref="car"></entry>
            <entry key="BB" value-ref="car2"></entry>
        </map>
    </property>
</bean>

<!-- 配置Properties属性值 -->
<bean id="dataSource" class="com.yl.collections.DataSource">
    <property name="properties">
        <!-- 使用props和prop子节点来为Properties属性赋值 -->
        <props>
            <prop key="user">root</prop>
            <prop key="password">1234</prop>
            <prop key="jdbcUrl">jdbc:mysql:///test</prop>
            <prop key="driverClass">com.mysql.jdbc.Driver</prop>
        </props>
    </property>
</bean>

<!-- 配置单例的集合bean，以供多个bean进行引用.需要导入util命名空间 -->
<util:list id="cars">
    <ref bean="car"/>
    <ref bean="car2"/>
</util:list>

<bean id="person4" class="com.yl.collections.Person">
    <property name="name" value="Jack"></property>
    <property name="age" value="25"></property>
    <property name="cars" ref="cars"></property>
</bean>

<!-- 通过p命名空间为bean的属性赋值，需要先导入p命名空间,相对于传统的方式，更加简洁 -->
<bean id="person5" class="com.yl.collections.Person" p:name="Queen" p:age="27" p:cars-ref="cars"></bean>

</beans>

```
