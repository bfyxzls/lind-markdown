# @Configuration(proxyBeanMethods = false)注解

@Configuration(proxyBeanMethods = false),Spring 5.2.0+的版本，建议你的配置类均采用Lite模式去做，即显示设置proxyBeanMethods = false。Spring Boot在2.2.0版本（依赖于Spring 5.2.0）起就把它的所有的自动配置类的此属性改为了false，即@Configuration(proxyBeanMethods = false)，提高Spring启动速度

在Spring框架中，`proxyBeanMethods`是一个配置选项，它用于控制是否使用CGLIB（Code Generation Library）代理来创建@Bean注解标记的组件的实例。在了解`proxyBeanMethods`的作用之前，我们需要先了解一下Spring中的代理对象和@Bean注解。

在Spring中，代理对象是在运行时生成的对象，它们包装了被代理对象，并可以拦截对被代理对象方法的调用，从而允许在方法调用前后执行额外的逻辑。Spring使用代理对象来实现诸如事务管理、AOP（面向切面编程）等功能。

@Bean注解用于标记一个方法，该方法将返回一个由Spring管理的Bean实例。通过在配置类中使用@Bean注解，我们可以告诉Spring容器在需要时如何创建和配置Bean实例。

现在，我们来看一下`proxyBeanMethods`的作用。它有两个可选值：

1. `true`（默认值）：表示使用CGLIB代理来创建@Bean注解标记的组件的实例。这意味着每次调用该@Bean方法时，Spring都会返回一个新的代理对象实例。这样可以确保在每次方法调用时，代理对象都能执行额外的逻辑（如事务管理）。
2. `false`：表示不使用代理，直接返回被@Bean注解标记的方法所创建的实例。这样，每次调用该@Bean方法时，Spring都会返回同一个实例对象。这对于状态不变的单例Bean很有用，可以提高性能。

使用`proxyBeanMethods`的目的是为了控制@Bean方法返回的对象是代理对象还是原始对象。这取决于你是否需要在方法调用时执行额外的逻辑，以及你是否需要保证每次调用@Bean方法时返回一个新的实例。

总结一下，`proxyBeanMethods`的作用是控制Spring是否使用代理对象来创建@Bean注解标记的组件实例，从而决定是否在每次方法调用时执行额外的逻辑，并确保返回一个新的实例对象。

# @AutoConfigureBefore注解
`@AutoConfigureBefore`是一个Spring Boot注解，用于控制自动配置类的加载顺序。当多个自动配置类存在时，可以使用`@AutoConfigureBefore`注解来明确它们之间的加载顺序。

通过在自动配置类上添加`@AutoConfigureBefore`注解，可以指定该自动配置类应该在指定的自动配置类之前加载。这可以用于解决依赖关系或确保某个自动配置类在其他自动配置类之前生效。

`@AutoConfigureBefore`注解接受一个或多个自动配置类作为参数，参数类型为Class。示例用法如下：

```java
@Configuration
@AutoConfigureBefore({FirstAutoConfiguration.class, SecondAutoConfiguration.class})
public class MyAutoConfiguration {
    // 自动配置类的定义
}
```

在上述示例中，`MyAutoConfiguration`自动配置类将在`FirstAutoConfiguration`和`SecondAutoConfiguration`之前进行加载。

需要注意的是，`@AutoConfigureBefore`注解并不能完全控制自动配置类的加载顺序。实际的加载顺序还受到其他因素的影响，如类路径扫描顺序、条件注解的匹配等。因此，使用`@AutoConfigureBefore`时应谨慎，确保理解自动配置的整体加载机制。

另外，还有一个相关的注解`@AutoConfigureAfter`，它的作用与`@AutoConfigureBefore`相反，用于指定某个自动配置类应该在指定的自动配置类之后加载。这两个注解可以结合使用，以更精确地控制自动配置类的加载顺序。

# @ConfigurationProperties注解

`@ConfigurationProperties`是一个Spring Boot注解，用于将外部配置属性绑定到Java类的属性上。它可以帮助我们方便地将配置文件中的属性值注入到应用程序中的对应属性中。

通过使用`@ConfigurationProperties`注解，我们可以创建一个与配置文件中的属性结构相对应的Java类，并将属性值自动绑定到该类的属性上。这样，在应用程序中可以直接使用该类的实例来访问和操作配置属性。

以下是一个示例：

```java
@ConfigurationProperties(prefix = "myapp")
public class MyAppProperties {
    private String name;
    private int version;

    // 省略 getter 和 setter 方法

    // 其他自定义方法
}
```

在上述示例中，我们创建了一个名为`MyAppProperties`的Java类，并使用`@ConfigurationProperties`注解指定了属性前缀为`myapp`。这意味着，配置文件中以`myapp`开头的属性将与该类的属性进行绑定。

接下来，在应用程序的配置文件（例如`application.properties`或`application.yml`）中，可以定义与`myapp`前缀匹配的属性：

```properties
myapp.name=My Application
myapp.version=1
```

