---
title: JSP自定义标签
date: 2018/02/16 20:43:51
comments: true
description: 本文简要叙述了JSP自定义标签的实现。
tags:
	- tags
categories:
	- Java
	- JSP
---

# JSP自定义标签

自定义标签的实现，叫做标签处理器，而简单的标签处理器是指继承**SimpleTag** 实现的标签管理器。
简单标签管理器不在被jsp容器缓存。但这并不意味着简单标签处理器会比之前的慢。
> 初始化性能指标显示，缓存标签处理器并不能提供较好的性能优化，但缓存这些标签让实现标签变得更加困难，而且让这些标签带来更多的潜在错误。                    

**JSP规范的作者在JSP规范的7.15一节写到**


**SimpleTag**接口中用于标签触发的方法只有一个**doTag**，并且该方法只会执行一次。业务逻辑，遍历以及页面内容的操作都在这里实现。

`javax.servlet.jsp.targext`包中包含了一个SimpleTag的基础类
**SimpleTagSupport**提供了SimpleTag的所有方法的实现，并便于扩展简单标签处理器。
**SimpleTagSupport**类中用getJspContext方法返回JspContext实例，这个实例在JSP容器中用SimpleTag的setJspContext方法传入。
***

## SimpleTag示例

### 编写标签处理器

#### MyFirstTag类

``` java
package com.sean;

import java.io.IOException;

import javax.servlet.jsp.JspContext;
import javax.servlet.jsp.JspException;
import javax.servlet.jsp.tagext.JspFragment;
import javax.servlet.jsp.tagext.JspTag;
import javax.servlet.jsp.tagext.SimpleTag;

public class MyFirstTag implements SimpleTag{
	JspContext jspContext;
	@Override
	public void doTag() throws JspException, IOException {
		System.out.println("doTag");
		jspContext.getOut().print("This is my first tag.");
	}

	@Override
	public JspTag getParent() {
		System.out.println("getParent");
		return null;
	}

	@Override
	public void setJspBody(JspFragment body) {
		
		System.out.println("setJspBody");
	}

	@Override
	public void setJspContext(JspContext jspContext) {
		System.out.println("setJspContext");
		this.jspContext=jspContext;
		
	}

	@Override
	public void setParent(JspTag parent) {
		System.out.println("setParent");
		
	}

}

```

### 注册标签
#### mytags.tld

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee web-jsptaglibrary_2_1.xsd"
		version="2.1">

	<description>
		Simple tag examples
	</description>
	<tlib-version>1.0</tlib-version>
	<short-name>My First Taglib Example</short-name>
	<tag>
		<name>firstTag</name>
		<tag-class>com.sean.MyFirstTag</tag-class>
		<body-content>empty</body-content>
	</tag>	
</taglib>

```
name节点用于说明这个标签名称；tag-class则用于指出标签处理器的**完整类名**。
当然一个标签库中可以定义多个标签。

### 使用标签

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib uri="/WEB-INF/mytags.tld" prefix="easy" %>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>firstTagTest</title>
  <body>
   Hello!
   <br>
   <easy:firstTag></easy:firstTag>
  </body>
</html>
```
我的项目名 `customtag`
启动服务器，输入即可。
`http://localhost:8080/customtag/firstTagTest.jsp`

***

## 处理属性

实现SimpleTag接口或者扩展SimpleTagSupport的标签管理器都可以有属性。

###  DataFrmatterTag
继承SimpleTagSupport
``` java
package com.sean;

import java.io.IOException;
import java.util.StringTokenizer;

import javax.servlet.jsp.JspContext;
import javax.servlet.jsp.JspException;
import javax.servlet.jsp.JspWriter;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class DataFrmatterTag extends SimpleTagSupport {
	private String header;
	private String items;
	public String getHeader() {
		return header;
	}
	public void setHeader(String header) {
		this.header = header;
	}
	public String getItems() {
		return items;
	}
	public void setItems(String items) {
		this.items = items;
	}
	public void doTag() throws IOException,JspException{
		JspContext jspContext=getJspContext();
		JspWriter out=jspContext.getOut();
		
		out.print("<table style='border:1px solid green'>\n"
					+"<tr><td><span style='font-weight:bold'>"
					+header+"</span></td></tr>\n");
		StringTokenizer tokenizer=new StringTokenizer(items,",");
		while(tokenizer.hasMoreTokens()){
			String token=tokenizer.nextToken();
			out.print("<tr><td>"+token+"</td></tr>\n");
		}
		out.print("</table>");
	}
	
}

```
`doTag()` 首先通过`getJspContext()`获取通过JSP容器传入的 `JspContext`对象
`JspContext jspContext=getJspContext();`
接着，通过`JspContext`实例的`getOut()`获取`JspWriter`对象
`JspWriter out=jspContext.getOut();`
然后，`doTag()`方法使用`StringTokenizer`解析`items`属性值。

###注册DataFrmatter标签

