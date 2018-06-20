# SpringMVC

SpringMVC基于模型-视图-控制器(Model-View-Controller，MVC)模式实现

- DispatcherServlet

  请求离开浏览器后，首先到达Spring的DispatcherServlet，DispatcherServlet是SpringMVC的前端控制器，它的任务是将请求发送给SpringMVC控制器(Controller)

- HandlerMapping

  应用程序中会有多个控制器，DispatcherServlet需要知道将请求发送给哪个控制器，因此DispatcherServlet会查询处理映射器(HandlerMapping)来确定请求发送的下一站，处理映射器会根据请求所携带的URL信息类进行决策。