现在，当应用程序启动时，Spring Boot将自动将配置文件中的属性值注入到`MyAppProperties`类的对应属性中。我们可以通过将`MyAppProperties`类注入到其他组件中来访问和使用这些属性。

```java
@Service
public class MyService {
    private final MyAppProperties appProperties;

    public MyService(MyAppProperties appProperties) {
        this.appProperties = appProperties;
    }

    public void doSomething() {
        String appName = appProperties.getName();
        int appVersion = appProperties.getVersion();
        // 使用应用程序属性进行操作
    }
}
```

在上述示例中，`MyService`组件通过构造函数注入`MyAppProperties`类，从而可以访问配置文件中的属性值。

总结一下，`@ConfigurationProperties`注解用于将外部配置文件中的属性值绑定到Java类的属性上，方便我们在应用程序中使用配置属性。

# EnableConfigurationProperties注解
`@EnableConfigurationProperties`是一个Spring Boot注解，用于启用使用`@ConfigurationProperties`注解标记的类作为Spring Bean来注入配置属性。

当我们使用`@ConfigurationProperties`注解定义一个类来绑定配置属性时，如果想要将该类实例化为一个Spring Bean，并使其在应用程序中可用，需要在配置类或主应用程序类上添加`@EnableConfigurationProperties`注解。

以下是一个示例：

```java
@Configuration
@EnableConfigurationProperties(MyAppProperties.class)
public class AppConfig {
    // 配置类的定义
}
```

在上述示例中，`@EnableConfigurationProperties`注解用于启用`MyAppProperties`类作为配置属性的绑定类，并将其实例化为一个Spring Bean。这样，在应用程序中就可以通过依赖注入将`MyAppProperties`类的实例注入到其他组件中使用。

需要注意的是，`@EnableConfigurationProperties`注解通常与`@Configuration`注解一起使用，以确保配置类被正确加载和处理。它可以放置在配置类上或主应用程序类上。

另外，当使用`@EnableConfigurationProperties`注解时，还需要确保目标类（例如`MyAppProperties`）已经被标记为`@Component`或`@Configuration`等注解，以使其成为一个Spring Bean。

总结一下，`@EnableConfigurationProperties`注解用于启用使用`@ConfigurationProperties`注解定义的类作为配置属性的绑定类，并将其实例化为Spring Bean，以便在应用程序中使用。

# @EnableGlobalMethodSecurity(prePostEnabled = true)这句话的作用
`@EnableGlobalMethodSecurity(prePostEnabled = true)`这句话的作用是在Spring应用程序中启用方法级别的安全性。

通过在Spring应用程序的配置类中添加这个注解，你可以使用Spring Security的`@PreAuthorize`和`@PostAuthorize`注解来对单个方法进行访问控制。

其中，`prePostEnabled = true`参数特别用于启用`@PreAuthorize`和`@PostAuthorize`注解的使用。通过`@PreAuthorize`注解，你可以指定在方法调用之前要评估的表达式，以确定用户是否有权限访问该方法。类似地，`@PostAuthorize`用于指定在方法执行后要评估的表达式，以验证返回结果。

下面是一个使用`@EnableGlobalMethodSecurity`的配置类示例：

```java
@Configuration
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends GlobalMethodSecurityConfiguration {

    @Autowired
    private CustomPermissionEvaluator customPermissionEvaluator;

    @Override
    protected MethodSecurityExpressionHandler createExpressionHandler() {
        DefaultMethodSecurityExpressionHandler expressionHandler =
            new DefaultMethodSecurityExpressionHandler();
        expressionHandler.setPermissionEvaluator(customPermissionEvaluator);
        return expressionHandler;
    }
}
```

在上面的示例中，`SecurityConfig`是一个配置类，用于启用全局方法安全性。它扩展了`GlobalMethodSecurityConfiguration`类，以提供一个自定义的`MethodSecurityExpressionHandler`，负责评估`@PreAuthorize`和`@PostAuthorize`注解中使用的表达式。`CustomPermissionEvaluator`是`PermissionEvaluator`接口的自定义实现，用于处理表达式中的权限评估。

通过启用全局方法安全性，你可以在方法上使用`@PreAuthorize`和`@PostAuthorize`注解，根据这些注解中定义的表达式来控制访问权限。

# 在spring.factory里声明的类，还需要在类上面添加@Component吗
在Spring的应用中，`@Component`注解用于将一个类声明为Spring管理的组件，让Spring能够自动扫描并将其实例化为Bean。但是，对于在`spring.factory`文件中声明的类，你不需要再在类上添加`@Component`注解。

`spring.factory`是Spring Boot自动生成的一个配置文件，它用于收集应用中的所有`@Configuration`类和其他一些特定的注解类，例如`@EnableAutoConfiguration`和`@ComponentScan`等。这些类都被Spring Boot自动识别并加载。

当Spring Boot扫描到`spring.factory`文件中的类时，它会自动创建相应的Bean并添加到应用的上下文中，无需额外的`@Component`注解。

总结起来，对于在`spring.factory`文件中声明的类，不需要再在类上添加`@Component`注解，Spring Boot会自动处理它们的实例化和管理。