``` xml
	<tag>
	<name>dataFormatter</name>
	<tag-class>com.sean.DataFrmatterTag</tag-class>
	<body-content>empty</body-content>
	<attribute>
		<name>header</name>
		<required>true</required>
	</attribute>
	
	<attribute>
		<name>items</name>
		<required>true</required>
	</attribute>
	</tag>
```

### dataFormatterTagTest.jsp
``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib uri="/WEB-INF/mytags.tld" prefix="easy" %>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    
    <title>My JSP 'dataFormatterTagTest.jsp' starting page</title>
	<title>Testing DataFormatterTag</title>
  </head>
  <body>
    <easy:dataFormatter items="Alabama,Alaska,Georgia,Florida" header="states"/>
    <br>
    <easy:dataFormatter header="Countries">
    	<jsp:attribute name="items">
    	US,UK,Canada,Korea
    	</jsp:attribute>
    </easy:dataFormatter>
  </body>
</html>
```
## 访问标签内容
### SelectElementTag类
``` java
package com.sean;

import java.io.IOException;

import javax.servlet.jsp.JspContext;
import javax.servlet.jsp.JspException;
import javax.servlet.jsp.JspWriter;
import javax.servlet.jsp.tagext.SimpleTagSupport;

public class SelectElementTag extends SimpleTagSupport {
	private String[] countries ={"China","Brazil","American","Japan"};
	public void doTag() throws IOException,JspException{
		JspContext jspContext=getJspContext();
		JspWriter out=jspContext.getOut();
		out.print("<select>\n");
		for(int i=0;i<countries.length;i++){
			getJspContext().setAttribute("value", countries[i]);
			getJspContext().setAttribute("text", countries[i]);
			getJspBody().invoke(null);
		}
		out.print("</select>\n");
	}
}
```

### 注册selectElementTag

``` xml
        <tag>
		<name>select</name>
		<tag-class>com.sean.SelectElementTag</tag-class>
		<body-content>scriptless</body-content>
	</tag>
```

### selectElementTagTest.jsp

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib uri="/WEB-INF/mytags.tld" prefix="easy" %>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">
    <title>My JSP 'selectElementTagTest.jsp' starting page</title>
  </head>
  
  <body>
    <easy:select>
    	<option value="${value}">${text}</option>
    </easy:select>
  </body>
</html>

```
我的项目名 `customtag`
启动服务器，输入即可。
`http://localhost:8080/customtag/selectElementTagTest.jsp`

## 编写EL函数

编写EL函数步骤
1.创建一个包含**静态方法**的public类。每个类的静态方法表示一个EL函数。**这个类可以不需要实现任何借口或者继承特定的类。可以像发布其他任何类一样发布这个类**
2.用function节点在标签库描述器注册函数


| 节点 | 说明 |
|--------|--------|
| description | 可选，标签说明 |
| display-name | 在xml工具中显示缩写名字 |
| icon | 可选，在xml工具中使用icon节点 |
| name | 函数的唯一名字 |
| function-class | 该函数对应实现的java类的全名 |
| function-signature | 该函数对应实现的java静态方法 |
| example | 可选，使用该函数的实例说明 |
| function-extension | 可以是一个或者多个节点 ，在xml工具中使用，用于提供该函数更多细节 |

使用该函数，须将`taglib`指令中的`uri`属性指向标签库描述，并指明使用的前缀。然后在JSP页面使用如下语法：
`${prefix:functionName(parameterList)}`

### MyFunction 

``` java
package com.sean.function;

public class MyFunction {
	public static String reverseString(String s){
		//字符串的反转
		return new StringBuffer(s).reverse().toString();
	}
}
```

### functiontags.tld
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<taglib xmlns="http://java.sun.com/xml/ns/j2ee"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee web-jsptaglibrary_2_1.xsd"
		version="2.1">
	<description>Function tag example</description>		
	<tlib-version>1.0</tlib-version>
	<uri>http://example.com/taglib/function</uri>
	<function>
		<description>Reverse a String</description>
		<name>reverseString</name>
		<function-class>com.sean.function.MyFunction</function-class>
		<function-signature>java.lang.String reverseString(java.lang.String)</function-signature>
	</function>
</taglib>
```

### 使用El

``` jsp
<%@ page language="java" import="java.util.*" pageEncoding="UTF-8"%>
<%@ taglib uri="http://example.com/taglib/function" prefix="f" %>
<%
String path = request.getContextPath();
String basePath = request.getScheme()+"://"+request.getServerName()+":"+request.getServerPort()+path+"/";
%>

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN">
<html>
  <head>
    <base href="<%=basePath%>">    
    <title>My JSP 'functionTagTest.jsp' starting page</title>
  </head>
  <body>
   ${f:reverseString("Hello, World") }
   <br>
   ${f:reverseString("Welcome")}
  </body>
</html>
```

## 发布自定义标签

在functiontags.tld增加`uri`节点
`<uri>http://example.com/taglib/function</uri>`

在jsp页面添加
`<%@ taglib uri="http://example.com/taglib/function" prefix="f" %>`

## 总结
自定义标签解决JavaBean中前端展现与后端逻辑分离的好办法。
