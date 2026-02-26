##### MVC

网站启动---Global.asax---Application.Start---完成路由初始化---定义了正则表达式规则

接收请求---路由匹配---控制器、Action

##### MapRoute

把Url规则包装成一个Route对象，放入Route集合（name不能相同）

##### HttpApplication

完成Http请求,定义一系列事件

##### HttpModule

**MVC（UrlRoutingModule）**

在 **PostResolveRequestCache** 事件中注册处理请求操作，匹配路由



##### 