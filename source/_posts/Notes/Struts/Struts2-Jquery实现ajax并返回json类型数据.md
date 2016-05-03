---
title: Struts2-Jquery实现ajax并返回json类型数据
tags:
  - struts
categories:
  - Notes
date: 2014-07-03 10:27:06
---
概要：此篇文章来源于网络，正好工作上用到，所以记录下来，因为此篇文章在多家博客可以搜到，所以现在本人也弄不清原作者是谁，所以不注明出处了。
## 首先需要的包（struts核心包和[json需要的包）：
- Struts核心包：
![](http://7xssec.com2.z0.glb.clouddn.com/notes-struts-ajax-1.png)
- json需要的包：
![](http://7xssec.com2.z0.glb.clouddn.com/notes-struts-ajax-2.png)

&trade;commons-logging-*.jar在导入struts核心包的时候就导入了，所以导入json包的时候可以去掉这个包。

## 页面效果：
![](http://7xssec.com2.z0.glb.clouddn.com/notes-struts-ajax-3.png)

## json_demo.jsp页面
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Simpleton Demo | struts+ajax返回json类型数据</title>

<link rel="shortcut icon" type="image/x-icon" href="images/Icon.png" />
<link rel="stylesheet" type="text/css" href="styles/base.css" />

</head>
<body background="images/bg.gif">
    
    <div id="div_json">
        <h5>录入数据</h5>
        <br />
        <form action="#" method="post">
            <label for="name">姓名：</label><input type="text" name="name" />
            <label for="age">年龄：</label><input type="text" name="age" />
            <label for="position">职务：</label><input type="text" name="position" />
            <input type="button" class="btn" value="提交结果"/>
        </form>
        <br />
        <h5>显示结果</h5>
        <br />
        <ul>
            <li>姓名：<span id="s_name">赞无数据</span></li>
            <li class="li_layout">年龄：<span id="s_age">暂无数据</span></li>
            <li class="li_layout">职务：<span id="s_position">暂无数据</span></li>
        </ul>
    </div>
    
    <div id="authorgraph"><img alt="" src="images/autograph.gif"></div>
    
    <script type="text/javascript" src="scripts/jquery-1.8.2.js"></script>
    <script type="text/javascript">
        
        /* 提交结果，执行ajax */
        function btn(){
            
            var $btn = $("input.btn");//获取按钮元素
            //给按钮绑定点击事件
            $btn.bind("click",function(){
                
                $.ajax({
                    type:"post",
                    url:"excuteAjaxJsonAction",//需要用来处理ajax请求的action,excuteAjax为处理的方法名，JsonAction为action名
                    data:{//设置数据源
                        name:$("input[name=name]").val(),
                        age:$("input[name=age]").val(),
                        position:$("input[name=position]").val()//这里不要加","  不然会报错，而且根本不会提示错误地方
                    },
                    dataType:"json",//设置需要返回的数据类型
                    success:function(data){
                        var d = eval("("+data+")");//将数据转换成json类型，可以把data用alert()输出出来看看到底是什么样的结构
                        //得到的d是一个形如{"key":"value","key1":"value1"}的数据类型，然后取值出来
                        
                        $("#s_name").text(""+d.name+"");
                        $("#s_age").text(""+d.age+"");
                        $("#s_position").text(""+d.position+"");
                        
                    },
                    error:function(){
                        alert("系统异常，请稍后重试！");
                    }//这里不要加","
                });
            });
        }
    
        /* 页面加载完成，绑定事件 */
        $(document).ready(function(){           
            btn();//点击提交，执行ajax
        });
    </script>
</body>
</html>
```

## JsonAction.java代码
```java
package com.simpleton.demo.action;

import java.util.HashMap;
import java.util.Map;
import javax.servlet.http.HttpServletRequest;
import net.sf.json.JSONObject;
import org.apache.struts2.interceptor.ServletRequestAware;
import com.opensymphony.xwork2.ActionSupport;

public class JsonAction extends ActionSupport implements ServletRequestAware{
    private static final long serialVersionUID = 1L;
    
    private HttpServletRequest request;
    private String result;

    public void setServletRequest(HttpServletRequest arg0) {
        this.request = arg0;
    }
    public String getResult() {
        return result;
    }
    public void setResult(String result) {
        this.result = result;
    }
    
    /**
     * 处理ajax请求
     * @return SUCCESS
     */
    public String excuteAjax(){
        
        try {
            //获取数据
            String name = request.getParameter("name");
            int age = Integer.parseInt(request.getParameter("age")); 
            String position = request.getParameter("position");
            
            //将数据存储在map里，再转换成json类型数据，也可以自己手动构造json类型数据
            Map<String,Object> map = new HashMap<String,Object>();
            map.put("name", name);
            map.put("age",age);
            map.put("position", position);
            
            JSONObject json = JSONObject.fromObject(map);//将map对象转换成json类型数据
            result = json.toString();//给result赋值，传递给页面
        } catch (Exception e) {
            e.printStackTrace();
        }
        return SUCCESS;
    }
}
```

## struts.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE struts PUBLIC
    "-//Apache Software Foundation//DTD Struts Configuration 2.1.7//EN"
    "http://struts.apache.org/dtds/struts-2.0.dtd">
<struts>
    
    <!--解决乱码    -->
    <constant name="struts.i18n.encoding" value="UTF-8"></constant> 
    
    <package name="simpleton" extends="struts-default,json-default">
        
        <action name="*JsonAction" method="{1}" class="com.simpleton.demo.action.JsonAction">
            <result name="fail"></result>
            <!-- 返回json类型数据 -->
            <result type="json">
                <param name="root">result<!-- result是action中设置的变量名，也是页面需要返回的数据，该变量必须有setter和getter方法 --></param>
            </result>
        </action>
        
    </package>
    
</struts>
```

至此一个demo完成。