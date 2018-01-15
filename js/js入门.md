js入门
=====
##### 什么是js
js是一个脚本语言，是一种轻量级的编程语言，可以插入html代码，由浏览器执行。

前端和客户端一样，都是用来展示界面的，从服务器中获取数据并更新网页。展示界面由html+css完成，这个类似于Android的xml布局，而js就相当于Android的java，从布局中获取控件（html的元素）并渲染数据。

和Android有一些区别，js代码可以嵌套在html代码中，但是必须要用script标签包含，也可以专门放在一个js文件中。如果放在html中通常的做法是把函数放入 head 部分中，或者放在页面底部。这样就可以把它们安置到同一处位置，不会干扰页面的内容。如果专门放在一个js文件中，要在html中引用。


##### js的使用：
1.DOM（文档对象模型 Document Object Model）：
 Android是获取布局中的空间渲染数据，而在前端中这个工作由DOM（Document Object Model）完成，它的作用是找到html中的元素并渲染数据，也可以改变html元素的样式（css），对DOM事件做出反应，添加和删除html元素。	

2.BOM（浏览器对象模型 Browser Object Model）
前端的工作既然是在浏览器中显示，那么和浏览器的交互是必要的，而BOM就是做这件事情的，比如说修改浏览器的窗口，获取用户屏幕的信息，当前网页的url，浏览器的信息，定时器，用户的cookies.....

3.AJAX（前端网络请求框架）
一般我们的数据都是从网络中获取的，和Android的网络请求不一样，个人感觉js的网络请求更加简洁（也可能是因为自己刚刚入门），下面是ajax的主要代码。

XMLHttpRequest 用于在后台与服务器交换数据。这意味着可以在不重新加载整个网页的情况下，对网页的某部分进行更新。


		//为了应对所有的现代浏览器，包括 IE5 和 IE6，请检查浏览器是否支持 XMLHttpRequest 对象。如果支持，则创建 XMLHttpRequest 对象。如果不支持，则创建 ActiveXObject ：
		var xmlhttp;
		if (window.XMLHttpRequest)
		  {// code for IE7+, Firefox, Chrome, Opera, Safari
		  xmlhttp=new XMLHttpRequest();
		  }
		else
		  {// code for IE6, IE5
		  xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
		  }
		//如需将请求发送到服务器，我们使用 XMLHttpRequest 对象的 open() 和 send() 方法：
		xmlhttp.open("GET","test1.txt",true);
		xmlhttp.send();

>open(method,url,async)   
>规定请求的类型、URL 以及是否异步处理请求。

>- method：请求的类型；GET 或 POST
>- url：文件在服务器上的位置
>- async：true（异步）或 false（同步）

>send(string)
>将请求发送到服务器。
>- string：仅用于 POST 请求

如果是post请求

	xmlhttp.open("POST","ajax_test.asp",true);
	//如果需要像 HTML 表单那样 POST 数据，请使用 setRequestHeader() 来添加 HTTP 头。然后在 send() 方法中规定您希望发送的数据：
	xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
	xmlhttp.send("fname=Bill&lname=Gates");






