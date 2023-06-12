# 依赖关系

MqProducerBeanDefinitionRegistry > MqProducerFactoryBean > MqProducerProxy

# FactoryBean

FactoryBean是一个工厂Bean，可以生成某一个类型Bean实例，它最大的一个作用是：可以让我们自定义Bean的创建过程。

```
public interface FactoryBean<T> {

    //返回的对象实例
    T getObject() throws Exception;
    //Bean的类型
    Class<?> getObjectType();
    //true是单例，false是非单例  在Spring5.0中此方法利用了JDK1.8的新特性变成了default方法，返回true
    boolean isSingleton();
}
```

# BeanFactory

BeanFactory是Spring容器中的一个基本类也是很重要的一个类是Spring容器中的一个基本类也是很重要的一个类，在BeanFactory中可以`创建和管理`Spring容器中的Bean，它对于Bean的创建有一个统一的流程。

Spring的本质是一个bean工厂(beanFactory)或者说bean容器，它按照我们的要求，生产我们需要的各种各样的bean，提供给我们使用。
只是在生产bean的过程中，需要解决bean之间的依赖问题，才引入了依赖注入(DI)这种技术。也就是说依赖注入是beanFactory生产bean时为了解决bean之间的依赖的一种技术而已。

beanFactory会在bean的生命周期的各个阶段中对bean进行各种管理，并且spring将这些阶段通过各种接口暴露给我们，让我们可以对bean进行各种处理，我们只要让bean实现对应的接口，
那么spring就会在bean的生命周期调用我们实现的接口来处理该bean.

在介绍BeanFactory前，我们先想一个问题。我们都知道BeanFactory主要的内容是帮我们生成Bean信息和管理Bean信息，那么我们在xml文件中<baen>属性的时候。Spring是如何帮我们生成Bean的呢

## FactoryBean和BeanFactory的关系

* beanFactory,bean的工厂
  * beanFactory是所有spring bean容器的顶级接口，它为spring的容器定义了一套规范，并提供像getBean这样的方法从容器中获取指定的bean实例
  * beanFactory在产生Bean的同时，还提供了解决Bean之间的依赖注入的能力，也就是所谓的DI.
* factoryBean,工厂bean，生产bean实例
  * factoryBean是一个工厂bean，它是一个接口，它的主要功能是动态生成某一类型的bean的一个实例.
  * 通过实现factoryBean，我们可以自定义一个bean，并且加载到ioc容器里面
  * 它里面有一个重要的方法叫getObject()，这个方法就是实现动态构建bean的一个过程
  * spring cloud里的openFeign组件，就是使用factoryBean来实现的
    BeanFactory和BeanDefinition是Spring中两个很重要的概念，它们是Spring中实现IoC容器的基础。BeanFactory是一个顶层接口，定义了IoC容器的核心功能，可以理解为IoC容器的基础设施；而BeanDefinition则是描述IoC容器要管理的Bean对象的元信息，可以理解为Bean的具体定义。它们之间的关系可以简述如下：

# BeanDefinitionReader

从xml文件、类路径下使用了@Component系列注解的类、或者从@Configuration注解的配置类，获取BeanDefintiions，然后注册到BeanFactory中。
我们通过BeanDefinitionReader从不同的方式，获取BeanDefintiions Bean的元数据。

那么BeanDefintiions是什么？

```
    	protected AbstractBeanDefinition(BeanDefinition original) {
    		setParentName(original.getParentName());
    		setBeanClassName(original.getBeanClassName());
    		setScope(original.getScope());
    		setAbstract(original.isAbstract());
    		setFactoryBeanName(original.getFactoryBeanName());
    		setFactoryMethodName(original.getFactoryMethodName());
    		setRole(original.getRole());
    		setSource(original.getSource());
    		copyAttributesFrom(original);
             。。。。。。省略
    }
```
从上面的代码我们大致的看出。BeanDefintiions其实就是对Bean的一些元数据定义,包括parenName 父类名称 baenClassName：类名，scope bean的作用域。Abstract是否是抽象的等信息。
通过 BeanDefinitionReader获取到BeanDefinition之后 。我们在通过BeanDefinitionRegistry将beanDefinition注册到BeanFacory中。存储在BeanFactory的一个conCurrentHashMap中。key为beanName,Value就是BeanDefinition元数据。
那么获取Bean就从conCurrentHashMap中通过BeanName获取对应的Bean信息。

