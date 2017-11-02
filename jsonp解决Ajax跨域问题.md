这几天用node做了个小项目，涉及到了在用Ajax发送请求时，想要调一些开源的api，遇到了跨域问题，用Ajax请求不能发送违反同源策略的请求（也就是跨域问题）。那么下面就说一下什么是同源策略以及怎么解决这个问题。首先我们知道因为浏览器的限制，所以Ajax请求不能发送违反同源策略的请求，那么什么是同源策略，同源策略就是要求我们发送的请求的协议，域名，端口号必须完全一致，那比如我们现在有一个index.html，在页面上有一个按钮用来发送Ajax请求，那么我的请求路径是localhost:3000/testAjax,然后我启动服务器以后用127.0.0.1:3000/index.html来访问，然后点击按钮发送Ajax请求，那么这个时候就会请求失败，其实请求已经发到了服务器，而且服务器也给我们做出了响应，但是因为浏览器的限制，我们拿不到服务器返回的数据，所以这个时候就会请求失败，而且还会报错。那么这个时候就是出现了跨域问题，这就像我们去调用一些api一样，我们本地的访问路径肯定不会和人家的请求路径一样，所以我们需要解决这个问题，那么就要用到一个东西叫做jsonp,大家在网上就可以搜到，而且人家解释的肯定比我好，我就是按照我的理解说一下。
jsonp呢是一个非官方的跨域解决方案，它是完全凭借程序员的聪明才智开发出来的，这些程序员真是太厉害啦，佩服，而我只是个菜鸟，呜呜呜。。不过我会好好学习，争取让自己变得也厉害一点，哈哈 ，闲话不多说，还是说我们的正题。那其实我们的网页它是有一些标签天生就具有跨域能力，比如img、link、script、iframe，那jsonp就是利用script标签的跨域能力来发送请求的。下面我们来说两种方式，一种是原生js的方式，一种是jQuery的方式。
1、JS方式：首先我们现在假设有一个路由：

    router.get("/testAjax" , function (req , res) {
	console.log("收到请求");
	var callback = req.query.callback;
	var obj = {
		name:"孙悟空",
		age:18,
		gender:"男",
		address:"花果山"
	}

	res.send(callback+"("+JSON.stringify(obj)+")");
    });
我们要返回给客户端这个obj对象，我们的页面：

     var btn01 = document.getElementById("btn01");
	 btn01.onclick = function () {
         //动态的创建一个script标签
		var script = document.createElement("script");
		//设置script的src
		script.src = "http://localhost:3000/testAjax?callback=fn";
		//将script添加到body中
		document.body.appendChild(script);
        }
       function fn(data) {
	      alert(data.name);
	 }
这样我们就可以访问到了，下面我们解释一下每一行代码的意思，因为jsonp利用的是script标签的跨域能力来发送请求，那么我们就需要一个script标签，我们可以在页面上写死这个标签，但是因为我们要控制发送请求的时机所以我们动态生成这个标签，那么就像我们的页面写的一样，在点击按钮发送请求的时候先创建一个script标签，那它的src就是我们要请求的路径了，它带了一个参数callback回调函数，这个函数叫fn，我们为什么要这么指定这个参数呢？因为后台路由的res.send是可以直接执行前端的方法，比如res.send('console,log(hello jsonp)'),比如res.send('alert(hello jsonp)')这些都是可以在前端控制台执行或是前端页面弹出的，所以我们想要拿到后台返回的数据，我们就想这个办法，在页面中定义一个函数，然后让后端调用这个函数并且传入参数，那我们前端就可以拿到数据了对不对。那么后端就要接收这个callback参数，并把它调用了，而且传参，这样就可以了。那路由中调用时为什么要JSON.stringify(obj)转化呢？因为后端返回的对象默认都转化为json对象，而我们现在是在拼串，所以需要把json对象手动转化为字符串。大家懂了吗？可能我说的有些繁琐但是很详细哦，每一步都提到了，嘿嘿，大家要耐心的看完哦！值得注意的是这个fn函数不可以写在window.onload里，它必须是一个全局函数。
下面我们来说jQuery的方式：
2、jQuery方式：

         $("#btn01").click(function () {
		//$.getJSON() 专门用来发送JSONP的请求
		$.getJSON("http://localhost:3000/testAJAX6?callback=?",function (data) {
			alert(data.name)
		});

	});
我们的路由还是那个路由，大家明显可以看到，我们jQuery简单多了是不是，它只需要调用一个getJSON()，串几个参数就ok了，剩下的都底层帮我们弄好了，那么getJSON()有三个参数：

    url：请求路径
    data:请求需要的参数，当然我们这里没有参数就没有传
    callback:回调函数，它的形参data就是我们服务器返回的数据，直接用就ok了。路径中的callback参数的值是？就可以了，因为jQuery已经帮我们处理啦。

现在我们就可以放心调用别人的api啦~~~

我是前端小嵘emily，大家多多指教哦！