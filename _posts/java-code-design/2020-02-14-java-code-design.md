---
title: java代码设计
permalink: /codedesign/java
tags: 代码设计 CodeDesign
pageview: true
show_date: true
sidebar:
  nav: docs-en-code
---
## 数据库设计
使用tk.mybits
使用example 代替mapper.xml条件查询
使用范型 实现通用sql操作

## Services设计
service 在Controller 的调用用 private 修饰
```java
    @Autowired
    private UserService userService;
```

## @Autowired的使用:推荐对构造函数进行注释

建议后写成下面的样子：
```java
private final EnterpriseDbService service;

@Autowired
public EnterpriseDbController(EnterpriseDbService service) {
   this.service = service;
}
```

### 切面编程
**使用场景** 将日志记录，性能统计，安全控制，事务处理，异常处理等代码从业务逻辑代码中划分出来
**代码实现** 1. 用自定义注解的方式实现
