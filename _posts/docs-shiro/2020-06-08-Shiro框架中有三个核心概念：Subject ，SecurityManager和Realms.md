---
title: Shiro框架中有三个核心概念：Subject ，SecurityManager和Realms
permalink: /web/shiro/three-core
tags: shiro
key: web-shiro-three-core
---

#### 2.1.1    Subject
`Subject` 一词是一个安全术语，其基本意思是“当前的操作用户”。称之为“用户”并不准确，因为“用户”一词通常跟人相关。在安全领域，术语“Subject”可以是人，也可以是第三方进程、后台帐户（Daemon Account）、定时作业（Corn Job）或其他类似事物。它仅仅意味着“当前跟软件交互的东西”。但考虑到大多数目的和用途，你可以把它认为是 `Shiro` 的“用户”概念。
在程序中你都能轻易的获得 `Subject` ，允许在任何需要的地方进行安全操作。每个 `Subject` 对象都必须与一个 `SecurityManager` 进行绑定，你访问Subject对象其实都是在与 `SecurityManager` 里的特定 `Subject` 进行交互。



#### 22.1.2    SecurityManager
`Subject` 的“幕后”推手是 `SecurityManager` 。` Subject` 代表了当前用户的安全操作，`SecurityManager` 则管理所有用户的安全操作。它是Shiro框架的核心，充当“保护伞”，引用了多个内部嵌套安全组件，它们形成了对象图。但是，一旦 `SecurityManager` 及其内部对象图配置好，它就会退居幕后，应用开发人员几乎把他们的所有时间都花在`Subject API`调用上。
那么，如何设置`SecurityManager`呢？嗯，这要看应用的环境。例如，`Web`应用通常会在Web.xml中指定一个`Shiro Servlet Filter`，这会创建`SecurityManager`实例，如果你运行的是一个独立应用，你需要用其他配置方式，但有很多配置选项。
一个应用几乎总是只有一个 `SecurityManager` 实例。它实际是应用的`Singleton`（尽管不必是一个静态`Singleton`）。跟`Shiro`里的几乎所有组件一样，`SecurityManager` 的缺省实现是`POJO`，而且可用`POJO`兼容的任何配置机制进行配置 - 普通的`Java`代码、`Spring XML、YAML、.properties`和.ini文件等。基本来讲，能够实例化类和调用`JavaBean`兼容方法的任何配置形式都可使用。

#### 22.1.3    Realms
`Shiro`的第三个也是最后一个概念是`Realm`。`Realm`充当了`Shiro`与应用安全数据间的“桥梁”或者“连接器”。也就是说，当与像用户帐户这类安全相关数据进行交互，执行认证（登录）和授权（访问控制）时，`Shiro`会从应用配置的Realm中查找很多内容。
从这个意义上讲，Realm实质上是一个安全相关的DAO：它封装了数据源的连接细节，并在需要时将相关数据提供给`Shiro`。当配置`Shiro`时，你必须至少指定一个`Realm`，用于认证和（或）授权。配置多个`Realm`是可以的，但是至少需要一个。
`Shiro`内置了可以连接大量安全数据源（又名目录）的`Realm`，如LDAP、关系数据库（JDBC）、类似INI的文本配置资源以及属性文件 等。如果缺省的`Realm`不能满足需求，你还可以插入代表自定义数据源的自己的`Realm`实现。
象其他内部组件一样，由`SecurityManager`来管理如何使用`Realms`来获取安全的身份数据。