从上面的分析：我们可以看到Bean的加载解析过程如下图所示：
![](./assets/img.png)

接下来我们针对BeanDefinitionReader、BeanDefinitionRegistry、BeanFactory分别分析：

## BeanDefinitionReader的实现

1. XmlBeanDefinitionReader：基于XML文件
   读取解析xml文件，通过Parser解析xml文件的标签。
   针对beans标签，生成对应的BeanDefintions，然后注册到BeanFactory中。
   针对其他有特殊功能的标签，如context:component-scan，context:anotation-config，还可以生成BeanFactoryPostProcessor，BeanPostProcessor接口实现类的bean等；除了可以生成BeanDefinitions之外，还可以实现其他功能。NamespaceHandler：XML标签名称空间处理器
   被XmlBeanDefinitionReader使用，XmlBeanDefinitionReader在处理每个XML标签名称空间的时候，如applicationContext.xml的context:，mvc:，通过一个DefaultNamespaceHandlerResolver来获取对应的NamespaceHandler实现类，然后通过这个NamespaceHandler实现类，进一步获取该命名空间的内部标签对应的BeanDefinitionParser实现类。

被XmlBeanDefinitionReader使用，专门用于处理xml文件的beans标签的标签处理器。即XmlBeanDefinitionReader读取xml文件，创建Document对象，然后交给BeanDefinitionDocumentReader处理。BeanDefinitionDocumentReader解析Document对象的Element节点，然后创建BeanDefinitions集合，通过XmlBeanDefinitionReader注册到XmlBeanDefinitionReader所在的BeanFactory。

2. AnnotatedBeanDefinitionReader：注册指定的类列表annotatedClasses
   可以使用编程方法，显示指定将哪些类需要注册到BeanFactory。
   主要是被AnnotationConfigApplicationContext使用，即基于注解配置的ApplicationContext，这是SpringBoot的默认ApplicationContext。典型使用为：先获取所有使用了@Configuration注解的类，然后通过AnnotatedBeanDefinitionReader生成与这些类对应的BeanDefinitions，并注册到BeanFactory。
3. ClassPathBeanDefinitionScanner：注册指定的basePackages下面的类
   扫描指定类路径（包）下面的类，检测是否存在@Component注解及其子注解，从而生成BeanDefinition，然后注册到BeanFactory。没有实现BeanDefinitionReader接口，但基于相同的设计思路：BeanDefinitionReader。与AnnotatedBeanDefinitionReader一样，都是获取指定类，生成该类的BeanDefinition注册到BeanFactory，而不是像xml文件一样已经通过bean标签显示说明这个就是bean。也是主要是被AnnotationConfigApplicationContext使用。与其他两种也是基于basePackages类路径扫描的方式不同之处为：context:component-scan标签：基于XML的ApplicationContext，实现类路径扫描，底层使用ComponentScanBeanDefinitionParser这个parser来处理。@ComponentScan：对于处理@Configuration配置类上面的@ComponentScan注解，则是通过ComponentScanAnnotationParser来处理的。
4. ConfigurationClassBeanDefinitionReader：基于@Configuration注解的类配置
   处理@Configuration注解的配置类，加在这些配置类上面的注解，即与@Configuration一起使用的注解，如@ComponentScan，@PropertySource，@Import，@Profile等。
   ConfigurationClassBeanDefinitionReader主要被ConfigurationClassPostProcessor调用，ConfigurationClassPostProcessor为BeanFactoryPostProcessor

# BeanDefinitionRegistry

注册BeanDefinitions。提供registerBeanDefinition，removeBeanDefinition等方法，用来从BeanFactory注册或移除BeanDefinition。
通常BeanFactory接口的实现类需要实现这个接口。
实现类（通常为BeanFactory接口实现类）的对象实例，被DefinitionReader接口实现类引用，DefinitionReader将BeanDefintion注册到该对象实例中。

除了上述的BeanDefinitionRegisry还有一个负责单例Bean注册的接口：SingletonBeanRegistry

用于注册单例Bean对象实例，实现类定义存储Bean对象实例的map，BeanFactory的类层次结构中需要实现这个接口，来提供Bean对象的注册和从Bean对象实例的map获取bean对象。

