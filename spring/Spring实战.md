

# █ Spring 核心

# 一. 概述

Spring 是一个开源的, 轻量级的开源框架, 最早由 Rod Johnson 创建, 致力于简化 java 开发, 而且是全方位的简化 java 开发. Spring 提供了 **IOC** 和 **AOP** 功能, 能降低项目组件间的耦合度, 便于日后维护与升级. 其本质是管理软件中的对象, 即创建并维护对象之间的关系

Spring 为系统提供了一个整体的解决方案, 开发者可以利用它本身提供的功能, 也可以与第三方框架和技术整合应用.

- 基于**POJO**的轻量级和**最小侵入式**编程
- 通过依赖注入和面向接口实现**松耦合**
- 基于 AOP 切面和惯例进行**声明式编程**
- 通过 AOP 切面和模板 template 减少样板式代码

## 1. POJO 最小侵入式

许多框架, 要求应用代码继承框架的某些类, 或是实现框架提供的某些接口, 从而导致应用和框架的绑定.

而 Spring 尽量避免应用代码与框架 API 发生耦合, 不需要继承 spring 的类或实现 spring 的接口. 大多数应用组件都只需要使用 spring 提供的注解, 但组件本质上仍是一个 POJO.

## 2. 依赖注入

处理实际业务的应用, 都会由多个类组成, 这些类相互协作来完成特定的业务逻辑.

耦合具有两面性: 

- 一方面, 紧密耦合的代码难以测试, 难以复用, 难以理解. 
- 另一方面, 一定程度的耦合又是必须的, 不同的类需要以适当的方式进行交互

传统做法中, 每个类需要管理与自己相互协作的对象 (即它所依赖的对象) 的引用. 会导致两个类的高度耦合. 而通过**依赖注入 (Dependency Injection, DI**), 对象的依赖关系将由系统中的专门组件负责协调, 在创建对象的时候进行设定.

创建应用组件之间协作的行为称为**装配 (wiring)**, Spring 通过**应用上下文 (Application Context)**全权负责对象的创建和组装, 装载 bean 的定义. Spring 提供了多种应用上下文的实现, 区别在于如何加载配置. 

## 3. 面向切面编程

DI 依赖注入让相互协作的软件组件保持松散耦合, 而**面向切面编程 AOP**则允许开发者把遍布应用各处的功能分离出来, 形成可重用的组件.

在一个完整的系统中, 除了实现自身核心的业务功能之外, 还需要许多诸如日志, 事务管理, 安全校验等系统服务的功能, 这些系统服务常常融入到自身具有核心业务逻辑的组件中去. 这样的系统服务通常被称为横切关注点, 因为他们会跨越系统的多个组件, 可能会带来额外的复杂性 

- 实现系统关注点功能的代码会重复出现在多个组件中
- 业务功能组件会因为那些与业务逻辑无关的代码而变得混乱

AOP 能够使这些服务模块化, 并以声明的方式将它们应用到需要影响的组件中去. 

## 4. 通过模板消除样板代码

spring 为了消除样板代码, 封装了多个模板. 通过使用 spring 提供的 jdbcTemplate, jmsTemplate 等模板, 可以大大减少样板代码的出现.

# 二. 容器与 Bean

## 1.容器

### 1.1. 容器的分类

在 Spring 应用中, 对象存在于 spring **容器 (container)**中. 容器是 spring 框架的核心, 负责创建对象, 装配对象, 配置并管理它们的整个生命周期. spring 容器使用 DI 管理构成应用的组件, 它会创建相互协作的组件之间的关联. 

spring 自带多个容器实现, 分成两种类型

- **bean 工厂 (BeanFactory)**, 最简单的容器, 提供基本的 DI 支持.
- **应用上下文 (ApplicationContext)**, 基于工厂构建, 并提供应用框架级别的服务.

### 1.2. 应用上下文

通常使用应用上下文. 不同的应用上下文作用类似, 区别在于其配置方式的不同

- **Annotation**ConfigApplicationmContext: 通过 java 配置的 spring 应用上下文

- **Annotation**Config**Web**ApplicationContext: 通过 java 配置的 spring web 应用上下文

- **ClassPathXml**ApplicationContext: 通过 `classpath` 下的 xml 配置

- **FileSystemXml**ApplicationContext: 通过文件系统中的 xml 配置

- **XmlWeb**ApplicationContext: 由 xml 配置的 web 应用上下文

```java
// 通过文件系统中的 xml 配置文件创建应用上下文
ApplicationContext context1 = new FileSystemXmlApplicationContext("c:/fileSystem.xml");

// 通过 classpath 下的 xml 配置文件创建应用上下文
ApplicationContext context2 = new ClassPathXmlApplicationContext("classpath.xml");

// 通过 java 配置类创建应用上下文
ApplicationContext context3 = new AnnotationConfigApplicationmContext("Config.class");
```

不论使用什么形式的应用上下文, 其主要目的有两个

- 对应用上下文本身进行设置, 开启哪些功能, 如声明式事务, 组件扫描, 切面支持等
- 声明需要注册到该应用上下文的 bean

java 配置方式由于更接近 java 编码方式, 并能提供类型检查, 更加安全, **推荐使用 java 配置方式**.

#### 1) xml 配置

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
            http://www.springframework.org/schema/beans
            		http://www.springframework.org/schema/beans/spring-beans.xsd
            http://www.springframework.org/schema/context
            		http://www.springframework.org/schema/context/spring-context.xsd">
    <!-- 应用上下文配置 -->
    <context:component-scan base-package="com.luojbin.spring.demo" />
    
    <!-- bean 声明 -->
    <bean id="date" class="java.util.Date"/>
</beans>
```

spring 应用上下文的 xml 配置文件如上所示, 主要包括以下几个部分

- `<beans>`: 表示该 xml 是spring 应用上下文配置文件, 并声明该配置文件要使用的命名空间
- `<context:component-scan>`: 应用上下文的配置内容之一, 指定组件扫描的包路径
- `<bean>`: 注册到该应用上下文的 bean

#### 2) java 配置

```java
@Configuration
@ComponentScan(basePackages = {"com.luojbin.spring.demo"})
public class RootConfig {
    @Bean
    public Date date() {
        return new Date();
    }
}
```

应用上下文的java 配置类, 与普通的 java 类区别不大, 但多了几个注解

- `@Configuration`: 表示该类是 spring 应用上下文的配置类
- `@ComponentScan`: 应用上下文配置内容之一, 指定组件扫描的包路径.
- `@Bean` 表示该方法的返回值要注册到该应用上下文中.

## 2. Bean

由 Spring 管理的这些组件/对象, 被称为 **Bean**. 不论应用上下文是以什么方式装载, 通过应用上下文获取 bean 对象的过程都是相似的: 先获取相应的应用上下文, 然后就可以通过应用上下文的 `getBean()` 方法从 spring 容器中获取 bean.

1. 首先在容器配置文件中添加 Bean 定义

   ```xml
   <bean id="BeanId" class="包名.类名" />
   ```

2. 在创建容器实例后, 调用 getBean() 方法获取 Bean 的实例. 

   其中`类名.class` 是通过反射, 获取方法区中的 class 对象, 用于指定获取对象的类型

   ```java
   // 通过 classpath 下的 xml 配置文件创建应用上下文
   ApplicationContext ac = new ClassPathXmlApplicationContext("classpath.xml");
   // 指定 beanId 和类型来获取容器中的 bean
   ac.getBean("BeanId", ClassA.class)
   // 如果一个类只有一个对应的 BeanId, 则 BeanId 可以省略, 即
   ac.getBean(ClassB.class)
   ```

### 2.1. Bean 的创建

Spring 采用了工厂模式. 

从本质上讲, Spring 的容器是一个维护 Bean 定义以及相互依赖关系的高级工厂接口, 通过指定 beanID, 并给定相应的类型, `BeanFactory` 或 `ApplicationContext` 就可以访问 Bean 定义, 获取 Bean 对象.

- 工厂模式 : 采用工厂方法去获取对象, 而不使用常规的 new
- 工厂方法 : 一类特殊的方法, 可以根据要求返回一个对象

  - 静态工厂方法 : 由 static 修饰, 调用时不需创建所在类的对象
  - 实例工厂方法 : 需要先创建所在类的一个工厂对象, 再由工厂对象去调用获取实例方法

Spring 可以通过以下三种方式配置并获取 Bean 对象

```xml
<!-- 1.指定 Bean 类型, 通过构造器获取对象, 默认使用无参构造, 提供参数时使用有参构造 -->
<bean id="date" class="java.util.Date" />	
 
<!-- 2.指定工厂类和静态工厂方法, 通过静态工厂方法获取对象 -->
<bean id="cal" class="java.util.Calendar" factory-method="getInstance" />

<!-- 3.指定工厂bean和实例工厂方法, 通过实例工厂方法获取对象 -->
<bean id="time" factory-bean="cal" factory-method="getTime" />
```

java配置

```java
@Configuration
public class SpringConfig{
    // 1.指定 Bean 类型, 通过构造器获取对象, 默认使用无参构造, 提供参数时使用有参构造
    @Bean
    public Date date(){
        return new Date();
    }

    // 2.指定工厂类和静态工厂方法, 通过静态工厂方法获取对象
    @Bean
    public Calendar cal(){
        return Calendar.getInstance();
    }

    // 3.指定工厂bean和实例工厂方法, 通过实例工厂方法获取对象
    @Bean
    public Time time(Calendar cal){
        return cal.getTime();
    }
}
```

执行类:

```java
ApplicationContext ac = new ClassPathXmlApplicationContext("applicationContext.xml");

// 采用无参构造获取的 bean, 实际调用的是 new Date()
Date d = ac.getBean("date", Date.class);

// 采用静态工厂方法获取 bean, 实际调用的是 Calendar.getInstance()
Calendar c = ac.getBean("cal", Calendar.class);

// 采用实例工厂方法获取 bean, 实际调用的是 cal1.getTime()
Time t = ac.getBean("time", Date.class);
```

### 2.2. Bean 的命名

在 Spring 容器中, 每个 Bean 都有自己的名字( 即 ID, 唯一的标识符 ), 该名字可以用 bean 元素的`id`或`name`属性指定.

在给 bean 定义好 ID 后, 可以通过 `<alias>` 元素指定别名, 可以通过别名访问到同一个 bean

```xml
<bean id="beanID" class="包名.类名" />
<alias name="beanID" alias="bean别名">
```

使用`@Bean`注解声明 bean 时, 默认以该方法名为 beanID, 也可以在`@Bean`注解的`value`属性中指定 beanID

```java
@Bean
public Bean methodNameBean(){...}

