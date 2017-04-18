---
layout: post
title: 使用注解进行 Servlet 开发
categories: JavaWeb
description: 使用注解进行 Servlet 开发
keywords: java, web, javaweb, JavaWeb, servlet
---

Servlet 3.0 开始提供注解、异步调用、直接文件上传支持。之前一直是在 `web.xml` 中配置 `<servlet>` 进行开发，写文整理下。

# 使用 @WebServlet 代替 `web.xml

关于 `@WebServlet` 参数：

|属性名         |类型           |属性描述|
|---------------|:--------------|--------|
|name           |String         | 指定servlet的name属性,等价于<Servlet-name>.如果没有显示指定，则该servlet的取值即为类的全限定名.
|value          |String[]       |等价于urlPatterns,二者不能共存.
|urlPatterns    |String[]       |指定一组servlet的url的匹配模式,等价于<url-pattern>标签.
|loadOnStartup  |int            |指定servlet的加载顺序,等价于<load-on-startup>标签.
|initParams     |WebInitParam[] |指定一组初始化参数,等价于<init-param>标签.
|asyncSupported | boolean       |申明servlet是否支持异步操作模式,等价于<async-supported>标签.
|displayName    |String         |servlet的显示名,等价于<display-name>标签.
|description    |String         |servlet的描述信息,等价于<description>标签.

```java

@WebServlet(name = "MyServlet", value = {"/my/1"},
loadOnStartup = 1, //启动项
initParams = { @WebInitParam(name = "initParameter", value = "张三") })
public class MyServlet extends HttpServlet {
	private static final long serialVersionUID = 671597860608235344L;

	public ChangePinServlet() {
		super();
	}

	@Override
	protected void doGet(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		// doPost(request, response);
		String initParameter = getInitParameter("initParameter"); // 张三
	}

	@Override
	protected void doPost(HttpServletRequest request,
			HttpServletResponse response) throws ServletException, IOException {
		//
	}

	@Override
	public void init() throws ServletException {
		//
	}
}
```

只需在Servlet对应的类上使用@WebServlet进行标注，我们就可以访问到该 Servlet 了，而不需要再在 `web.xml` 文件中进行配置。@WebServlet的 `urlPatterns` 和 `value` 属性都可以用来表示 Servlet 的部署路径，它们都是对应的一个数组。


# 异步调用

对于一个 Servlet 如果要支持异步调用的话我们必须指定其 `asyncSupported`属性为 `true`（默认是 `false`）
```
@WebServlet(
asyncSupported=true, //支持异步调用
name = "MyServlet", value = {"/my/1"},
loadOnStartup = 1, //启动项
initParams = { @WebInitParam(name = "initParameter", value = "张三") })
```

实例如下：

```java
// in dpget/doPost method
final PrintWriter writer = resp.getWriter();
writer.println("异步之前输出的内容。");
writer.flush();
//开始异步调用，获取对应的AsyncContext。
final AsyncContext asyncContext = request.startAsync();
//设置超时时间，当超时之后程序会尝试重新执行异步任务，即我们新起的线程。
asyncContext.setTimeout(10*1000L);
//新起线程开始异步调用，start方法不是阻塞式的，它会新起一个线程来启动Runnable接口，之后主程序会继续执行
asyncContext.start(new Runnable() {

	@Override
	public void run() {
		try {
			Thread.sleep(5*1000L);
			writer.println("异步调用之后输出的内容。");
			writer.flush();
			//异步调用完成，如果异步调用完成后不调用complete()方法的话，异步调用的结果需要等到设置的超时
			//时间过了之后才能返回到客户端。
			asyncContext.complete();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

});
writer.println("可能在异步调用前输出，也可能在异步调用之后输出，因为异步调用会新起一个线程。");
writer.flush();
```