# BeanFactory

接口具体实现类

1. DefaultListableBeanFactory

BeanFactory接口体系的默认实现类，实现以上接口的功能，提供BeanDefinition的存储map，Bean对象对象的存储map。

其中Bean对象实例的存储map，定义在FactoryBeanRegistrySupport，FactoryBeanRegistrySupport实现了SingletonBeanRegistry接口，而DefaultListableBeanFactory的基类AbstractBeanFactory，继承于FactoryBeanRegistrySupport。

2. StaticListableBeanFactory

用于存储给定的bean对象实例，不支持动态注册功能，是ListableBeanFactory接口的简单实现。

# beanFactoty后置处理器： BeanFactoryPostProcessor

benFactoryPostProCessor是BeanFactory的后置处理器：

在BeanFactory创建好，加载好其所包含的所有beanDefinitions，但是还没有实例化bean之前，执行，具体为调用postProcessBeanFactory方法。

1. 加载更多的bean元数据
   ConfigurationClassPostProcessor，用于从BeanFactory中检测使用了@Configuration注解的类，对于这些类对应的BeanDefinitions集合，遍历并依次交给ConfigurationClassParser，ConfigurationClassBeanDefinitionReader处理，分别是处理与@Configuration同时使用的其他注解和将类内部的使用@Bean注解的方法，生成BeanDefinition，注册到BeanFactory。
2. 对bean元数据进行加工处理
   BeanDefinition属性填充、修改：在postProcessBeanFactory方法中，可以对beanFactory所包含的beanDefinitions的propertyValues和构造函数参数值进行修改，如使用PropertyPlaceHolderConfigurer来对BeanDefinition的propertyValues的占位符进行填充、赋值。或者使用PropertyResourceConfigurer获取config文件中属性，对BeanDefinitions的相关属性进行赋值或者值覆盖。

bean对象后置处理器：BeanPostProcessor

Bean后置处理器：负责对已创建好的bean对象进行加工处理。

主要是可以对新创建的bean实例进行修改，提供了一个类似于hook机制，对创建好的bean对象实例进行修改。

核心方法
postProcessBeforeInitialization：在创建好bean实例，但是在任何初始化回调执行之前，如InitializingBean的afterPropertiesSet，先执行该方法。

postProcessAfterInitialization：在创建好bean实例，并且所有的初始化回调都执行完了，如InitializingBean的afterPropertiesSet，再执行该方法。

# BeanFactory和BeanDefinition

* BeanFactory是IoC容器的接口，是一个工厂模式的典型实现，提供了获取Bean、注册Bean等基本功能，同时还支持不同种类和不同定义方式的对象实例化和获取，例如从XML、注解或Java框架（如Hibernate、Struts 2等）中定义的Bean。BeanFactory的核心方法是getBean()，用于获取Bean实例。BeanFactory主要负责创建Bean对象，并控制Bean的生命周期、作用域和依赖等关系。
* BeanDefinition是描述IoC容器要管理的Bean对象的元信息，它包含了Bean的定义信息，例如Bean的类型、属性、依赖等信息。Spring会将我们在配置中定义的Bean信息封装为一个BeanDefinition对象，并提供一些API（比如setScope、getPropertyValues等方法）来获取和设置元信息，从而控制Bean的创建、初始化和依赖注入等行为。BeanDefinition是描述Bean信息的抽象模型，但并不创建Bean实例本身，它主要负责Bean的定义和组装，并在BeanFactory中保存和管理Bean定义。BeanDefinition通过BeanName和BeanDefinitionHolder关联到BeanFactory中，存储在BeanFactory的BeanDefinitionMap中，BeanFactory负责解析和执行BeanDefinition。
* 总之，BeanFactory是IoC容器的基本接口，负责创建和管理Bean实例；而BeanDefinition是保存Bean元信息的抽象模型，负责控制Bean的创建和组装��为。BeanDefinition提供了对Bean类型、作用域、属性、依赖、初始化和销毁等方面的控制，而BeanFactory则负责解析和执行BeanDefinition，创建出Bean实例并负责管理Bean实例的声明周期和作用域等。在Spring中，BeanFactory和BeanDefinition密不可分，BeanFactory是BeanDefinition的执行者，并根据BeanDefinition中的元信息进行Bean的实例化和管理。
