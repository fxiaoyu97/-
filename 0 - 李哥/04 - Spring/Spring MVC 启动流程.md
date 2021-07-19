# Spring MVC 启动流程

在 `web.xml` 文件中给 Spring MVC 的 Servlet 配置了 `load-on-startup`，所以程序启动的时候会初始化 Spring MVC，在 `HttpServletBean` 中将配置的 `contextConfigLocation` 属性设置到 Servlet 中，然后在 `FrameworkServlet` 中创建了 `WebApplicationContext`，`DispatcherServlet` 根据 `contextConfigLocation` 配置的 `classpath` 下的 xml 文件初始化了 Spring MVC 总的组件。