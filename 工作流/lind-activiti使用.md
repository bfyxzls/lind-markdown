# 作用
快速为你的项目接入工作流的功能，同时对外开放了一些标准的事件，方便使用者进行个性化功能的扩展
> 注意：如果你项目中使用了mybatis，需要注意它与lind-activiti里的mybatis的版本，当出现冲突后，可以把lind-activiti里的排除

# 依赖引用
```
<dependency>
    <groupId>com.pkulaw</groupId>
    <artifactId>pkulaw-activiti</artifactId>
    <version>1.0.0</version>
</dependency>
```
# 配置
```
spring:
  activiti:
    check-process-definitions: false
    font:
      activityFontName: 宋体
      labelFontName: 宋体
```

# 使用
```$xslt
@SpringBootApplication(exclude = {org.activiti.spring.boot.SecurityAutoConfiguration.class,
        org.springframework.boot.autoconfigure.security.servlet.SecurityAutoConfiguration.class,})

public class Application {
    public static void main(String[] args) {
        SpringApplication.run(FilingApplication.class, args);
    }
}
```
# 事件监听
* com.pkulaw.activiti.event包
* ActivitiEventListener 工作流级别的事件监听
* TaskListener 某个节点产生的事件监听

# 测试地址
* 模型列表：/view/model/list
