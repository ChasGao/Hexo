---
title: JSP文件与HTML文件的注释
date: 2016-12-16 17:44:05
tags: 
	- JSP
type: "tags"
categories: 
	- JSP
---
JSP注释格式是`<%-- 注释内容 --%>`，区别于HTML注释格式`<!-- 注释内容 -->`;HTML的注释可以通过浏览器查看源码，JSP的注释内容无法通过源码查看到，所以JSP的注释不会被发送到客户端。
- - -
JSP页面中JSP脚本放在<%和%>之间，可以包含任何可执行的Java代码。
- - -
JSP输出表达式 格式：<%=表达式%> 可以输出变量值和方法返回值。
- - -
JSP3个编译指令 <%@ page 属性名="属性值"...%> 、<%@ include 属性名="属性值"...%>、 <%@ taglib 属性名="属性值"...%>。
- - -
JSP的7个动作指令：forward指令`<jsp:forward page="{relativeURL|<%=expression%>}">{<jsp:param.../>}</jsp:forward>`
	include指令`<jsp:include page="{relativeURLū <%=expression%>}" flush="true"><jsp:param name="parameterName" value="patameterValue"/></jsp:include>`
	useBean、 setProperty、 getProperty 指令使用较少。
- - -	
JSP脚本中的9个内置对象:输入输出对象:request,response,out;错误处理对象:exception;作用域控制对象:page,request,session,application;Servlet相关对象:config,pageContext。
