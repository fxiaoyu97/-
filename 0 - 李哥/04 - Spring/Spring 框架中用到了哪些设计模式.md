# Spring 框架中用到了哪些设计模式

- 代理模式：在 AOP 和 Remoting 中被用的比较多。
- 单例模式：在 Spring 配置文件中定义的 Bean 默认为单例模式。
- 模板方法：用来解决代码重复的问题。比如. RestTemplate, JmsTemplate, JpaTemplate。
- 前端控制器：Spring 提供了 DispatcherServlet 来对请求进行分发。
- 视图帮助(View Helper )：Spring 提供了一系列的 JSP 标签，高效宏来辅助将分散的代码整合在视图里。
- 依赖注入：贯穿于 BeanFactory / ApplicationContext 接口的核心理念。
- 工厂模式：BeanFactory 用来创建对象的实例。