@Bean(value = "valueName")
public Bean valueNameBean(){...}
```

### 2.3. Bean 的配置

#### 1) 作用域

Spring 默认采用单例模式创建对象, 具体说明如下

- 若一个类对应一个 BeanID, 则无论调用几次`getBean()`, 都只有一个实例
- 若一个类拥有多个 BeanID, 则该类每个 BeanId 可以拥有一个实例
- 若一个类拥有一个 BeanID, 并通过 `alias` 获取别名, 共用一个实例

可以通过 bean 元素的 `scope` 属性指定作用域, 默认采用单例模式创建对象,

```xml
<bean id="bean名" class="包名.类名" scope="作用域" />
```

java 配置方式使用 `@Scope` 注解

```java
@Bean
@Scope("prototype")
public ScopeBean scopeBean(){...}
```

Spring 支持以下几种作用域, 通常使用`singleton`或`prototype`.

| 作用域         | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| **singleton**  | 单例模式, 无论执行 getBean 多少次, 都返回同一个对象          |
| **prototype**  | 原型模式, 每执行一次 getBean, 就创建一个 bean 对象           |
| request        | 在一次HTTP请求中, 一个bean定义对应一个实例, 仅限于WEB环境    |
| session        | 在一个HTTP Session中,一个bean定义对应一个实例, 仅限于WEB环境 |
| global Session | 在全局的 HTTP Session中, 一个bean对应一个实例, 仅在基于portlet的WEB应用中 |

#### 2) 延迟加载 (懒汉/饿汉)

默认情况下, 容器启动之后, 会将作用域为 singleton 的 bean 对象创建好 (饿汉模式)

延迟加载时, 容器启动之后, 不会自动创建 bean 对象, 调用 `getBean()` 方法才会创建 (懒汉模式)

```xml
<bean id="bean名" class="包名.类名" lazy-init="true" />
```

java 配置使用`@Lazy`注解启用延迟加载, `value=true`属性可以省略

```java
@Bean
@Lazy
public LazyMan lazyMan(){...}
```

#### 3) 生命周期回调

Spring 容器会管理对象的生命周期, 包括对象的创建, 初始化, 销毁等过程

##### a) 初始化/销毁方法

容器创建好 bean 的实例后, 会立即调用初始化方法, 每创建一个对象, 就会调用一次

容器关闭的时候, 要销毁对象, 在此之前还要先执行销毁方法, 只能作用于单例模式的 bean, 否则失效

```xml
<bean id="bean名" class="包名.类名" init-method="方法名" destroy-method="方法名" />
```

java 配置的 `@Bean` 注解同样提供了`initMethod`和`destroyMethod`, 以指定初始化/销毁方法

```java
@Bean(intiMethod="init", destroyMethod="destroy")
public LifeBean lifeBean(){...}
```

为了展示单例 bean 的销毁方法, 需要手动关闭应用上下文. 但 ApplicationContext 接口未提供关闭的方法, 由其子类 AbstractApplicationContext 提供

```java
AbstractApplicationContext aac = (AbstractApplicationContext) ac;
aac.close();
```

##### c) 默认方法

在顶级的 `<beans>` 元素中, 可以通过 `default-init-method` 属性, 或 `default-destroy-method` 属性对容器中所有 bean 指定初始化回调方法和销毁回调方法

```xml
<beans default-init-method="初始化方法"　default-destroy-method="销毁方法" >
　　< bean id="bean名" class="包名.类名"/>
</beans>
```

- `@PostConstruct`: 初始化方法, 需要 JavaEE 的依赖, 比如 tomcat
- `@PreDestroy:`: 销毁方法, 需要 JavaEE 的依赖, 比如 tomcat

# 三. Spring IOC

## 1. IOC 与 DI

### 1.1. IOC 

- Inversion of Control 控制反转
- IOC 是一种思想, 指程序中获取对象的方式发生反转, 由第三方框架/容器注入依赖的对象, 由外部注入依赖关系(*Dependency Injection*), 而不是传统的调用方自行维护要调用的组件对象. 从而降低组件间的耦合度
- Spring 采用 DI 方式实现 IOC, IOC 是 Spring 框架的基础和核心. IOC 是目标, DI 是手段.

![](Spring实战.assets/01_1.png)

### 1.2. DI

Dependency Injection  依赖注入

将成员 Bean 对象, 以构造/方法参数方式, 注入主对象, 建立依赖关系

容器的工作就是创建 bean 时注入这些依赖关系

DI 是实现 IOC 的主要技术途径

创建应用对象之间写作关系的行为通常称为**装配(wiring)**, 这也是依赖注入的本质.

## 2. xml 装配

### 2.1. 注入方式

#### 1) 构造器注入

在创建主 bean 的时候, 通过`<constructor-arg> `元素注入成员 bean, 再调用含参构造方法

```xml
<bean id="主BeanID" class="主Bean类名" >
    <!-- 可以通过构造参数下标(index)确定注入目标, 下标从0开始 -->
    <constructor-arg index="构造参数下标" ref="成员BeanID"/>
    <!-- 可以通过构造参数名(name)确定注入目标 -->
    <constructor-arg name="构造参数名" ref="成员BeanID"/>
</bean>
```

#### 2) Setter 注入

容器先通过无参构造创建了主 Bean, 再根据`<property>`元素的`name`属性调用 setter 方法注入成员 Bean.

```xml
<bean id="主BeanID" class="主bean类名">
　　<property name="bean属性" ref="成员beanID" />
</bean>
```

### 2.1. 注入类型

#### 1) 基本类型

可以通过字符串, 指定`<value>` 元素或者`value`属性属性或构造器参数的值. 容器将字符串从 String 类型转换为实际类型后注入

```xml
<bean id="beanID" class="包名.类型" >
    <!-- 使用 value 属性指定值 -->
    <property name="成员变量" value="基本类型值" />
    <!-- 使用 value 元素指定值 -->
    <property name="成员变量">
        <value>基本类型值</value>
    </property>
</bean>
```

#### 2) 引用类型

使用`ref`属性指定beanID, spring 会将指定的 bean 注入到当前构造的 bean 中. 

```xml
<bean id="主BeanID" class="主bean类名">
　　<property name="属性" ref="otherBeanID" />
</bean>
```

#### 3) 集合类型

通过`<list>`, `<set>`, `<map>`, `<props>`等元素, 可以定义或设置与 Java 中对应的 List, Set, Map, Properties 类型的属性值, 给 bean 注入一个集合.

```xml
<Bean id="主beanID" class="包名.类型" >
　　<property name="成员变量名">
　　　　<!-- 通过 list/set/map/props 等元素声明的集合 -->
　　</property>
</bean>
```

- 最外层 bean 元素是主 Bean 定义, 指定主 BeanID 和 类型
- 第二层`<property>`元素是主 Bean 对象的各个成员属性, 用`name`属性指定成员变量名
- 最内层是具体的集合具体数据, 其格式根据集合类型决定

具体示例如下

- 实体类

  ```java
  public class ValueBean {
  	private String name;
  	private int age;
  	private List<String> interest;
  	private Set<String> city;
  	private Map<String,Double> score;
  	private Properties db;
      // 省略基本方法
  }
  ```

- bean配置

  ```xml
  <bean id="vb" class="day02.D7_zValueBean" >
      <!-- List 和 Set 类型用 value 元素指定具体的值 -->
      <property name="interest">
          <list>
              <value>唱歌</value>
              <value>跳舞</value>
          </list>
      </property>
      <property name="city">
          <set>
              <value>北京</value>
              <value>南京</value>
          </set>
      </property>
      
      <!-- Map 使用 entry 元素的 key 属性指定键, 用 value 属性指定值 -->
      <property name="score">
          <map>
              <entry key="英语" value="90"/>
              <entry key="数学" value="85"/>
          </map>
      </property>
      
      <!-- Properties 使用 prop 元素的 key 属性指定键, 由标签的内容指定值 -->
      <property name="db">
          <props>
              <prop key="username">luojbin</prop>
              <prop key="password">lucky</prop>
          </props>
      </property>
  </bean>
  ```

#### 4) 空数据

可以为 bean 注入 null 或空字符串

```xml
<!-- 使用 null 元素注入 null 值 -->
<property name="nullVal">
    <null/>
</property>
<!-- 使用空值value注入空字符串 -->
<property name="emptyVal" value="" />
```

### 2.3. 命名空间

#### 1) 命名空间

xml 的文件头如下所示, 需要指定命名空间和对应的规范文件, `xsd`或`dtd`

```xml
<beans xmlns="默认命名空间"
       xmlns:前缀A="命名空间A"
       xmlns:前缀B="命名空间B"
       xsi:schemaLocation="命名空间A路径 具体文件A
                           命名空间B路径 具体文件B">
</beans>
```

spring 已经将需要的命名空间和schema 都放在了包内, 可以通过包内的`spring.handlers`和`spring.schemas`文件中找到具体的路径和映射文件

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
       http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util.xsd">
```

#### 2) c 命名空间

需要先在文件头声明使用 c 命名空间

`xmlns:p="http://www.springframework.org/schema/p"`

#### 3) p 命名空间

需要先在文件头声明使用 p 命名空间

`xmlns:p="http://www.springframework.org/schema/p"`

#### 4) util 命名空间

需要先在文件头声明使用 util 命名空间, 并指定 xsd

```xml
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       
       xsi:schemaLocation="
       http://www.springframework.org/schema/beans 	
       http://www.springframework.org/schema/beans/spring-beans.xsd
                           
       http://www.springframework.org/schema/util 
       http://www.springframework.org/schema/util/spring-util.xsd">
```

`<util:list>`, `<util:set>`, `<util:map>`, `<util:props>`等元素可以利用直接量构造集合bean, 并可以通过 setter 或构造器注入到其他bean 中.

```xml
<!-- 声明集合 bean 的格式与直接注入集合类似, 注意添加 util 命名空间 -->
<util:list id="interestBean">
    <value>吃酒</value>
    <value>喝烟</value>
</util:list>
<util:set id="cityBean">
    <value>北京</value>
    <value>南京</value>
    <value>西安</value>
</util:set>
<util:map id="scoreBean">
    <entry key="English" value="90" />
    <entry key="Math" value="85" />
</util:map>
<util:properties id="dbBean">
    <prop key="username">luojbin</prop>
    <prop key="password">lucky</prop>
</util:properties>
```

`<util:properties>` 元素可以通过读取`location`属性指定`properties`文件, 将内容存放到 `Properties` 对象里面, 以便在应用中读取配置使用.

```xml
<util:properties id="config" location="classpath:文件路径"/>
```

### 2.4. Spring 表达式

Spring 提供了 **Spring 表达式,** 类似于 EL 表达式, 可以方便地获取一个 Bean 对象中封装的数据.

- 实例域值: `#{bean.实例域}`
- 集合元素: `#{集合[下标]}`
- 映射的值: `#{映射['key']}`
- properties: `#{props.key}`

![image-20200220220913202](Spring实战.assets/image-20200220220913202.png)

### 2.6. 自动装配

