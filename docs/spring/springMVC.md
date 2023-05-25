- [基础概念](#基础概念)
  - [核心组件](#核心组件)
  - [工作流程](#工作流程)
- [开发细节](#开发细节)
  - [RequestMapping注解](#requestmapping注解)
    - [支持路径占位符](#支持路径占位符)
  - [获取请求参数](#获取请求参数)
    - [通过`ServletAPI`获取](#通过servletapi获取)
    - [通过控制器形参获取](#通过控制器形参获取)
    - [处理乱码](#处理乱码)
  - [数据共享](#数据共享)
    - [request域内共享](#request域内共享)
    - [session \& application 共享 (略)](#session--application-共享-略)
- [视图](#视图)
  - [转发视图(InternalResourceView)](#转发视图internalresourceview)
  - [重定向视图(RedirectView)](#重定向视图redirectview)
  - [重定向和转发的区别](#重定向和转发的区别)
  - [视图控制器](#视图控制器)
- [RESTful](#restful)
  - [HiddenHttpMethodFilter](#hiddenhttpmethodfilter)

# 基础概念

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

# 开发细节

## RequestMapping注解

### 支持路径占位符

```java
@RequestMapping("/test/{id}/{user}")
public String test(@PathVariable("id") String id, @PathVariable("user") String user) {
    log.debug("id: {}, user: {}", id, user);
    return "success";
}
```

## 获取请求参数

假设前端发来的请求参数如下：

```html
<a th:href="@{/testParam(username='admin',password=123456)}">测试获取请求参数-->/testParam</a><br>
```

### 通过`ServletAPI`获取

```java
@RequestMapping("/testParam")
public String testParam(HttpServletRequest request) {
    String username = request.getParameter("username");
    String password = request.getParameter("password");
    log.debug("username: {}, password: {}", username, password);
    return "success";
}
```

### 通过控制器形参获取

控制器中形参和请求**参数名相同**，就可以获取到请求参数的值

```java
@RequestMapping("/testParam")
    public String test(String username, String password) {...}
```

当形参和请求参数名不同时，可用`@RequestParam`注解指定请求参数名

```java
@RequestMapping("/testParam")
public String test(@RequestParam("username") String user, @RequestParam("password") String pass) {...}
```

`@RequestParam`有三个属性：

- value: 请求参数名
- required: 该请求参数是否必须，默认为true
- defaultValue: 形参的默认值

`RequestHeader`和`CookieValue`与`RequestParam`有相同的属性，区别在于：

- `RequestParam`是为了获取请求参数的内容
- `RequestHeader`是为了获取请求头的内容
- `CookieValue`是为了获取Cookie的内容

### 处理乱码

SpringMVC提供了`CharacterEncodingFilter`过滤器，可以解决乱码问题，需要在`web.xml`中配置：

```xml  
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <!-- 设置编码格式 -->
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <!-- 强制请求和响应的编码一致 -->
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <!-- 对所有请求进行过滤 -->
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

## 数据共享

### request域内共享

有以下方法共享：

- `ServletAPI`

    ```java
    @RequestMapping("/testForward")
    public String testForward(HttpServletRequest request) {
        // 设置共享数据
        request.setAttribute("msg", "hello");
        // 转发到testParam请求
        return "testParam";
    }
    ```

- `ModelAndView`

    ```java
    @RequestMapping("/test")
    public ModelAndView testModelAndView() {
        ModelAndView modelAndView = new ModelAndView();
        // 添加共享数据
        modelAndView.addObject("message", "Hello Spring MVC");
        // 设置视图名称
        modelAndView.setViewName("success");
        // 返回ModelAndView对象，而不是String
        return modelAndView;
    }
    ```

- `Model`
- `Map`
- `ModelMap`

这三个其实是类似，来看下他们的关系图

![20230522142334](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230522142334.png)

详细分析放在后面阅读源码的时候记录。

### session & application 共享 (略)

# 视图

视图有很多种类，常用的有：

- 转发视图
- 重定向视图

SpringMVC中的视图接口是`View`接口，作用是渲染数据，将模型Model中的数据展示给用户。

## 转发视图(InternalResourceView)

控制器返回的视图名称带`forward:`前缀时，不会被视图解析器解析，而是创建一个`InternalResourceView`对象，该对象会将请求转发给指定的视图。

## 重定向视图(RedirectView)

控制器返回的视图名称带`redirect:`前缀时，不会被视图解析器解析，而是创建一个`RedirectView`对象，该对象会将请求转发给指定的视图。

## 重定向和转发的区别

- 转发：
  - 客户端只发一次请求(发给服务端)
  - 可以共享request域内的数据
  - 不能跨域访问(即只能在同一个web应用内访问)

- 重定向：
  - 客户端发两次请求(第一次发给服务端，第二发给服务端响应的重定向地址)
  - 不能共享request域内的数据
  - 可以跨域访问

## 视图控制器

如果Controller中的方法仅用于页面跳转，可以通过视图控制器的方法在`springmvc.xml`中进行配置

```xml
<mvc:view-controller path="/" view-name="index"/>
<!-- 开启mvc注解驱动 -->
<mvc:annotation-driven>
```

由于配置视图控制器会导致Controller中其他映射失效，所以还需要开启mvc注解驱动

# RESTful

## HiddenHttpMethodFilter

浏览器只能发送`GET`和`POST`方式的请求 