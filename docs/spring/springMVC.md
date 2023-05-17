## 核心组件

- `DispatcherServlet`: 前端控制器，负责接收请求，响应结果，相当于转发器，中央处理器
- `HandlerMapping`: 处理器映射器，负责根据请求找到Handler(Handler等于Controller)
- `HandlerAdapter`: 处理器适配器，负责根据Handler执行适当的方法
- `Handler`: 请求处理器，处理实际请求的处理器
- `ViewResolver`: 视图解析器，负责根据逻辑视图名解析成真正的视图View

## 工作流程

![20230515222702](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230515222702.png)

1. 客户端发送Http request被`DispatcherServlet`拦截；
2. `HandlerMapping`根据被拦截的URI找到对应的`Handler`，并将涉及的`Handler`及其拦截器一并返回给`DispatcherServlet`；
3. `DispatcherServlet`根据`Handler`找到对应的`HandlerAdapter`，并将`request`、`response`、`Handler`等传给`HandlerAdapter`，由`HandlerAdapter`执行`Handler`；
4. `Handler`执行完后会返回一个`ModelAndView`对象，其中`Model`代表数据对象，`View`代表视图对象；
5. `ViewResolver`根据`View`找到对应的视图对象，如.html、.jsp等；
6. `DispatcherServlet`将`Model`数据对象发送给对应的视图对象渲染，最后将渲染结果返回给客户端。