Spring IOC 容器可以自动装配 bean 之间的关联关系, 减少 xml 注入配置工作

可以通过 `autowire` 属性启用自动装配, 默认为`no`, 建议采用 `byName` 方式以避免冲突

```xml
<bean id="beanID" class="包名.类名" autowire="自动装配方式" />
```

| 属性值        | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| `no`          | **默认值**, 不进行自动装配                                   |
| `autodetect`  | 自动选择 byType 或 constructor 方式, 有无参构造则采用 ByType |
| `byName`      | 通过成员变量的变量名, 查找 BeanID, 没找到则为null            |
| `byType`      | 通过成员变量的类型, 查找相应的 Bean, 没找到则为null, 多个则报错 |
| `constructor` | 通过构造器参数类型, 查找 Bean                                |

## 3. java 装配

java 配置通过`@Bean`注解修饰的方法来声明. 而对于bean的各种依赖的注入, 则如同普通 java 对象一般, 在方法体内对 bean 进行各种set即可.

与xml各种繁杂的标签相比, java 配置方式要简单得多, 而且可以通过代码执行各种操作, 更加灵活和强大. 

需要为bean 注入依赖时, 有以下两种办法

1. 在 bean2 的方法体内调用其他的`@Bean`方法, spring会拦截该方法的调用, 直接返回该方法注册到容器中的bean, 而不会重复执行方法体. 

   ```java
   @Bean
   public Bean bean2(){
       Bean bean2 = new Bean("bean-2");
       // 在方法体中调用 bean1() 方法, 作为 bean2 的依赖
       bean2.setContent(bean1());
       return bean2;
   }
   ```

2. 在 bean3 的参数列表中声明需要传入指定类型的 bean, spring 将会在容器内寻找合适的对象传入方法中.

   ```java
   // 在参数列表中传入 bean1, 作为 bean3 的依赖
   @Bean
   public Bean bean3(@Qualifier("bean1")Bean bean1){
       Bean bean3 = new Bean("bean-3");
       bean3.setContent(bean1);
       return bean3;
   }
   ```

可以注入基本类型或



## 4. 组件扫描与注解

### 3.1. 组件注解与扫描

Spring 启动以后, 可以扫描指定的包下所有类, 如果类含有特定的注解, 如 `@Component`, Spring 就会将该类作为一个 bean 纳入容器中管理. 组件扫描相当于在配置文件中批量定义 bean 元素

1. 在应用上下文配置文件中启用组件扫描

   ```xml
   <context:component-scan base-package="要扫描的包名" />
   ```

   java config 开启组件扫描, 在配置类上添加 `@ComponentScan`注解

   ```java
   @ComponentScan(basePackages = {"com.loyofo.spa.webapp"})
   @Configuration
   public class appConfig{}
   ```

2. 在组件类中添加组件注解, spring提供了多个等效注解, 效果基本一样, 仅为丰富语义而设置了多个.

   - `@Component`: 普通组件, 没有特殊含义
   - `@Controller`: web应用中的 controller 层组件
   - `@Service`: web 应用中的 service 层组件
   - `@Repository`: web 应用中的 dao 层组件
   - `@Configuration`: 配置类组件, 对应用上下文进行设置, 并注册手动声明的bean

组件扫描时, 默认以首字母小写的类名作为 beanID, 也可以在该类的注解后指定 BeanID.

```java
// 通过 value 属性指定 beanID, 也可以省略 value 或不指定
@Component(value = "beanID")
@Component("beanID")
@Component
```

### 3.2. 依赖注入

spring 可以自动为扫描得到 bean 注入依赖关系, 可以采用以下注解实现依赖注入:

#### 1) @Autowired + @Qualifier

`@Autowired` 可以在**构造器**, **setter**, **实例域**上使用, 默认以类型匹配 bean. 若类型匹配的 bean 不只一个, 则需要配合 `@Qualifier`注解指定 beanID

`@Qualifier` 写在方法参数或实例域前面, 声明要被注入的 BeanID, 只有一个类型匹配的 bean时可省略.

```java
// 在构造器上使用
@Autowired
public 类名(@Qualifier("beanID") 参数类型 参数名 ){...}

// 在 setter 方法上使用
@Autowired
public void set方法(@Qualifier("beanID") 参数类型 参数名 ){...}

// 在实例域上使用, 通过反射注入
@Autowired
@Qualifier("beanID")
private 属性类型 属性名;
```

#### 2) @Resource

`@Resource` 可以在 setter 方法或实例域上使用. 默认以实例域名称匹配 beanID, 若 beanID 与实例域名称不同, 则可以通过`name`属性指定 beanID

```java
// 在 setter 方法上使用
@Resource(name="注入的beanID")
public void set方法( 参数类型 参数名 ){...}

// 在实例域上使用, 通过反射注入
@Resource(name="注入的beanID")
private 属性类型 属性名;
```

#### 3) @Inject + @Named

Spring 3.0 开始增加的对**JSR-330**标准的支持, 使用前要添加 JSR-330 的 jar 包 `javax.inject-1.jar`

和 `@Autowired` + `@Qualifier` 用法一致, 但需要额外添加依赖, 仅作了解

#### 4) @Value

`@Value`, 给 bean 注入**基本类型**或 **spring 表达式**的值, 可以在 setter 或实例域上使用.

```java
// 在 setter 上使用
@Value("基本类型值")
public void set方法(参数类型 参数名 ){...}

@Value("#{Spring表达式}")
public void set方法(参数类型 参数名 ){...}

// 在实例域上使用, 通过反射注入
@Value("基本类型值")
private 属性类型 属性名;

@Value("#{Spring表达式}")
private 属性类型 属性名;
```

# 四. 高级装配

## profile

## 条件化的 bean

spring 4 提供了一个 `@Conditional` 注解, 可以用在 `@Bean` 方法上, 如果给定的条件计算结果为 `true`, 就会创建这个 bean, 否则忽略这个 bean

```java
@Bean
@Conditional(MyCondition.class)
public MyBean myBean() {
    ...
}
```

注意到 `@Conditional` 注解要求一个 `Condition` 类型的参数, `Condition` 是一个接口, 规定了 `matches()` 方法, 这个方法就是需要判断的条件. 

