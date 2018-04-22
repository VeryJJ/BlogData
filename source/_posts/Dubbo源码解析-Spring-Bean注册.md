---
title: Dubbo源码解析-Spring Bean注册
tags: [Dubbo]
date: 2018-04-22 19:34:18
categories: 技术
---

相信大家对于Dubbo Provider/Consumer的配置非常熟练，但这背后的实现原理清楚吗？如果有不太清楚的朋友，可以再往下阅读下。

<!-- more -->

## 知识点

+ 自定义 Spring XML Bean机制

---

## 背景

据我们所知

Spring 注解方式声明Bean的方式是在Class上打上@Component注解（@Component的扩展注解也可），当Spring容器启动时，Spring会自动扫面所有带有@Component注解的Class，自动注册到Bean容器中。

例如：

```java
@Component
public class Student {
    //do something
}
```

也可以通过XML文件配置的方式声明Bean

例如：

```xml
<bean id="Student" class="com.test.spring.beans.Student"></bean>
```


但，回过头来看Dubbo，我们并没有通过上述的方式去声明Dubbo配置中的Bean，却也能像使用Spring Bean一样用@Autowire去注入Dubbo服务的Bean，这其中的原理是什么呢？让我们从源码中找答案。

---

## 原理

### 启动Spring容器

在SpringBoot+Dubbo的搭配中，Java应用的启动入口main方法一般会这么写。通过此步骤去启动Java程序并将Dubbo Bean注册Spring容器。

```java
package com.sample;

@ComponentScan(basePackages = {
        "com.sample.myapp"
})

@SpringBootApplication
@EnableScheduling
public class MyApplication {
    public static void main(String[] args) {
        //启动Spring容器
        SpringApplication application = new SpringApplication(MyApplication.class,  
                "classpath:/spring/dubbo-config.xml");  //指定Dubbo配置文件
        application.run(args);
    }
}

```

### Spring如何识别Dubbo 自定义Bean标签

Spring为了支持用户自定义类加载到Spring容器，提供了org.springframework.beans.factory.xml.NamespaceHandler接口和org.springframework.beans.factory.xml.NamespaceHandlerSupport抽象类，NamespaceHandler#init方法会在对象的构造函数调用之后、属性初始化之前被DefaultNamespaceHandlerResolver调用。dubbo的DubboNamespaceHandler类正是继承了NamespaceHandlerSupport，其代码实现如下：

```java
public class DubboNamespaceHandler extends NamespaceHandlerSupport {

    static {
        Version.checkDuplicate(DubboNamespaceHandler.class);
    }

    public void init() {
        registerBeanDefinitionParser("application", new DubboBeanDefinitionParser(ApplicationConfig.class, true));
        registerBeanDefinitionParser("module", new DubboBeanDefinitionParser(ModuleConfig.class, true));
        registerBeanDefinitionParser("registry", new DubboBeanDefinitionParser(RegistryConfig.class, true));
        registerBeanDefinitionParser("monitor", new DubboBeanDefinitionParser(MonitorConfig.class, true));
        registerBeanDefinitionParser("provider", new DubboBeanDefinitionParser(ProviderConfig.class, true));
        registerBeanDefinitionParser("consumer", new DubboBeanDefinitionParser(ConsumerConfig.class, true));
        registerBeanDefinitionParser("protocol", new DubboBeanDefinitionParser(ProtocolConfig.class, true));
        registerBeanDefinitionParser("service", new DubboBeanDefinitionParser(ServiceBean.class, true));
        registerBeanDefinitionParser("reference", new DubboBeanDefinitionParser(ReferenceBean.class, false));
        registerBeanDefinitionParser("annotation", new AnnotationBeanDefinitionParser());
    }

}
```

registerBeanDefinitionParser方法使用的是父抽象类NamespaceHandlerSupport的默认实现，第一个参数是elementName，即元素名称，即告诉Spring你要解析哪个标签，第二个参数是BeanDefinitionParser的实现类，BeanDefinitionParser是Spring用来将xml元素转换成BeanDefinition对象的接口。dubbo的DubboBeanDefinitionParser类就实现了这个接口，负责将标签转换成bean定义对象BeanDefinition。


所以，以后想要了解Dubbo Bean初始化相关细节，可以查看DubboBeanDefinitionParser#parse的代码实现。

例如：

+ Dubbo Bean 会有哪些默认设置
    - dubbo服务提供者使用dubbo:service标签时，如果既不设置id，也不设置name，则dubbo给ServiceBean在Spring容器中定义的ID是什么？
+ Dubbo xml文件中的配置是怎么作用到Dubbo Bean中去的


#### 关于NamespaceHandlerSupport

+ spring.handlers           # 指定xml namespace的解析handler类
+ spring.schemas            # 指定xml xsd文件位置
+ dubbo.xsd                 # 设计你要的xml配置格式
+ DubboNamespaceHandler     # 自定义NamespaceHandler,完成从xml中读取配置内容，并转换成Spring Bean进行注册

