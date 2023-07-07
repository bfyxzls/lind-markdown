## 强调和高亮背景

中国是`伟大`的民族！
==Highlight==

## 链接

[大叔博客园](http://www.cnblogs.com/lori "仓储大叔")

## 引用

> .net
>
>> .net standard
>>
>>> .net core
>>>
>>

## 删除

中国是~~社会主义~~的国家

## 斜体和粗体

*斜体*和**粗体**,***斜又粗***

## 上标和下标

* a~1~+a~2~=A
* a^2+b^2=c^2

## 流程图

在gitlab,github网面上需要使用```mermaid才可以渲染流程图

```mermaid
graph TD
A(morgan项目发起结账)-->|说明|B(pixel项目)
B-->C(获取配置服务里的模版)
B-->D(获取报表)
B-->E(获取科目余额)
B-->F(到hawk项目去校验)
```

```mermaid
gantt
dateFormat YYYY-MM-DD
title 微服务
section 服务发现
eureka:2019-05-01,5d
consul:2019-05-04,5d
section 消息队列
rabbitmq:2019-05-01,5d
kafka:2019-05-02,5d
```

```mermaid
sequenceDiagram 　　
participant Alice 　
participant Bob 　
Alice->John:Hello John, how are you? 　
loop Healthcheck 　　　　
    John->John:Fight against hypochondria 
end 　　
Note right of John:Rational thoughts <br/>prevail... 　　
John-->Alice:Great! 
John->Bob: How about you? 
Bob-->John: Jolly good!
```

```mermaid
graph LR 
id1(Start)-->id2(Stop) 
style id1 fill:#f9f,stroke:#333,stroke-width:4px; 
style id2 fill:#ccf,stroke:#f66,stroke-width:2px,stroke-dasharray:5,5;

```

```mermaid
graph LR
classDef default fill:#f90,stroke:#555,stroke-width:4px;
id1(Start)-->id2(Stop)
```

## 表格


| 姓名 | 性别 |
| ---- | ---- |
| 张三 | 男   |

## 项目符号

* 第一章
  * 第一节
    * 第一单元
      * 第一课
      * 第二课
    * 第二单元
  * 第二节
* 第二章

## 编号

1. 微软
2. IBM
3. SUN

## 分割线

---

写信了

---

## 添加脚注

仓储大叔[^1]

[^1]: 是一位架构师，开发了Lind.DDD框架，同时是多年微软MVP，在工作上帮到了很多开发人员。
    
## 任务列表

* [X]  .net
* [X]  java
* [X]  node.js

## 数学公式

```math
\oint_c x^3\,dx+4y^2\,dy

2=\left(
\frac
{\left(3-x\right)\times2} 
{3-x}
\right)
```

今天就说到这里吧，希望可以帮助到大家！

## 时序图

```mermaid
sequenceDiagram
    participant A as Alice
    participant B as Bob
    A->>B: 请求数据
    B->>A: 返回数据
```

## oauth2时序图

```mermaid
sequenceDiagram
    participant User as 用户
    participant Client as 客户端应用
    participant AuthorizationServer as 授权服务器
    participant ResourceServer as 资源服务器

    User-->Client: 启动授权流程
    Client-->AuthorizationServer: 发起授权请求
    AuthorizationServer-->User: 引导用户登录并同意授权
    User-->AuthorizationServer: 输入用户名和密码
    AuthorizationServer-->User: 验证用户身份
    User-->AuthorizationServer: 同意授权
    AuthorizationServer-->Client: 生成授权码
    Client-->AuthorizationServer: 使用授权码请求访问令牌
    AuthorizationServer-->Client: 发放访问令牌
    Client-->ResourceServer: 使用访问令牌请求受保护资源
    ResourceServer-->Client: 返回受保护资源

```