```java
public interface Condition {
	boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

`matches()` 方法很简单但功能强大, 它通过传入的 `ConditionContext` 和 `AnnotatedTypeMetadata` 对象, 获取需要的信息, 从而判断是否需要满足指定条件.

- `ConditionContext` 
  - 获取并查看应用上下文中的bean
  - 获取并查看环境变量
  - 通过 ResourceLoader 探查加载的资源
  - 通过 ClassLoader  探查 classpath 下是否存在某类
- `AnnotatedTypeMetadata` 
  - 获取 @Bean 方法上的其他注解

# 五. 切面 AOP



## 1. 概述

### 1.1. 为什么

切面, 即横截面

特点: 在不改变软件原有功能情况下为软件插入(扩展)横切面功能.

对于横向功能利用AOP可以大大简化软件的开发:

没有使用AOP:

![image-20200426103610330](Spring实战.assets/image-20200426103610330.png)

使用AOP后:

![image-20200426103620417](Spring实战.assets/image-20200426103620417.png)

![image-20200426103626422](Spring实战.assets/image-20200426103626422.png)

### 1.2. 切面技术类型

织入, 扩展, 代理

Spring  AOP 与 AspectJ

启用aop

JAVA方式，添加@EnableAspecJAutoProxy 注解
xml方式，添加`<aop:aspectj-autoproxy/>`

在类上添加`@aspect`注解后，该类的bean就会成为切面，但需要先用spring管理该bean，显式声明或组件扫描都可以

`@aspect`注解的类，不能再成为别的切面的目标

#### AOP 拦截器 过滤器 之间的区别

1. 过滤器: 拦截处理WEB请求!
2. Spring MVC 拦截器: 拦截处理Spring MVC的请求流程
3. AOP: 拦截Spring中各个组件之间方法请求.

## 2. Spring AOP

### 2.1. 通知类型

目标方法: 被AOP拦截的业务方法, 称为目标方法

切面方法在执行时机：就在目标方法之前, 之后执行.

- `@Before`: 切面方法在目标方法之前执行
- `@After`: 切面方法在目标方法之后执行
- `@AfterReturning`: 切面方法在目标方法正常结束之后执行
- `@AfterThrowing`: 切面方法在目标方法异常之后执行
- `@Around`: 环绕通知, 切面方法在目标方法的执行前后都有相关操作

![image-20200426103651584](Spring实战.assets/image-20200426103651584.png)

案例:

```java
@Component
@Aspect
public class DemoAspect {
    // 声明 test 方法将在 userService 的全部方法之前运行
    @Before("bean(userService)")
    public void test(){
        System.out.println("Hello World!");
    }
    @After("bean(userService)")
    public void test2(){
        System.out.println("After");
    }
    @AfterReturning("bean(userService)")
    public void test3(){
        System.out.println("AfterReturning");
    }
    @AfterThrowing("bean(userService)")
    public void test4(){
        System.out.println("AfterThrowing");
    }
}
```

环绕通知, 可以在业务方法前后调用

- 必须有返回值值 Object
- 必须有参数 ProceedingJoinPoint
- 必须抛出异常
- 需要在方法中调用 jp.proceed()
- 返回业务方法的返回值

![image-20200426103705382](Spring实战.assets/image-20200426103705382.png)

```java
@Component
@Aspect
public class Demo1Aspect {
    @Around("bean(userService)")
    public Object test5(ProceedingJoinPoint jp) throws Throwable{
        Object val = jp.proceed();
        System.out.println(" 业务结果 :"+val);
        throw new UserNotFoundException(" 就是不让登录 ");
    }
}
```

### 2.2. 切入点

pointcut声明
注解方式
返回值为void的方法+@pointcuts注解的表达式

用于定位APO的切入位置: 指定切入到具体的类或方法, 一致统一的命名规则将有助于编写有效的切入点表达式

切点表达式

> execution(modifiers-pattern? ret-type-pattern declaring-type-pattern?name-pattern(param-pattern) throws-pattern?)

其中
modifiers-pattern?  方法修饰符，可选
ret-type-pattern 返回值类型，必须
declaring-type-pattern? 包名与类名，可选
name-pattern 方法名，必须
(param-pattern) 参数列表，必须
throws-pattern?) 异常列表，可选

规则

| 切入点类型                   | 描述 | 示例 |
| ---------------------------- | ---- | ---- |
| bean组件切入点               |      |      |
| 类切入点                     |      |      |
| 方法切入点 (execution: 执行) |      |      |
|                              |      |      |
|                              |      |      |
|                              |      |      |

- bean组件切入点
  - bean(userService)
  - bean(userService) || bean(noteService) || bean(notebookService)
  - bean(*Service)
- 类切入点
  - within(类名)
  - within(类名) || within(类名)
  - within(cn.tedu.note.service.impl.UserServiceImpl)
  - within(cn.tedu.note. .impl. ServiceImpl)
- 方法切入点 (execution: 执行)
  - execution(修饰词 类名.方法名(参数类型))
  - execution(* cn.tedu.note.service.UserService.login(..))
  - execution(* cn.tedu.note. . Service.list*(..))

### 2.3. 示例

开发一个AOP案例:

1. 导入Aspect J 包, Spring AOP 底层利用了AspectJ 实现的(有误, 实现与 AspectJ 无关, 只是借用了注解)

2. 创建切面组件

   ```java
   @Component
   @Aspect
   public class DemoAspect {
       // 声明 test 方法将在 userService 的全部方法之前运行
       @Before("bean(userService)")
       public void test(){
           System.out.println("Hello World!");
       }
   }
   ```

3. 配置Spring AOP: 

   xml 方式

   ```xml
   <!-- 配置组件扫描 -->
   <context:component-scan base-package="cn.tedu.note.aop"/>
   <!-- 使 @Aspect 注解生效 -->
   <aop:aspectj-autoproxy/> 
   ```

   java config 方式

   ```java
   @EnableAspectJAutoProxy
   ```

## 3. 底层原理

spring aop基于代理，在运行时织入

before不能中断方法调用，除非抛出异常

spring aop是基于代理的，只有外部调用public方法会触发拦截，而目标类自身方法调用，不会经过代理类，无法触发切面操作

代理模式: 不改变原有功能为软件扩展新功能.

AOP封装了动态代理功能, 提供了更加简便的使用方式!

经典面试问题:

AOP 的底层技术是什么 ?

答案 : 使用了 动态代理 技术 .

关键点:

![image-20200426103726445](Spring实战.assets/image-20200426103726445.png)

![image-20200426103731782](Spring实战.assets/image-20200426103731782.png)



1. Spring AOP 利用了 AspectJ AOP实现的!

2. AspectJ AOP 的底层用了动态代理

3. 动态代理有两种

  目标方法有接口时候自动选用 JDK 动态代理

  目标方法没有接口时候选择 CGLib 动态代理

## 4. 注意事项

### 4.1. 代理对象的问题

Spring AOP 通过代理来实现功能扩展, 只支持方法级的织入. 

由于是代理对象, 只有从外部调用被切对象时, 才会触发切面的代理对象, 此时切面才能工作

如果在被切对象内部的方法调用, 由于未经过代理对象, 则不会触发切面的操作.

解决方法: 再次从外部调用被切对象

### 4.2. 作用域

由于 spring 应用中可能存在多个不同的应用上下文, 而如果一个应用上下文中存在被切对象, 则必须在该应用上下文启用 aop 切面功能.

spring aop 使用 aspectJ 的切点表达式语言

aop与事务

rootConfig 扫描controller 以外的所有, 包括切面和service, config

webConfig 扫描controller

|                          | controller切面 | service切面 | 事务切面 |
| ------------------------ | -------------- | ----------- | -------- |
| rootConfig+ EnableAspect | 无             | 有          | 有       |
| webConfig+ EnableAspect  | 有             | 无          | 无       |
| 都加EnableAspect         | 有             | 有          | 有       |
|                          |                |             |          |

rootConfig, 管理controller 切面, service 切面, 事务切面, service bean

rootConfig+EnableAspect 可以启用service 切面切入 service bean, 事务切面切入 service bean

webConfig, 只管理 controller bean

webConfig+enableAspect 可以启动 controller 切面切入controlelr bean



总结, 关键在于被切组件, 而切面则可以在别的上下文中声明

Note: @EnableAspectJAutoProxy applies to its local application context only, allowing for selective proxying of beans at different levels. 

Please redeclare @EnableAspectJAutoProxy in each individual context, e.g. the common root web application context and any separate DispatcherServlet application contexts, if you need to apply its behavior at multiple levels.

### 4.3. 切面顺序

`@Order` 可以指定切面执行的顺序, 序号越小越先执行

# 三. AOP 面向切面编程

## 1. 概述

### 1.1. 什么是 AOP

Aspect Oriented Programming, 面向切面编程
切面(儿): 事务的横截面

特点: 在不改变软件原有功能情况下, 为软件插入(扩展)横切面功能
spring利用反射机制, 在动态调用业务方法时执行切面组件.

### 1.2. 通知

目标方法: 被 AOP 拦截的业务方法, 称为目标方法
切面方法的执行时机: 在目标方法执行的之前或之后执行, 

## 2. Aspect 的注解

### 2.1. Aspect 注解

在类的前面添加 @Aspect 注解, 声明该类用作切面组件.
若该切面组件要由 Spring 框架统一处理, 还要加上 @Component

> @Component
> **@Aspect**
> public class DemoAspect{...}

### 2.2. Before 注解

在方法前添加 Before 注解, 指定要处理的 BeanID, 
该方法将在 指定的 Bean 的全部方法之前运行

> **@Before**("bean(要处理的BeanID)")
> public void test(){...}


### 2.3. After 注解

在方法前添加 After 注解, 指定要处理的 BeanID, 
该方法将在 指定的 Bean 的全部方法之后运行

> **@After**("bean(要处理的BeanID)")
> public void test2(){...}

### 2.4. AfterReturning 注解

调用的业务方法正常执行后, 执行该方法

### 2.5. AfterThrowing 注解

调用的业务方法抛出异常后, 执行该方法

### 2.6. Around 注解

环绕通知, 可以在业务方法前面进行预处理, 也可以在业务方法后面进行后处理, 甚至可以篡改业务方法的正常返回结果

1. 必须有返回值 Object

2. 必须有参数 ProceedingJoinPoint, 实际上是通过反射获得的业务方法

3. 必须抛出异常

4. 需要在方法中调用 jp.proceed(), 即执行业务方法

5. 返回与业务方法同类型的返回值

   	@Around("bean(userService)")
   	public Object test5(ProceedingJoinPoint jp) throws Throwable{
   		System.out.println("Hello @Around");
   		//业务方法前处理
   		Object val = jp.proceed();
   		//业务方法后处理
   		System.out.println(val);
   		//还能干预业务方法的返回结果
   		//return new User();
   		
   		//异常也是方法的返回值, 可以只抛异常, 不写return 
   		throw new UserNotFoundException("就是不让登录");
   	}

#### 2) JoinPoint 

JoinPoint 对象可以获取目标业务方法的详细信息, 方法签名, 调用参数等

> jp.getSin

## 3. 编程步骤

1. 导入依赖的包 aspectj :　aspectjweaver

   	<dependency>
   		<groupId>aspectj</groupId>
   		<artifactId>aspectjweaver</artifactId>
   		<version>1.5.3</version>
   	</dependency>

2. 编写切面组件

   	@Component
   	@Aspect
   	public class DemoAspect {
   		//声明 test 方法将在 userService 的全部方法之前运行
   		@Before("bean(userService)")
   		public void test(){
   			System.out.println("Hello World");
   		}
   	}

3. 配置到spring

   	<!-- 配置切面组件扫描 -->
   	<context:component-scan base-package="cn.tedu.note.aop" />
   	<!-- 使 aspect 的 before 注解生效 -->
   	<aop:aspectj-autoproxy />

4. spring利用反射机制, 在动态调用业务方法时执行切面组件的方法.


## 4. 切入点表达式

AOP 注解后面使用切入点表达式, 通过 BeanID 指定要切入的 Bean 组件类

### 4.1. Bean组件切入点

通过 BeanID 指定要切入的 Bean 组件类

#### 1) 单点切入

> bean( BeanID )

	@Before("bean(userService)")
	@Before("bean(noteService)")

#### 2) 多点切入

可以使用双竖线表示多个切入点

> bean( BeanID_1 ) || bean( BeanID_2 )

	@Before(" bean(userService) || bean(noteService) ")

#### 3) 通用切入

可以使用通配符 ＊, 切入满足匹配规则的 Bean 组件类

> bean(*匹配字符串)

	@Before(" bean(*Service) ")

### 4.2. 类切入点

通过完整类名, 指定 AOP 的切入点

#### 1) 单类切入

> within(类名)

#### 2) 多类切入

> within(类名1)||within(类名2)

#### 3) 通用切入

一个星号只能匹配一级包名

> within(包名.*.*类名)

### 4.3. 方法切入点

参数类型可以用两个点".."表示
一致统一的类和方法的命名规则, 将有助于编写有效的切入点表达式

> execution(修饰词 类名.方法名(参数类型))


- execution(* cn.tedu.note.service.UserService.login(..)) 
- execution(* cn.tedu.note.*.*Service.get*(..)) 

# 四. 代理

## 1. 概述

代理模式: 在不改变原有功能的前提下, 为其扩展功能
静态代理: 
动态代理: 

客户端 -> 对外接口 -> 代理 -> 目标功能

代理, 具有对外接口中的所有方法, 但没有具体实现, 具体实现仍由目标功能提供

将动态代理具体实现过程封装, 就是 AOP, 提供对外接口功能和更加简便的使用方式

经典面试问题:
AOP 的底层技术是什么? - 使用了动态代理技术, 是反射的一部分.

关键点:

1. Spring AOP 利用了 AspectJ AOP 实现的.
2. AspectJ AOP 的底层用了动态代理
3. 动态代理有两种

   1. 当方法中有时, 使用 JDK 动态代理
   2. 当方法中什么时, 使用CGLib

Spring AOP -> AspectJ AOP -> 动态代理 -> JDK 动态代理/ CGLib

### 2. AOP 拦截器 过滤器

1. 过滤器: 拦截处理 WEB 请求
2. Spring MVC 拦截器: 拦截处理 Spring MVC 的请求流程
3. AOP: 拦截业务层

# 五. 声明式事务处理

## 1. 编程式事务与声明式事务

### 1.1. 编程式事务

通过编写程序执行语句, 控制事务的提交与回退, 保护事务边界内的数据库操作.
缺点: 

### 1.2. 声明式事务

通过 AOP, 使用注解(通知)的声明, 控制事务的提交与回退
@Before 开启事务边界
@Transactional 声明业务方法
@AfterReturning 关闭事务边界,提交事务
@AfterThrowing 回退事务, 只处理 RuntimeException, 其他异常仍会继续提交
@After 释放资源

除了具体的业务方法, 其他事务控制操作都已经封装好, 我们只需要在数据库操作方法前加上 @Transaction, 再添加配置事务管理器 Bean 组件.

# 六. 事务隔离属性

一共有四种 
一般采用默认的 @Transactional(isolation=Isolation.READ_COMMITTED)级别, 是并发性能和安全性能这种的选择, 是

建议所有业务方法都使用 @Transactional, 如果是只读方法, 建议增加 readOnly=true

面试题目: Spring 声明式事务是如何实现的?
	- 声明式事务利用 **AOP 技术**实现, 而 AOP 技术采用了**动态代理**, 动态代理的底层利用**反射机制**实现

# █ Spring Web

# 一. 分层理论

## 1. 目的

为了系统好维护，系统的设计应该要做到“高内聚，低耦合”。

**高内聚**：指的是类的职责要单一，这样，一个类就可以会拆分成多个类（比如 AccountService 拆分成了AccountService 和 AccountDAO）,这样，就形成了分层的概念。

**低耦合**：指的是类与类之间不要直接依赖。（ AccountService 要调用 AccountDAO，应该使用依赖注入）。

## 2. 三层结构

1. 表示层：也叫视图层, 是数据展现和操作的界面，以及请求分发。

2. 业务层：封装了业务逻辑。

3. 持久层: 封装了数据访问逻辑。

  表示层调用业务层，业务层调用持久层。

  上一层通过接口来调用下一层提供的服务

  下一层的具体实现发生了改变，不影响上一层

![image-20200312225828350](Spring实战.assets/image-20200312225828350.png)

![image-20200312225837969](Spring实战.assets/image-20200312225837969.png)



## 3. MVC模式

MVC 属于表示层的一种软件架构思想.

### 3.1. M-MODEL

模型的职责是负责业务逻辑, 包括两层:

业务数据, 以及业务处理逻辑.

比如实体类, DAO, Service 都属于模型层.

### 3.2. V-View

视图的职责是负责显示界面和用户交互, 收集用户信息

视图层主要是不包含业务逻辑和控制逻辑的 JSP

### 3.3. C-Controller

控制器是模型层和视图层之间的桥梁, 用于控制流程

比如在 Servlet 项目中的单一控制器 ActionServlet

## 4. Spring MVC

### 3.1. 什么是 Spring MVC?

用来简化基于MVC架构的web应用程序开发的框架。

注：SpringMVC是spring中的一个模块。

### 3.2. Spring MVC 五大组件

#### 1) DispatcherServlet 前端控制器

`org.springframework.web.servlet.DispatcherServlet`

将浏览器发来的请求进行分发, 控制调用各个处理器或组件来响应不同的请求

在 web.xml 项目配置文件中配置

#### 2) HandlerMapping 映射处理器

`org.springframework.web.servlet.handler.SimpleUrlHandlerMapping`

前端控制器分发请求的依据, 规定了哪些请求应该由哪个处理器来处理

在 spring.xml 配置文件中配置, 使用 `@RequestMapping` 注解则不需在配置文件中配置.

#### 3) Controller 处理器

`org.springframework.web.servlet.mvc.Controller`

处理具体的业务逻辑, 将处理结果封装成 `ModelAndView` 对象传回

新编写的 controller 应实现这一接口, 使用 `@Controller` 注解的话, 则不需要实现接口.

#### 4) ModelAndView 模型与视图名

`org.springframework.web.servlet.ModelAndView`

用于封装处理器处理后的数据, 并包含用于定位下一个视图的视图名

#### 5) ViewResolver 视图解析器

`org.springframework.web.servlet.view.InternalResourceViewResolver`

处理器返回视图名字符串, 视图解析器根据处理器返回的视图名, 解析出真正要使用的视图对象的路径

在 spring.xml 配置文件中配置

### 3.3. Spring MVC 工作原理

![image-20200312225918255](Spring实战.assets/image-20200312225918255.png)

1. 服务器收到请求后, 将请求发送给 `DispatcherServlet`, 准备进行分发;
2. `DispatcherServlet` 获取 `HandlerMapping` 的配置;
3. 根据配置信息, 调用对应的 `Controller` 来处理请求;
4. `Controller` 将处理结果封装成 `ModelAndView`，返回给 `DispatcherServlet`;
5. `DispatcherServlet` 依据 `ViewResolver`, 解析出要使用的视图对象;
6. 调用对应的视图对象（比如jsp）来生成相应的页面;
7. 服务器将生成的响应页面发送给浏览器.

# 二. Spring MVC 的应用

## 1. 基于 xml 的 Spring MVC 应用

1. 导包 `spring-webmvc`
2. 添加配置文件 `spring-mvc.xml` 。
3. 在项目的 web.xml 配置 `DispatcherServlet`。
4. 写 Controller, 必须实现 `org.springframework.web.servlet.mvc.Controller` 接口。
5. 写 jsp。
6. 在配置文件 spring-mvc.xml 当中，添加 HandlerMapping, ViewResolver 的配置。

1.2. 案例. Hello请求

![image-20200312225949416](Spring实战.assets/image-20200312225949416.png)

1) web.xml

```xml
<servlet>
    <description></description>
    <display-name>DispatcherServlet</display-name>
    <servlet-name>DispatcherServlet</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <init-param>
        <description></description>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:*.xml</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>DispatcherServlet</servlet-name>
    <url-pattern>*.do</url-pattern>
