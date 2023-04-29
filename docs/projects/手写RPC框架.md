- [基本概念](#基本概念)
  - [如何设计RPC](#如何设计rpc)
  - [RPC协议](#rpc协议)

根据Java Guide的《从零开始手把手教你实现一个简单的RPC框架》记录的手写RPC框架

# 基本概念

RPC：Remote Procedure Call，即远程过程调用，通过RPC我们可以远程调用服务器上某个服务的方法，RPC可以帮助这一过程简化为本地方法调用的形式，不用考虑底层网络编程TCP/UDP等细节。

![20230425220954](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230425220954.png)

根据上图:

- 首先是客户端(client functions)本地通过RPC框架调用方法函数
- 客户端桩(client stub)其实是个代理类，将客户端调用的方法、参数等封装和序列化成网络传输的消息体，传递给服务器端
- 服务器端的server stub类似Dispatcher，将收到的消息体反序列化，找到对应的服务方法(server funcitons)，执行并返回结果。

## 如何设计RPC

RPC的流程图可以简化如下：

![20230425222228](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230425222228.png)

可以看出首先是服务端向注册中心注册服务，注册中心给客户端提供了**发现服务**的功能，然后客户端可以通过注册中心返回的服务信息，来调用服务端的服务方法。

## RPC协议

包含的内容有：

- 魔数：用来标识一个协议包，通常是4个字节；
- 序列化算法：标明了实际数据的序列化方式，通常是1个字节；
- 消息体长度
