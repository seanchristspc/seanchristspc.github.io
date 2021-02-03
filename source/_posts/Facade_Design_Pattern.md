---
title: Facade模式
date: 2018/02/08 20:34:56
comments: true
description: 本文简要讲述了设计模式中的Facade模式。
tags:
	- Facade
categories:
	- DesignPattern
---

#Facade模式
简单窗口
&ensp; 使用Facade模式可以为相互关联在一起的错中复杂的类整理出高层的接口。其中的Facade角色可以让系统对外只有一个简单的接口。
***
## 演示程序类图
![Facade模式](Facade_Design_Pattern/Facade_Design_Pattern.png)

***
## 代码
### Database类
``` java
package com.sean.Facade.pagemaker;

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.IOException;
import java.util.Properties;

public class Database {
	private Database(){
		//防止外部new出Database对象，所以声明为private
	}
	public static Properties getProperties(String dbname){
		String filename=dbname+".txt";
		Properties prop=new Properties();
		try {
			prop.load(new FileInputStream("/home/sean/Documents/"+filename));
		} catch (FileNotFoundException e) {
			
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		return prop;
	}
}

```
### HtmlWriter类
``` java
package com.sean.Facade.pagemaker;

import java.io.IOException;
import java.io.Writer;

public class HtmlWriter {
	private Writer writer;
	public HtmlWriter(Writer writer){
		this.writer=writer;
	}
	public void title(String title){
		//输出标题
		try {
			writer.write("<html>");
			writer.write("<head>");
			writer.write("<title>"+title+"</title>");
			writer.write("</head>");
			writer.write("<body>\n");
			writer.write("<h1>"+title+"</h1>");
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	public void paragraph(String msg){
		//输出段落
		try {
			writer.write("<p>"+msg+"</p>");
		} catch (IOException e) {
			e.printStackTrace();
		}
	}
	public void link(String href,String caption){
		//输出超链接
		paragraph("<a href=\"" +href +"\">"+caption+"</a>");
	}
	public void mailto(String mailaddr,String username){
		//输出邮件地址
		link("mailto:"+mailaddr,username);
	}
	public void close(){
		//结束输出html
		try {
			writer.write("</body>");
			writer.write("</html>\n");
			writer.close();
		} catch (IOException e) {
			
			e.printStackTrace();
		}
		
	}
}

```

### PageMaker类
``` java
package com.sean.Facade.pagemaker;

import java.io.FileWriter;
import java.io.IOException;
import java.util.Properties;

public class PageMaker {
	private PageMaker(){
		//防止外部new出PageMaker实例，所以声明为private方法
	}
	public static void makeWelcomePage(String mailaddr,String filename){
		try {
			Properties mailprop=Database.getProperties("maildata");
			String username=mailprop.getProperty(mailaddr);
			HtmlWriter writer=new HtmlWriter(new FileWriter("/home/sean/Documents/"+filename));
			writer.title("Welcome to "+username+"'s page!");
			writer.paragraph(username+"欢迎到来"+username+"的主页。");
			writer.paragraph("等你的邮件喔！");
			writer.mailto(mailaddr, username);
			writer.close();
			System.out.println(filename+"is created for "+mailaddr +" ("+username+")");
		} catch (IOException e) {
			e.printStackTrace();
		}
	}

}

```

### Main类
``` java
package com.sean.Facade;

import com.sean.Facade.pagemaker.PageMaker;

public class Main {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		PageMaker.makeWelcomePage("seanchristspc@gmail.com", "welcome.html");

	}

}
```

***
## 要点
### Facade(窗口)
Facade角色向系统外部提供高层接口。在实例程序中由**PageMaker**扮演此角色。

### 构成系统的许多其他角色
这些角色各自完成自己的工作，他们并不知道Facade角色。Facade角色调用其他角色进行工作，但是其他角色不会调用Facade角色。代码中 **Database**和**HtmlWriter**扮演其他角色。

### Client
Client角色负责调用Facade角色
***
## 个人理解

Facade模式就是把复杂的系统变**看起来**简单。所谓看起来简单就是指在编写代码的时候还是挺复杂的，只是在使用某个功能是对外的接口比较少而且明确。
该模式还是挺好理解的，代码也不复杂。

参照
> 图解设计模式


写这个只是为了加深自己对设计模式的理解，如不明白，可以看 《图解设计模式》。
程序类图使用idea 生成的