</servlet-mapping>
```

2) controller

```java
public class HelloController implements Controller{
    @Override
    public ModelAndView handleRequest(HttpServletRequest req,
                                      HttpServletResponse res) throws Exception {
        System.out.println("handleRequest()");
        /*
         * ModelAndView 有两个常用构造器 :
         * ModelAndView(String viewName)
         * ModelAndView(String viewName, Map data)
         */
        return new ModelAndView("hello");
        return new ModelAndView("hello");
    }
}
```

3) hello.jsp

```html
<h1>hello spring mvc</h1>
```

4) spring-mvc.xml

```xml
<!-- 配置 HandlerMapping -->
<bean class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
    <property name="mappings">
        <props>
            <!-- 指定访问名和与之对应的 controllerBean -->
            <prop key="/hello.do">helloController</prop>
        </props>
    </property>
</bean>
<!-- 配置 Controller -->
<bean id="helloController" class="controller.HelloController"/>
<!-- 配置视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <!-- 指定视图名对应的前缀 -->
    <property name="prefix" value="/WEB-INF/"/>
    <!-- 指定视图名对应的后缀 -->
    <property name="suffix" value=".jsp"/>
</bean>
```

## 2. 基于注解的 Spring MVC 应用

2.1. 编程步骤
1. 导包 spring-webmvc
2. 添加配置文件。
3. 配置 DispatcherServlet。
4. 写 Controller。
5. 写 jsp。
6. 在配置文件当中，添加 ViewResolver 的配置, 组件扫描, 添加 mvc 注解扫描。

![image-20200312230157583](Spring实战.assets/image-20200312230157583.png)

2.2. 使用注解编辑 Controller 处理器类

1. 不需要实现 Controller 接口

2. 可以添加多个方法, 处理多种请求

3. 返回值可以是 ModelAndView, 也可以是 String, 返回的 String 就是视图名

4. 使用 `@Controller` 注解, 则可使用组件扫描添加控制器

5. 方法名不作要求, 可以在方法前或类前添加 `@RequestMapping` 注解,

   相当于在xml 中配置 HandlerMapping

   类前的注解相当于模块名, 方法前的注解作为具体页面的访问路径

2.3. 案例: 注解 hello.do

1) web.xml

```shell
配置 dispatcherServlet, 同上 , 略
```

2) spring.xml

```xml
<!-- 配置组件扫描 , 启用 @Controller 注解 , 不需在 xml 中配置 bean -->
<context:component-scan base-package="controller" />
<!-- 配置 MVC 注解扫描 , 启用 @RequestMapping, 不需在 xml 中配置 HandlerMapping -->
<mvc:annotation-driven />
<!-- 配置视图解析器 -->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

3) HelloController

```java
@Controller
public class HelloController2{
    @RequestMapping("/hello.do")
    public String hello2(){
        System.out.println(" 这是我重新写的一个 hello.do");
        return "hello";
    }
}
```

# 三. 接收与发送数据

## 1. 接收请求参数

### 1.1. 传入请求对象 Request

Spring 可以直接将 `HttpServletRequest` 对象作为参数注入到处理器方法中,

优点: 直接, 与传统方式操作类似

缺点: 从 request 对象获取参数后, 需要自行进行类型转换

```java
@RequestMapping("/login.do")
public String login(HttpServletRequest request) {
    String adminCode = request.getParameter("adminCode");
    System.out.println("adminCode:" + adminCode);
    return "index";
}
```

### 1.2. 传入请求参数 RequestParam

`DispatcherServlet` 调用处理器的方法之前, 会根据 Java 反射机制, 分析方法的结构.

如果方法中定义了参数, 且与 requuest 对象传回的参数同名, Spring 会自动调用 request 对象的方法, 先获得其中的参数值, 然后赋给方法对应的形参。

如果方法参数与请求属性的名称不一致，要使用 `@RequestParam` 注解, 以映射不一样的名称。

优点: 参数类型自动转换

缺点: 可能会出现类型转换异常

如果参数太多, 则方法变得特别复杂, 实际中很少使用

```java
@RequestMapping("/login2.do")
public String login2(String adminCode, @RequestParam("pwd") String password) {
    System.out.println("login2()");
    System.out.println("adminCode:" + adminCode + " pwd:" + password);
    return "index";
}
```

### 1.3. 封装成对象后传入

定义一个实体类, 用于封装参数值, Spring 自动将请求参数封装成对象, 再将该类对象作为方法参数传入.

**名称一致** : 该类中的成员属性, 要与网页的请求参数名一致

**自动转换** : 容器会自动将请求参数的字符串值转换为对应的类型, 可能会转换异常

**读写方法** : 该类要提供 get/set 方法

```java
@RequestMapping("/login3.do")
public String login3(AdminParam ap) {
    System.out.println("login3()");
    String adminCode = ap.getAdminCode();
    System.out.println("adminCode:" + adminCode);
    return "index";
}
```

## 2. 向页面传值

当 controller 组件处理请求后, 需要向 JSP 传递数据时, 可以采用以下方式

在满足需求的情况下, 应尽量选择生命周期短的, 以节约服务器资源

### 2.1. 使用 `HttpServletRequest`

默认情况下, `DispatcherServle` 会使用**转发**进行页面的跳转, 地址不变, 并将 `request` 对象传递给下一个servlet, 下一个 servlet (如 jsp )可以从中获取数据

```java
@RequestMapping("/login4.do")
public String login4(AdminParam ap, HttpServletRequest req){
    System.out.println(" 访问 login4, 参数绑定 request 后写出 ");
    String adminCode = ap.getAdminCode();
    req.setAttribute("admin", adminCode);
    return "index";
}
```

### 2.2. 使用 `ModelAndView` 对象

在 Controller 处理方法完成后, 将模型数据封装到一个 Map 里, 再将模型数据 Map 与视图名一同打包, 返回一个 `ModelAndView` 对象

