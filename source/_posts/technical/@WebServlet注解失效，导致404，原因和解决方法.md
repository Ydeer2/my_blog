---
cover: https://gcore.jsdelivr.net/gh/cdn-x/xaoxuu@main/posts/20250706150531375.jpg
title: "@WebServlet注解失效，导致404，原因和解决方法"
date: 2025-08-13 19:17:23
comments: true
categories:
  - 杂
tags:
  - bug解决
---

# @WebServlet注解失效，导致404，原因和解决方法

本人在使用IDEA写javaWeb项目的时候，路径没有出现错误，但发现当我使用@WebServlet时，前端无法访问到后端的资源，而当我不使用@WebServlet选择使用xml配置文件时，前端却又可以访问到资源。
注解编程

```
@WebServlet()
```

xml配置

```
	<servlet>
		<servlet-name></servlet-name>
		<servlet-class></servlet-class>
	</servlet>
	<servlet-mapping>
		<servlet-name></servlet-name>
		<url-pattern></url-pattern>
	</servlet-mapping>
```

最后发现web.xml文件中莫名开启了metadata-complete="true"

```
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
                      http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0"
         metadata-complete="true">
<!--metadata-complete="true"表示只支持配置文件，不支持注解，md害我找了怎么久-->
</web-app>
```

但是之前我在使用添加Web框架的时候不会出现这个情况。
通过回忆发现，是因为这次创建javaWeb模块的时候，我使用的是Maven Archetype创建Web项目。通过这种方式创建的Web.xml配置版本过低。会自动出现metadata-complete="true"。
![](https://cdn.jsdelivr.net/gh/Ydeer2/Images@main/sda.png)
如果先创建一个空项目，然后在手动添加框架支持就不会出现这种情况。
通过JBLJavaToWeb插件，直接生成也不会出现问题。

## 解决办法

将metadata-complete="true"改为false就可以了
