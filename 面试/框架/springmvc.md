## springmvc执行流程：

<img src="../../../Library/Application Support/typora-user-images/image-20210823140926683.png" alt="image-20210823140926683" style="zoom:50%;" />

处理器映射器：通过url映射对于的controller。处理器映射器 HandlerMapping 根据配置找到相应的 Handler

处理器适配器：对相应的controller进行适配。

- 客户端（浏览器）发送请求，直接请求到 DispatcherServlet。
- DispatcherServlet 根据请求信息调用 HandlerMapping，解析请求对应的 Handler。
- 解析到对应的 Handler（也就是我们平常说的 Controller 控制器）后，开始由 HandlerAdapter 适配器处理。
- HandlerAdapter 会根据 Handler来调用真正的处理器开处理请求，并处理相应的业务逻辑。
- 处理器处理完业务后，会返回一个 ModelAndView 对象，Model 是返回的数据对象，View 是个逻辑上的 View。
- ViewResolver 会根据逻辑 View 查找实际的 View。
- DispaterServlet 把返回的 Model 传给 View（视图渲染）。
- 把 View 返回给请求者（浏览器）