```java
@RequestMapping("/login5.do")
public ModelAndView login5(AdminParam ap) {
    System.out.println("login5()");
    String adminCode = ap.getAdminCode();
    // step1. 将数据添加到 Map 对象里面
    Map<String, Object> data = new HashMap<String, Object>();
    // 相当于执行了 request.setAttribute
    data.put("admin", adminCode);
    // step2. 将 Map 对象添加到 ModelAndView
    ModelAndView mav = new ModelAndView("index", data);
    return mav;
}
```

### 2.3. 使用 `ModelMap` 对象

在 Controller 处理方法中声明一个 `ModelMap` 参数来充当数据载体, 容器会自动将该对象传入, 可以给该 ModelMap 对象添加键值对数据, 从而传递给下一个页面

```java
@RequestMapping("/login6.do")
public String login6(AdminParam ap, ModelMap mm) {
    System.out.println("login6()");
    String adminCode = ap.getAdminCode();
    // 相当于执行了 request.setAttribute
    mm.addAttribute("adminCode", adminCode);
    return "index";
}
```

### 2.4. 使用 `HttpSession` 对象

session 与 request 类似, 但存活时间更长, 可以在 session 对象上绑定数据, 向页面传值

```java
@RequestMapping("/login7.do")
public String login7(AdminParam ap, HttpSession session){
    System.out.println("login7, 使用 Session 传值 ");
    String adminCode = ap.getAdminCode();
    session.setAttribute("adminCode", adminCode);
    return "index";
}
```

### 2.5. 使用 `@ModelAttribute` 注解

1) 在传入参数前使用

如果在传入参数面前使用 `@ModelAttribute` 注解, 则该参数会被传递到下一个页面

但要求传入参数的页面, 与传出参数的页面, 两个页面对该参数的命名一致, 此条件不一定满足, 因而较少用

```java
@RequestMapping("/login11.do")
public String login11(
    @ModelAttribute("adminCode") String admin,
    @RequestParam("pwd") String password){
    System.out.println(" 访问 login6");
    System.out.println(admin +":"+ password);
    return "index";
}
```

2) 在 controller 的 bean 属性方法前使用

也可以在 controller 类中, 将bean 属性作为 modelAttribute 传递到下一个页面

实际上 ssm 框架通常要求 controller 是单例, 必须是无状态的, 因此不常使用其属性来传递参数.

```java
private String testMA= "test ModelAttribute";
@ModelAttribute("testMA")
public String getTestMA() {
    return testMA;
}
```

### 2.6. 案例: BMI 指数计算

1) 要求:

计算一个人的 BMI 指数

BMI = 体重(kg) / 身高(m) / 身高(m)

BMI < 19, 过轻; BMI > 24 过重; 否则正常

2) 输入页面.jsp

```html
<body style="font-size:30px;">
    <form action="bmi.do" method="post">
        身高 (m):<input type="text" name="height" /><br />
        体重 (kg)):<input type="text" name="weight" /><br />
        <input type="submit" value=" 计算 "/>
    </form>
</body>
```

3) Controller 代码

```java
@Controller
public class BmiController {
    @RequestMapping("tobmi.do")
    public String tobmi(){
        System.out.println(" 访问 tobmi.do");
        return "bmi";
    }
    @RequestMapping("bmi.do")
    public String calbmi(BmiParam bp, ModelMap mm){
        double bmi = bp.getWeight()/bp.getHeight()/bp.getHeight();
        String status=" 正常 ";
        if(bmi<19){
            status=" 过轻 ";
        }
        if(bmi>24){
            status=" 过重 ";
        }
        mm.addAttribute("status", status);
        mm.addAttribute("bmi",bmi);
        return "view";
    }
}
```

4) 响应页面.jsp

```jsp
<body>
    <h2>result</h2>
    <p> 你的 BMI 指数的是 ,${bmi} , 属于 :${status }</p>
</body>
```

## 3. 重定向视图

### 3.1. 使用 String 重定向

返回值是字符串时, 视图解析器默认以**转发**方式访问指定视图, 若想以**重定向**的方式跳转, 则要使用 `redirect: `前缀来标记:

```java
return "redirect:重定向地址"
```

### 3.2. 使用 ModelAndView 重定向

若处理器方法的返回值是 `ModelAndView`, 需要先创建一个 `RedirectView `类的对象, 再根据这个对象创建新的 `ModelAndView`, 从而实现重定向. 这种方法比较麻烦, 实际中较少使用.

```java
RedirectView rv = new RedirectView('重定向地址');
return new ModelAndView(rv);
```

## 4. 中文乱码问题

### 4.1. 乱码原因

在表单提交时, 对于中文等非 **ASCII** 字符, 浏览器会根据当前页面的编码方式, 对表单数据进行编码

而服务器默认采用 **ISO-8859-1** 进行解码, 若不进行编码方式声明, 则会出现乱码

### 4.2. 解决方式

此前解决中文乱码时, 在 servlet 中添加了 request 的编码方式声明. 而在 Spring 框架中, request 已由容器采用默认编码方式声明, 用户不易自行修改

SpringMVC 提供了一个过滤器`CharacterEncodingFilter`, 只需要配置该过滤器即可解决表单中文乱码。

1. 表单数据以 post 方式提交
2. 在 web.xml 中配置 `CharacterEncodingFilter` 过滤器
3. 过滤器的编码方式, 要与表单页面编码保持一致

```xml
<filter>
    <filter-name>encodingFilter</filter-name>
    <filter-class>
        org.springframework.web.filter.CharacterEncodingFilter
    </filter-class>
    <!-- 配置采用的编码方式 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
</filter>
<!-- 配置要过滤的页面请求 -->
<filter-mapping>
    <filter-name>encodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 5. 异常处理

### 1.1. Spring 处理异常

默认情况下, Spring 框架会将异常输出到浏览器, 直接抛给最终用户

#### 1) 配置简单异常处理器

`org.springframework.web.servlet.handler.SimpleMappingExceptionResolver`

Spring 捕获到了指定的异常, 就直接跳转到新页面, 不能执行复杂的操作, 只适合简单的异常处理

在 Spring-mvc.xml文件中配置简单异常处理器

```xml
<bean class="包名.SimpleMappingExceptionResolver">
    <property name="异常映射名">
        <props>
            <prop key="异常类型"> 视图名 </prop>
        </props>
    </property>
</bean>
```

#### 2) 自定义异常处理方法

定义一个方法, 添加 `@ExceptionHandler` 注解

在方法内定义异常处理逻辑

该方法只处理所在类抛出的异常, 不处理其他类的异常

```java
@ExceptionHandler
// 这是一个异常处理方法 , 参数 e 是处理器方法所抛出的异常
public String HandleEx(Exception e, HttpServletRequest request){
    System.out.println(" 这是一个异常处理方法 ");
    if(e instanceof NumberFormatException){
        e.printStackTrace();
        System.out.println("哟 , 数字格式化异常啊 ");
        request.setAttribute("msg", " 亲 , 请输入正确的数字 ");
        return "error3";
    }
    if (e instanceof StringIndexOutOfBoundsException){
        e.printStackTrace();
        System.out.println("哟 , 下标越界异常啊 ");
        request.setAttribute("msg", " 亲 , 你的下标越界了哟 ");
        return "error3";
    }
    return "error";
}
```

#### 3) 统一异常处理

在 controller 中的`@ExceptionHandler` 方法, 只能处理该 controller 中抛出的异常, 为了统一处理所有 controller 抛出的同类异常, 可以使用 `@ControllerAdvice`

## 6. 拦截器

### 6.1. 什么是拦截器

`DispatcherServlet` 收到请求之后，如果有拦截器，会先调用拦截器，然后再调用 `controller`。

Spring 的 `HandlerMapping` 处理器支持拦截器应用.

过滤器是 servlet 规范, 拦截器是 Spring 框架的组件.

![image-20200312231055874](Spring实战.assets/image-20200312231055874.png)

### 6.2. 拦截器接口

拦截器必须实现 `HandlerInterceptor` 接口或继承 `HandlerInterceptorAdaptor` 类

#### 1) preHandle()

前处理, 执行处理器之前被调用

`DispatcherServlet` 收到请求后, 如果有拦截器, 会先调用拦截器, 满足条件后再放行, 继续调用 controller 处理器. 否则流程中止, 不再执行后续操作.

`DispatcherServlet`, 拦截器, `controller` 会共享 `request` 和 `response`

返回 true 表示会继续调用其他拦截器和处理器

返回 false 表示中断流程, 不再执行后续拦截器和处理器

```java
public boolean preHandle(HttpServletRequest request, HttpServletResponse response,
                         Object handler)
    throws Exception {
    System.out.println(" 前处理方法 ");
    return true;
}
```

#### 2) postHandle()

处理器执行后, 视图处理前调用

`controller` 处理器已执行完, 正准备将 `ModelAndView` 返回给 `DispatcherServlet` 时调用

此时可以通过 `ModelAndView` 对象, 对模型数据或视图进行处理

```java
public void postHandle(HttpServletRequest request, HttpServletResponse response, 
                       Object handler, ModelAndView modelAndView) throws Exception {
    System.out.println("后处理方法");
}
```

#### 3) afterCompletion()

整个请求处理完毕后调用, 如性能监控等. 只有在 `preHandle` 方法返回 true 时才会执行

```java
public void afterCompletion(HttpServletRequest request, HttpServletResponse response,
                            Object handler, Exception ex) throws Exception {
    System.out.println(" 最后的方法 ");
}
```

### 6.3. 拦截器的使用

1. 写一个java类, 实现 HandlerInterceptor 接口
2. 在接口方法里, 实现拦截处理逻辑
3. 配置拦截器

![image-20200312231143963](Spring实战.assets/image-20200312231143963.png)

# 二. 视图技术

# 三. MVC 进阶

# 四. Spring Web Flow

# 五. Spring Security

Spring Security主要做两件事，一件是认证，一件是授权

## 1. 启用 security

创建一个 `WebSecurityConfigurerAdapter` 的实现类, 这是 spring security 的主要配置起点, 提供了三个 `configure` 方法, 根据参数的不同, 完成以下三个方面的配置

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    /** 配置如何通过拦截器保护请求 */
    @Override
    protected void configure(HttpSecurity http) throws Exception {}

    /** 配置 user-detail 服务 */
    @Override
    public void configure(AuthenticationManagerBuilder auth) throws Exception {}
    
    /** 配置Spring Security 的 filter 链 */
    @Override
    public void configure(WebSecurity webSecurity) throws Exception {}
}
```

在spring security 的配置类中, 我们还使用了 `@EnableWebSecurity` 这一开关注解, 

如果使用 springMvc, 并且使用表单提交数据,  `@EnableWebMvcSecurity` 在 `@EnableWebSecurity` 的基础上自动为表单增加了一个 CSRF 的 token. 

默认情况下,  spring security 对所有请求都要求经过认证(登录), 但是 spring security 没有默认的用户配置, 则没有人能通过认证, 因此, 默认配置的 spring security 会拦截所有的请求.

