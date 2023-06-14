# Tomcat 总体架构

![20230612162902](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230612162902.png)

如上图所示，Tomcat分为连接器和容器两部分

## 连接器

主要功能：

- 网络通信
  - 监听网络端口
  - 接收请求
- 应用层协议解析
  - 读取字节流，根据具体协议(HTTP)解析，生成`TomcatRequest`对象
- `TomcatRequest/Response`与`ServletRequest/Response`对象之间的转换

上述3个主要功能对应3个组件：

- `EndPoint`: 网络通信，提供字节流；
- `Processor`: 应用层协议解析，生成`TomcatRequest`对象；
- `Adapter`: `TomcatRequest/Response`与`ServletRequest/Response`对象之间的转换

> 其中`Adapter`运用到[适配器模式](../java/%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md#适配器模式)