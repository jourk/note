---
title: Java-解析Java中文乱码
tags:
  - java
categories:
  - Notes
date: 2014-09-05 06:27:06
---
>有关乱码的处理－－中国程序员永远无法避免的话题
## 常见解决办法

对于Java由于默认的编码方式是 UNICODE,所以用中文也易出问题,常见的解决是
```java
String s2 = new String(s1.getBytes("ISO-8859-1"),"GBK");
```
## utf8解决JSP中文乱码问题

一般说来在每个页面的开始处，加入：
```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
     pageEncoding="UTF-8"%>

<%
 request.setCharacterEncoding("UTF-8");
%>
```
charset=UTF-8  的作用是指定JSP向客户端输出的编码方式为“UTF-8”
pageEncoding=”UTF-8″  为了让JSP引擎能正确地解码含有中文字符的JSP页面，这在LINUX中很有效
request.setCharacterEncoding(“UTF-8”); 是对请求进行了中文编码
有时，这样仍不能解决问题，还需要这样处理一下：
```java
String msg = request.getParameter("message");
String str=new String(msg.getBytes("ISO-8859-1"),"UTF-8");
out.println(st);
```
## Tomcat 5.5 中文乱码

### 1、修改配置文件

1)只要把%TOMCAT安装目录%webapps/servlets-examples/WEB-INF/classes/filters/SetCharacterEncodingFilter.class文件拷到你的webapp目录/filters下，如果没有filters目录，就创建一个。

2)在你的web.xml里加入如下几行：
```xml
<filter>
  <filter-name>Set   Character   Encoding</filter-name>
  <filter-class>filters.SetCharacterEncodingFilter</filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>GBK</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>Set   Character   Encoding</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```
3)完成.

### 2、get方式的解决办法

1) 打开tomcat的server.xml文件，找到区块，加入如下一行：
```xml
URIEncoding="GBK"
```
完整的应如下：
```xml
<Connector port="80"   maxThreads="150"   minSpareThreads="25"   maxSpareThreads="75"
enableLookups="false"   redirectPort="8443"   acceptCount="100"   debug="0"
onnectionTimeout="20000"  disableUploadTimeout="true"  URIEncoding="GBK" />
```
2) 重启tomcat,一切OK。
## xmlHttpRequest中文问题

页面jsp用的GBK编码

```java
<%@ page contentType="text/html; charset=GBK"%>
```
javascript部分

```javascript
function addFracasReport() {
     var url="controler?actionId=0_06_03_01&actionFlag=0010";
     var urlmsg="&reportId="+fracasReport1.textReportId.value;  //故障报告表编号
     var xmlHttp=Common.createXMLHttpRequest();
     xmlHttp.onreadystatechange = Common.getReadyStateHandler(xmlHttp, eval("turnAnalyPage"));
     xmlHttp.open("POST",url,true);
     xmlHttp.setRequestHeader( " Content-Type " , " application/x-www-form-urlencoded);
     xmlHttp.send(urlmsg);
 }
```
后台java中获得的reportId是乱码，不知道该怎么转，主要是不知道xmlHttp.send(urlmsg);以后是什么编码？在后面用java来转，试了几种，都没有成功，其中有：

代码
```java
public static String UTF_8ToGBK(String str) {
     try {
         return new String(str.getBytes("UTF-8"), "GBK");
     } catch (Exception ex) {
         return null;
     }
}

public static String UTF8ToGBK(String str) {
     try {
         return new String(str.getBytes("UTF-16BE"), "GBK");
     } catch (Exception ex) {
         return null;
     }
}

public static String GBK(String str) {
     try {
         return new String(str.getBytes("GBK"),"GBK");
     } catch (Exception ex) {
         return null;
     }
}
public static String getStr(String str) {
     try {
         String temp_p = str;
         String temp = new String(temp_p.getBytes("ISO8859_1"), "GBK");
         temp = sqlStrchop(temp);
         return temp;
     } catch (Exception e) {
         return null;
     }
}
```