为了使 spring security 正常投入使用, 我们需要做以下工作

- 配置用户存储, 以便正常登录系统通过认证
- 指定哪些请求需要认证, 哪些请求需要什么权限
- 提供一个登录入口, 让用户进行登录

如果需要对 spring security 进行一些自定义的配置, 可以重载其中的三个 `configure` 方法

## 1. 身份验证管理

### 使用内存定义的用户信息

仅限开发使用

```java
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
        // 指定用户名和密码, 并用 authorities 授权
        .withUser("test").password("test").authorities("TEST", "ROLE_USER").and()
        // roles() 方法是 authorities() 的语法糖, 自动添加 `ROLE_` 的前缀, 效果一样
        .withUser("admin").password("admin").roles("USER", "ADMIN");
}
```

### 使用数据库定义的用户信息

指定数据库, 但是对表的结构有要求, 需要满足 spring security 的预设

```java
@Autowired
private DataSource dataSource;
@Override
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    // 如果使用 spring security 的用户授权信息表, 仅此一行就够了
    auth.jdbcAuthentication().dataSource(dataSource)
        // 如果你的用户信息表与 spring security 的有所不同, 可以指定查询语句
        .usersByUsernameQuery("通过用户名查询用户信息的sql")
        .authoritiesByUsernameQuery("通过用户名查询授权信息的sql")
        // 如果数据库中的密码采用了加密存储, 可以指定密码的编码器, 以便正常认证
        // 可以接受所有 PasswordEncoder 的实现类
        .passwordEncoder(new BCryptPasswordEncoder());
}
```

### 使用 LDAP 认证



### 使用自定义的用户信息

如果我们使用其他的用户信息存储, 则需要提供一个自定义的 `UserDetailService` 的实现, 然后通过用户名获取到具体的用户信息

```java
public interface UserDetailsService {
    // 通过用户名, 获取到用户的具体信息
    UserDetails loadUserByUsername(String var1) throws UsernameNotFoundException;
}
```

然后指定使用自定义的 `UserDetailsService`

```java
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.userDetailsService(new SecurityUserDetailService());
}
```

### 使用自定义的验证逻辑

如果使用的不是 spring security 预设的基于授权的控制逻辑, 则需要使用提供自定义的认证器

```java
public void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.authenticationProvider(
        // 自定义的认证器
        new CustomAuthenticationProvider(
            myUserDetailService, 
            new BCryptPasswordEncoder(), 
            userFeignClient, 
            jedisUtil)
    );
}
```

其中, 自定义的认证器需要实现 `AuthenticationProvider` 接口, 实现两个方法

```java
// 自定义的认证逻辑, 验证通过需要返回一个 Authentication 对象, 并在其中添加用户信息
Authentication authenticate(Authentication var1) throws AuthenticationException;

// 当前认证器支持什么认证方式
boolean supports(Class<?> var1);
```

## 2. http 请求安全处理

需要对不同的请求进行细粒度的权限控制, 则需要重载 `configure(HttpSecurity)` 方法.

一般的配置逻辑如下

- 使用 `and()` 开启一段配置
- 指定要设置的 url, 可以使用 `antMachers()`/`regexMatchers()`/`anyRequest()` 等规则
- 指定上述 url 要的拦截规则, 放行/已登录/其他规则

spring security 会依次校验规则, 因此需要将具体的规则放在前面, 通用的规则放在后面

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http
        .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS) // 不使用session

        // 自定义登录逻辑, 登录接口, 登录成功/登录失败的处理
        .and()
        .formLogin()
        .loginProcessingUrl("/login")
        .successHandler(loginAuthenticationSuccessHandler)
        .failureHandler(loginAuthenticationFailureHandler)

        // 设置请求的鉴权方式
        .and()
        .authorizeRequests()
        // 指定url放行
        // ant 风格的匹配表达式, 直接允许
        .antMatchers("/actuator/**").permitAll()
        // 使用正则表达式, 需要登录(已有userDetail信息)
        .regexMatchers("/user/.*").authenticated()
        // 满足 spring el 表达式(这里调用 rbacService.hasPermission 方法)
        .anyRequest().access("@rbacService.hasPermission(request,authentication)")

        // 设置异常处理器
        .and()
        .exceptionHandling()
        // 没权限的, AccessDeniedException
        .accessDeniedHandler(requestAccessDeniedHandler)
        // 没登录的, AuthenticationException
        .authenticationEntryPoint(loginAuthenticationEntryPoint)

        // 可以使用另外的登录逻辑
        .and().apply(smsCodeAuthenticationSecurityConfig)
        .and().apply(wechatMobileAuthenticationSecurityConfig)
        
        // 跨域
        .and().cors()
        .and().csrf().disable();

    // 添加过滤器, 有一个map记录了序号, 最后按顺序调用, 若序号相同, 则按添加顺序调用
    // 如 UsernamePasswordAuthenticationFilter : 800
    // 则 captchaFilter = 800-1 = 799
    //    rateFilter = 799+1 =800
    //    loginFilter = 800-1 = 799
  	// captchaFilter>loginFilter>rateFilter>UsernamePasswordAuthenticationFilter
    http
        .addFilterBefore(captchaFilter, UsernamePasswordAuthenticationFilter.class)
        .addFilterAfter(rateFilter, CaptchaFilter.class)
        .addFilterBefore(loginFilter, UsernamePasswordAuthenticationFilter.class);
}
```

在校验方法中, 最灵活的就是使用 `access()` 方法, 该方法接受一个 SpEL 表达式, 当结果为 true 时放行.

#### 安全性

如果需要, 可以指定某些接口使用或不使用 https

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .anyRequest().authenticated()

        .and()
        // requiresChannel 开始通道的配置
        .requiresChannel()
        // requiresInsecure 声明使用 http
        .antMatchers("/http/**").requiresInsecure()
        // requiresSecure 声明使用 https
        .antMatchers("/https/**").requiresSecure();
}
```

#### CSRF 防护

spring security 默认开启了 csrf 防护, 对于会引起状态变化的请求(非 get/head/options/trace 的请求), 都要求其提交一个名为`_csrf` 的 token 并与服务器内的数据进行校验

这对于现代前后端分离的应用来说并不适用, 因此可以将其关闭

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http...
		// 关闭 csrf 防护
        .csrf().disavle();
}
```

## 3. web安全

## 身份认证

### 1. 页面方式

自定义登录页

登录失败页

登录成功后页面

### 2. 接口方式

登录接口

登录成狗后

## 保护方法

# █ Spring 后端

# 一. Spring JDBC

### 3.1. Spring JDBC

Spring 框架对 jdbc 的封装。

### 3.2. 基本应用

导包

`spring-webmvc, spring-jdbc, ojdbc, dbcp, junit `

添加 spring 配置文件, 配置连接池, 组件扫描

```xml
<!-- 读取 db.properties 文件 -->
<util:properties id="db" location="classpath:db.properties"/>
<!-- 配置连接池 -->
<bean id="ds" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
    <property name="driverClassName" value="#{db.driver}" />
    <property name="url" value="#{db.url}" />
    <property name="username" value="#{db.user}" />
    <property name="password" value="#{db.pwd}" />
</bean>
<!-- 配置组件扫描 -->
<context:component-scan base-package="dao" />
```

配置 JdbcTemplate。

```xml
<!-- 配置 JDBC_Template -->
<bean id="jt" class="org.springframework.jdbc.core.JdbcTemplate" >
    <property name="dataSource" ref="ds" />
