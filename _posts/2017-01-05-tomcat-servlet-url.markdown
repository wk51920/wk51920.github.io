---
layout: post
title: tomcat容器中，Servlet地址匹配问题
date: 2017/01/05
---
# 问题描述
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;初次接触Servlet的同学可能会因为不清楚Servlet的URL匹配规则，而在访问Servlet地址时，出现`404 The requested resource is not available.`的问题。

# 环境介绍

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1. 容器选择使用tomcat 9.0;
<br/>
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2. 部署在webapps下的项目结构如图：
![项目结构图][1]

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**注意：在这里提醒一下初学者，`WebContent`文件夹即相当于tomcat中的`webapps` 文件夹。**

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 3. web.xml中servlet的配置如下：

{% highlight html linenos %}
 \<servlet\>
\<servlet-name\>Hello\</servlet-name\>
\<servlet-class\>servlets.BeerSelect\</servlet-class\>
  \</servlet\>
  
  \<servlet-mapping\>
\<servlet-name\>Hello\</servlet-name\>
\<url-pattern\>/SelectBeer.do\</url-pattern\>
  \</servlet-mapping\>
{% endhighlight %}

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 4. form.html页面内容如下：

{% highlight xml linenos %}
\<!DOCTYPE html\>
\<html\>
\<head\>
\<meta charset="UTF-8"\>
\<title\>Insert title here\</title\>
\<link rel="stylesheet" href="../css/form.css" /\>
\</head\>
\<body\>
   \<h1\>Beer Selection Page\</h1\>
   \<form action="SelectBeer.do" method="POST"\>
   Select beer characteristics.
   \<p\>
  Color:
  \<select name="color" size="1"\>
 \<option value="light"\>light\</option\>
 \<option value="amber"\>amber\</option\>
 \<option value="brown"\>brown\</option\>
 \<option value="dark"\>dark\</option\>
  \</select\>
   \</p\>
   \<input type="submit"/\>
   \</form\>
\</body\>
\</html\>
{% endhighlight %}
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; **如果按照上述配置去提交表单，必然会产生找不到资源的404错误。**

# 问题解答

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 1. web.xml中配置的用于外部访问的Servlet地址为`<url-pattern>/SelectBeer.do</url-pattern>` ，在地址前加入了斜线“/”，表示此地址是一种绝对地址，实际应该请求的地址是：`localhost:8888/你的项目名/SelectBeer.do`。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; 2. 我们在form.html中提交表单的地址为：`action="SelectBeer.do"`，这种写法是一种相对寻址的方式，相对于form.html所在的目录去生成地址，form.html在web目录下，而不在项目的根目录中，因此翻译成完整的地址为：`localhost:8888/你的项目名/web/SelectBeer.do`可以看到，此地址实际上与web.xml中配置的目标Servlet地址并不相同，**这就是造成404错误的根本原因**。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;3. 既然知道了是生成的URL不匹配，那么想办法让URL匹配即可。我们可以在表单中使用绝对寻址的方式使地址匹配，内容如下：

```
`action="/你的项目名/SelectBeer.do"
```
`&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;此地址翻译成完整地址为：`localhost:8888/你的项目名/SelectBeer.do\`，这与web.xml配置的Servlet地址相符，因此可以顺利访问到配置的Servlet。

[1]:	/assets/images/2017-01-05-tomcat-servlet-url/01.png