Spring容器会默认加载classpath/META-INF下的spring.handlers和spring.schemas两个文件，来加载xsd和对应的NamespaceHandler,所以dubbo-config-spring包下的META-INF目录下也有这两个文件


---

## 练习DEMO

**1. 设计配置属性和JavaBean**

设计好配置项，并通过JavaBean来建模，本例中需要配置People实体，配置属性name和age（id是默认需要的）

```java
public class People {  
    private String id;  
    private String name;  
    private Integer age;  
}  
```

**2. 编写XSD文件**

为上一步设计好的配置项编写XSD文件，XSD是schema的定义文件，配置的输入和解析输出都是以XSD为契约，本例中XSD如下

```xml
<?xml version="1.0" encoding="UTF-8"?>  
<xsd:schema   
    xmlns="http://veryjj/cutesource/schema/people"  
    xmlns:xsd="http://www.w3.org/2001/XMLSchema"   
    xmlns:beans="http://www.springframework.org/schema/beans"  
    targetNamespace="http://veryjj/cutesource/schema/people"  
    elementFormDefault="qualified"   
    attributeFormDefault="unqualified">  
    <xsd:import namespace="http://www.springframework.org/schema/beans" />  
    <xsd:element name="people">  
        <xsd:complexType>  
            <xsd:complexContent>  
                <xsd:extension base="beans:identifiedType">  
                    <xsd:attribute name="name" type="xsd:string" />  
                    <xsd:attribute name="age" type="xsd:int" />  
                </xsd:extension>  
            </xsd:complexContent>  
        </xsd:complexType>  
    </xsd:element>  
</xsd:schema> 
```

关于xsd:schema的各个属性具体含义就不作过多解释，可以参见http://www.w3school.com.cn/schema/schema_schema.asp

<xsd:element name="people">对应着配置项节点的名称，因此在应用中会用people作为节点名来引用这个配置

<xsd:attribute name="name" type="xsd:string" />和<xsd:attribute name="age" type="xsd:int" />对应着配置项people的两个属性名，因此在应用中可以配置name和age两个属性，分别是string和int类型

完成后需把xsd存放在classpath下，一般都放在META-INF目录下（本例就放在这个目录下）

**3. 编写NamespaceHandler和BeanDefinitionParser完成解析工作**

```java
 
public class MyNamespaceHandler extends NamespaceHandlerSupport {  
    public void init() {  
        registerBeanDefinitionParser("people", new PeopleBeanDefinitionParser());  
    }  
}  

public class PeopleBeanDefinitionParser extends AbstractSingleBeanDefinitionParser {  
    protected Class getBeanClass(Element element) {  
        return People.class;  
    }  
    protected void doParse(Element element, BeanDefinitionBuilder bean) {  
        String name = element.getAttribute("name");  
        String age = element.getAttribute("age");  
        String id = element.getAttribute("id");  
        if (StringUtils.hasText(id)) {  
            bean.addPropertyValue("id", id);  
        }  
        if (StringUtils.hasText(name)) {  
            bean.addPropertyValue("name", name);  
        }  
        if (StringUtils.hasText(age)) {  
            bean.addPropertyValue("age", Integer.valueOf(age));  
        }  
    }  
}  
```

**4. 编写spring.handlers和spring.schemas串联起所有部件**

spring提供了 spring.handlers和spring.schemas这两个配置文件来完成这项工作，这两个文件需要我们自己编写并放入META-INF文件夹 中，这两个文件的地址必须是META-INF/spring.handlers和META-INF/spring.schemas，spring会默认去 载入它们，本例中spring.handlers如下所示：

spring.handlers

```
http\://veryjj/cutesource/schema/people=study.schemaExt.MyNamespaceHandler
```


spring.schemas

```
http\://veryjj/cutesource/schema/people.xsd=META-INF/people.xsd
```

以上就是载入xsd文件

**5. 使用自定义schema定义Spring Bean**

到此为止一个简单的自定义配置以完成，可以在具体应用中使用了。使用方法很简单，和配置一个普通的spring bean类似，只不过需要基于我们自定义schema，本例中引用方式如下所示：

```xml
<beans xmlns="http://www.springframework.org/schema/beans"  
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"   
    xmlns:cutesource="http://veryjj/cutesource/schema/people"  
    xsi:schemaLocation="  
http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd  
http://veryjj/cutesource/schema/people http://veryjj/cutesource/schema/people.xsd">  
    <cutesource:people id="cutesource" name="黄老师" age="27"/>  
</beans> 
```

其中xmlns:cutesource="http://veryjj/cutesource/schema/people" 是用来指定自定义schema，xsi:schemaLocation用来指定xsd文件。<cutesource:people id="cutesource" name="黄老师" age="27"/>是一个具体的自定义配置使用实例。

**6. 注入自定义schema定义的Spring Bean**

跟Spring Bean的注入方式完全一样，按你喜欢的方式来。

---