</bean>
```

注：JdbcTemplate把一些重复性的代码（比如获取连接，关闭连接，异常处理等等都写好了），我们只需要调用该对象
的方法就可以很方便的访问数据库。

![image-20200421211601307](Spring实战.assets/image-20200421211601307.png)

调用 JdbcTemplate 的方法来访问数据库。

注：通常将JdbcTemplate注入到DAO。

![image-20200421211612511](Spring实战.assets/image-20200421211612511.png)

3.3. 案例: 查询员工表

1) 创建数据表

```sql
create table t_emp(
    id number(8) primary key,
    name varchar2(20),
    age number(3)
);
create sequence t_emp_seq;
create table tt_dept(
    id number(8) primary key,
    deptname varchar2(20),
    addr varchar2(50)
);
create sequence t_dept_seq;
```

2) 创建 DAO 类

```java
@Repository("employeeDao")
public class EmployeeDao {
    // 可以用自动装配
    //@Autowired
    //@Qualifier("jt")
    // 也可以用 Resource
    // 来自 JavaEE, 可以手动导入 javax.annotation-api 包
    @Resource(name="jt")
    private JdbcTemplate jt;
    public void save(Employee e){
        String sql= "insert into t_emp values(t_emp_seq.nextval,?,?)";
        Object[] args ={e.getName(),e.getAge()};
        jt.update(sql,args);
    }
    public List<Employee> findAll(){
        String sql = "select * from t_emp";
        return jt.query(sql, new EmpRowMapper());
    }
    public Employee findById(int id){
        String sql = "select * from t_emp where id=?";
        Object[] args = {id};
        Employee emp = null;
        try{
            emp = jt.queryForObject(sql, args,new EmpRowMapper());
        }catch (EmptyResultDataAccessException e){
            return null;
        }
        return emp;
    }
    public void modify(Employee e){
        String sql = "update t_emp set name=?,age=? where id=?";
        Object[] args = {e.getName(),e.getAge(),e.getId()};
        jt.update(sql,args);
    }
    public void delete(int id){
        String sql = "delete from t_emp where id=?";
        Object[] args = {id};
        jt.update(sql,args);
    }
    // 封装了对 ResultSet 的处理
    class EmpRowMapper implements RowMapper<Employee> {
        /*
 		 * 告 jdbcTemplate, 如何将一条记录转换成一个实体对象
		 * index: 正在被处理的记录的下标
		 */
        public Employee mapRow(ResultSet rs, int rowNum) throws SQLException {
            Employee e = new Employee();
            e.setId(rs.getInt("id"));
            e.setName(rs.getString("name"));
            e.setAge(rs.getInt("age"));
            return e;
        }
    }
}
```

3) 创建实体类

```java
public class Employee implements Serializable{
    private static final long serialVersionUID = 5272466894454999383L;
    // getter, setter, toString 此处省略
}
```

4) 创建测试类

```java
@SuppressWarnings("resource")
public class TestCase {
    private EmployeeDao dao;
    @Before
    // 运行 @test 测试方法前 , 会先执行 @Before 的方法 , 完成准备工作
    public void init(){
        String config = "spring-JDBC.xml";
        ApplicationContext ac = new ClassPathXmlApplicationContext(config);
        dao = ac.getBean("employeeDao",EmployeeDao.class);
    }
    @Test // 新增记录
    public void test1(){
        Employee e = new Employee();
        e.setName(" 巢皮 ");
        e.setAge(30);
        dao.save(e);
    }
    @Test // 查询所有
    public void test2(){
        List<Employee> emps = dao.findAll();
        System.out.println(emps);
    }
    @Test // 查询一个
    public void test3(){
        Employee emp = dao.findById(10);
        System.out.println(emp);
    }
    @Test // 修改记录
    public void test4(){
        Employee emp = dao.findById(1);
        System.out.println(emp);
        emp.setName(" 熊坡 ");
        emp.setAge(25);
        dao.modify(emp);
        emp = dao.findById(1);
        System.out.println(emp);
    }
    @Test // 删除一个
    public void test5(){
        Employee emp = dao.findById(1);
        System.out.println(emp);
        dao.delete(1);
        emp = dao.findById(1);
        System.out.println(emp);
    }
}
```

## 事务控制

### 2.4. 声明式事务

#### 编程式事务处理

传统编程事务处理非常繁琐:

```java
try{
    conn= 打开连接
        conn.setAutoCommit(false);
    // 数据库操作 1
    // 数据库操作 2
    // 数据库操作 3
    conn.commit();
}catch(e){
    conn.rollback();
}finally{
    conn.close()
}
```

#### 声明式事务处理

Spring 支持声明式事务处理, 底层是利用AOP实现的, 只需要简单配置即可使用.

就是所有业务方法使用`@Transactional`, 如果是只读方法, 建议增加 `readOnly=true`

![image-20200426103804367](Spring实战.assets/image-20200426103804367.png)

#### 使用声明式事务处理

1. 配置事务管理器

  ```xml
  <!-- spring-mybatis.xml -->
  <bean id="txManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
      <property name="dataSource" ref="dataSource"/>
  </bean>
  <!-- 设置 注解驱动的事务管理 -->
  <tx:annotation-driven transaction-manager="txManager"/>
  ```

2. 在业务方法上使用 事务注解即可

#### 案例: 批量删除

![image-20200426103817163](Spring实战.assets/image-20200426103817163.png)

1. 开发持久层

2. 持久层 NoteDao

  ```java
  int deleteNoteById(String noteId);
  ```

  SQL NoteMapper.xml

  ```xml
  <delete id="deleteNoteById" parameterType="string">
      delete from cn_note where cn_note_id=#{noteId}
  </delete>
  ```

3. 业务层

  业务层接口方法 NoteService

  ```java
  int deleteNotes(String... noteIds) throws NoteNotFoundException;
  ```

  实现业务方法 NoteServiceImpl

  ```java
  @Transactional
  public int deleteNotes(String... noteIds)
      throws NoteNotFoundException {
      for(String id: noteIds){
          int n=noteDao.deleteNoteById(id);
          if(n!=1){
              throw new NoteNotFoundException("ID 错误 ");
          }
      }
      return noteIds.length;
  }
  ```

  抛出异常 NoteNotFoundException 时候会触发Spring事务回滚操作.

4. 测试 NoteServerTest

  ```java
  @Test
  public void testDeleteNotes(){
      String id1 = "3febebb3-a1b7-45ac-83ba-50cdb41e5fc1";
      String id2 = "9187ffd3-4c1e-4768-9f2f-c600e835b823";
      String id3 = "ebd65da6-3f90-45f9-b045-782928a5e2c0";
      String id4 = "A";//"fed920a0-573c-46c8-ae4e-368397846efd";
      int n = service.deleteNotes( id1, id2, id3, id4);
      //int n = service.deleteNotes(
      // new String[]{id1, id2, id3, id4});
      System.out.println(n);
  }
  ```

  在提交的ID有错误时候会出现异常, 并且回滚数据库事务.



### Spring 事务管理

#### 只读属性

对于单纯读取数据库操作, 可以设置 `readOnly=true`, 可以提高数据的放效率.

```java
@Transactional(readOnly=true)
public List<Map<String, Object>> listNotesInTrashBin(
    String userId) throws UserNotFoundException {
```

#### 事务的传播属性

业务过程重构时候需要业务方法调用业务方法, 这样就需要一个业务方法的事务传播到另外一个业务方法中, 整合为一个事务.

案例:

![image-20200426103835386](Spring实战.assets/image-20200426103835386.png)

![image-20200426103842549](Spring实战.assets/image-20200426103842549.png)

步骤:

1. 创建数据表:

  ```sql
  create table cn_star (
      cn_star_id varchar(50),
      cn_user_id varchar(50),
      cn_stars int,
      primary key(cn_star_id)
  );
  ```

2. 声明 StarsDao

  ```java
  public interface StarsDao {
      Stars findStarsByUserId(String userId);
      int insertStars(Stars stars);
      int updateStars(Stars stars);
  }
  ```

3. 声明SQL StarsMapper.xml

  ```xml
  <mapper namespace="cn.tedu.note.dao.StarsDao">
      <select id="findStarsByUserId" parameterType="string" resultType="entity.Stars">
          select cn_star_id as id, cn_user_id as userId, cn_stars as stars
          from cn_star
          where cn_user_id = #{userId}
      </select>
      <insert id="insertStars" parameterType="entity.Stars">
          insert into cn_star( cn_star_id, cn_user_id, cn_stars)
          values(#{id}, #{userId}, #{stars})
      </insert>
      <update id="updateStars" parameterType="entity.Stars">
          update cn_star
          set cn_stars = #{stars}
          where cn_user_id=#{userId} or cn_star_id=#{id}
      </update>
  </mapper>
  ```

4. 声明业务层接口方法 NoteService

  ```java
  boolean addStars(String userId, int stars) throws UserNotFoundException;
  ```

5. 实现方法 NoteServiceImpl

  ```java
  @Transactional
  public boolean addStars(String userId, int stars)
      throws UserNotFoundException {
      if(userId==null||userId.trim().isEmpty()){
          throw new UserNotFoundException("ID 空 ");
      }
      User user=userDao.findUserById(userId);
      if(user==null){
          throw new UserNotFoundException(" 木有人 ");
      }
      // 检查是否已经有星了
      Stars st=starsDao.findStarsByUserId(userId);
      if(st==null){// 如果没有星星
          String id = UUID.randomUUID().toString();
          st = new Stars(id, userId, stars);
          int n = starsDao.insertStars(st);
          if(n!=1){
              throw new RuntimeException(" 失败 ");
          }
      }else{// 如果有星星 , 就在现有星星数量上增加
          int n = st.getStars()+stars;
          if(n<0){
              // n = 0;
              throw new RuntimeException(" 扣分太多 !");
          }
          st.setStars(n);
          n = starsDao.updateStars(st);
          if(n!=1){
              throw new RuntimeException(" 失败 ");
          }
      }
      return true;
  }
  ```

6. 测试: NoteServiceTest

  ```java
  @Test
  public void testAddStars(){
      String userId="03590914-a934-4da9-ba4d-b41799f917d1";
      boolean b = service.addStars(userId, 5);
      System.out.println(b);
      b = service.addStars(userId, 6);
      System.out.println(b);
  }
  ```

7. 重构 addNote方法, 实现添加笔记时候送星星功能, 进而实现事务的传播:

  ```java
  @Transactional
  public Note addNote(String userId, String notebookId, String title)
      throws UserNotFoundException, NotebookNotFoundException {
      if(userId==null||userId.trim().isEmpty()){
          throw new UserNotFoundException("ID 空 ");
      }
      User user=userDao.findUserById(userId);
      if(user==null){
          throw new UserNotFoundException(" 木有人 ");
      }
      if(notebookId==null||notebookId.trim().isEmpty()){
          throw new NotebookNotFoundException("ID 空 ");
      }
      int n=notebookDao.countNotebookById(notebookId);
      if(n!=1){
          throw new NotebookNotFoundException(" 没有笔记本 ");
      }
      if(title==null || title.trim().isEmpty()){
          title=" 葵花宝典 ";
      }
      String id = UUID.randomUUID().toString();
      String statusId = "1";
      String typeId = "1";
      String body = "";
      long time=System.currentTimeMillis();
      Note note = new Note(id,notebookId,userId,statusId,typeId,title,body,time,time);
      n = noteDao.addNote(note);
      if(n!=1){
          throw new NoteNotFoundException(" 保存失败 ");
      }
      // 当前的事务 , 会传播到 addStart 方法中整合为一个事务 !
      addStars(userId, 5);
      return note;
  }
  ```

##### 事务传播(propagation)属性

事务的传播属性, 用来指定多个业务方法事务间的关系, 通过`@Transactional`注解中的`propagation`来设置.

`org.springframework.transaction.annotation.Propagation`提供了多个选项, 默认使用`REQUIRED`

| 传播属性        | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| `REQUIRED`      | 需要事务, 如果没有事务创建新事务, 如果当前有事务参与当前事务 |
| `MANDATORY`     | 必须有事务, 如果当前没有事务就抛异常                         |
| `NEVER`         | 绝不, 绝对不能有事务传播, 但可以有自己的事务, 如果在事务中调用则抛出异常 |
| `NESTED`        | 嵌套, 必须被嵌套到其他事务中                                 |
| `NOT_SUPPORTED` | 不支持事务, 不论是事务传播还是自己的事务                     |
| `SUPPORTS`      | 支持事务, 如果没有事务也不会创建新事务                       |
| `REQUIRES_NEW`  | 必须是新事务, 如果有当前事务, 挂起当前事务并且开启新事务.    |

#### 事务隔离属性

隔离属性, 用来指示并发操作相同的数据记录时要如何处理. 通过 `@Transactional` 的 `isolation` 属性来设置. 

`org.springframework.transaction.annotation.Isolation`提供了多个选项, 通常使用`READ_COMMITTED` .

| 隔离级别           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `DEFAULT`          | 使用数据库默认的隔离级别                                     |
| `SERIALIZABLE`     | 串行化, 所有事务排队操作, 前一个事务完全结束后, 后续事务才能操作. 并发性能最低, 限制最高, 最安全 |
| `REPEATABLE_READ`  | 可以重复读取                                                 |
| `READ_COMMITTED`   | 可以读取提交后的数据, 执行更新操作时再锁定数据, 锁定期间, 后续事务能读取修改前的已提交数据, |
| `READ_UNCOMMITTED` | 可以读取未提交的数据, 执行更新操作时不锁定数据, 其他事务都能看到修改后未提交的数据, 并发效率最高, 但是最不安全. 因为修改数据的时候有可能会回滚, 那么其他事务就会获取到错误不可靠的数据. |

![image-20200426103918878](Spring实战.assets/image-20200426103918878.png)



# 二. Spring 与 ORM

# 三. NoSQL

# 四. 缓存

# 五. 保护方法

# 六. 

# █ Spring 集成

## 一. 远程服务

## 二. REST 服务

## 三. 消息队列

### 1. 概述

### 2. JMS 与 ActiveMQ

### 3. AMQP 与 RabbitMQ

## 四. WebSocket

## 五. JMX

## 六. Spring Boot



