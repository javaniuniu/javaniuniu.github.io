---
title: FilterRegistrationBean类方法说明
permalink: /Filter/FilterRegistrationBean/explanation
tags: Filter拦截器 类解释
pageview: true
show_date: true
sidebar:
  nav: docs-en-code
---
[FilterRegistrationBean示例](/Filter/FilterRegistrationBean/example)

#### 构造函数和描述
FilterRegistrationBean()    
创建一个新的FilterRegistrationBean实例.

FilterRegistrationBean(Filter filter, ServletRegistrationBean... servletRegistrationBeans)      
创建一个新的FilterRegistrationBean实例，以将其注册到指定的ServletRegistrationBean .

#### Methods

| 修饰符和类型|	方法和说明|描述|
| :-----| :---- | :---- |
| void| 	addServletNames(String... servletNames)|为过滤器添加servlet名称.|
| void| 	addServletRegistrationBeans(ServletRegistrationBean... servletRegistrationBeans)|为过滤器添加ServletRegistrationBean .|
| void	| addUrlPatterns(String... urlPatterns)|添加将向其注册过滤器的URL模式.
| protected void	| configure(FilterRegistration.Dynamic registration)|配置注册设置.|
| protected Filter| 	getFilter()|返回正在注册的过滤器.|
| Collection<String>| 	getServletNames()|返回一个可变的servlet名称集合，将向其注册过滤器.|
| Collection<ServletRegistrationBean>	| getServletRegistrationBeans()|返回将注册过滤器的ServletRegistrationBean的可变集合.|
| Collection<String>| 	getUrlPatterns()|返回一个可变的URL模式集合，过滤器将针对该URL模式进行注册.|
| boolean| 	isMatchAfter()|返回是否应在ServletContext的任何声明的Filter映射之后匹配过滤器映射.|
| void| 	onStartup(ServletContext servletContext)|使用初始化所需的所有Servlet，过滤器，侦听器上下文参数和属性来配置给定的ServletContext .|
| void| 	setDispatcherTypes(DispatcherType first, DispatcherType... rest)|使用指定元素set dispatcher types便捷方法.|
| void| 	setDispatcherTypes(EnumSet<DispatcherType> dispatcherTypes)|设置应与注册一起使用的调度程序类型.|
| void| 	setFilter(Filter filter)|设置要注册的过滤器.|
| void| 	setMatchAfter(boolean matchAfter)|设置是否在ServletContext的任何声明的过滤器映射之后匹配过滤器映射.|
| void| 	setServletNames(Collection<String> servletNames)|设置将向其注册过滤器的servlet名称.|
| void| 	setServletRegistrationBeans(Collection<? extends ServletRegistrationBean> servletRegistrationBeans)|设置ServletRegistrationBean ，将向其注册过滤器.|
| void| 	setUrlPatterns(Collection<String> urlPatterns)|设置注册过滤器的URL格式.|
