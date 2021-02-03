---
title: Proxy模式
date: 2018/02/13 20:39:35
comments: true
description: 本文简要讲述了设计模式中的Proxy模式。
tags:
	- Proxy
categories:
	- DesignPattern
---

# Proxy模式

**只在必要时生成实例**

---

## 类表

| 名字 | 说明 |
|--------|--------|
| Printer | 表示带名字的打印机类(本人) |
| Printable | Printer和PrinterProxy的共同接口 |
| PrinterProxy | 表示带名字的打印机类(代理人) |
| Main | 测试程序行为的类 |


## 类图

![Proxy](Proxy_Design_Pattern/Proxy_Design_Pattern.png)

## 代码
### Printable接口
``` java
package com.sean.Proxy;

public interface Printable {

	public abstract void setPrinterName(String name);
	public abstract String getPrintName();
	public abstract void print(String string);
}

```
### Printer
``` java
package com.sean.Proxy;

public class Printer implements Printable {
	private String name;
	public Printer(){
		
	}
	public Printer(String name){
		this.name=name;
		heavyJob("正在生成Printer实例（"+name+")");
	}
	public void setPrinterName(String name) {
		this.name=name;

	}

	public String getPrintName() {
		
		return name;
	}

	public void print(String string) {
		System.out.println("===="+name+"====");
		System.out.println(string);

	}
	public void heavyJob(String msg){
		System.out.print(msg);
		for(int i=0;i<5;i++){
			try{
				Thread.sleep(1000);
			}catch(InterruptedException e){
				e.printStackTrace();
			}
			System.out.print(".");
		}
		System.out.println("结束。");
	}

}

```
### PrinterProxy
``` java
package com.sean.Proxy;

public class PrinterProxy implements Printable {
	private String name;
	private Printer real;
	public PrinterProxy(){
		
	}
	public PrinterProxy(String name){
		this.name=name;
	}
	public synchronized void setPrinterName(String name) {
		
			if(real!=null){
				real.setPrinterName(name);
			}
			this.name=name;
	}

	public String getPrintName() {
		
		return name;
	}

	public void print(String string) {
		realize();
		real.print(string);

	}
	public synchronized void realize(){
		if(real==null){
			real=new Printer(name);
		}
	}

}
```
### Main
``` java
package com.sean.Proxy;

public class Main {

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		Printable p=new PrinterProxy("Alice");
		System.out.println("现在的名字是"+p.getPrintName()+"。");
		p.setPrinterName("Bob");
		System.out.println("现在的名字是"+p.getPrintName()+"。");
		p.print("Hello,world!");

	}

}
```


## 参照
> 《图解设计模式》


写这个只是为了加深自己对设计模式的理解，如不明白，可以看 《图解设计模式》。
程序类图使用idea 生成的
