- [注解和枚举](#注解和枚举)
  - [注解](#注解)
    - [常用注解](#常用注解)
  - [lombok](#lombok)
    - [@Builder](#builder)
  - [枚举](#枚举)
- [泛型](#泛型)
  - [泛型类型](#泛型类型)
  - [泛型擦除](#泛型擦除)
- [IO](#io)
  - [序列化](#序列化)
  - [字节流](#字节流)
  - [字符流](#字符流)
  - [缓冲流](#缓冲流)
  - [Kryo](#kryo)
  - [NIO](#nio)
    - [三大组件](#三大组件)
    - [文件编程](#文件编程)
  - [BIO](#bio)
  - [AIO](#aio)
- [反射](#反射)
  - [获取Class的方式](#获取class的方式)
  - [反射的作用](#反射的作用)
  - [动态代理](#动态代理)
    - [JDK代理](#jdk代理)
    - [cglib代理](#cglib代理)
- [网络编程](#网络编程)
- [多线程](#多线程)
  - [Unsafe类](#unsafe类)
    - [内存操作](#内存操作)
    - [内存屏障(待完善)](#内存屏障待完善)
  - [线程的生命周期](#线程的生命周期)
  - [对象头](#对象头)
    - [mark word(待完善)](#mark-word待完善)
    - [重量级锁](#重量级锁)
    - [轻量级锁](#轻量级锁)
    - [偏向锁](#偏向锁)
    - [无锁](#无锁)
  - [线程池（待完善）](#线程池待完善)
    - [Executors](#executors)
    - [ExecutorService](#executorservice)
  - [线程组](#线程组)
  - [线程通信模式](#线程通信模式)
    - [happens-before](#happens-before)
    - [线程共享](#线程共享)
      - [volatile（待完善）](#volatile待完善)
  - [线程同步](#线程同步)
    - [互斥锁](#互斥锁)
      - [synchronized（待完善）](#synchronized待完善)
      - [ReentrantLock（待完善）](#reentrantlock待完善)
      - [condition（待完善）](#condition待完善)
      - [ReadWriteLock（待完善）](#readwritelock待完善)
      - [StampedLock（待完善）](#stampedlock待完善)
    - [信号量（待完善）](#信号量待完善)
      - [P、V操作](#pv操作)
- [SPI(待完善)](#spi待完善)

# 注解和枚举

## 注解

### 常用注解

- `@SuppressWarnings`:可用于抑制编译器警告，常见可抑制的警告类型有：

  - "unchecked"：取消未检查的转换警告
  - "deprecation"：取消已过期 API 的警告
  - "rawtypes"：取消使用原始类型的警告
  - "unused"：取消未使用变量或方法的警告

- `@Documented`: 是一种元注解，被该注解标注的类，在利用`javadoc`生成文档时会包含注解信息，例如：

  ![20230426214106](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230426214106.png)

- `@Retention`: 指定注解的存活时间，如编译、类加载、运行时
  - RetentionPolicy.SOURCE：注解仅在源代码中保留，编译时会被丢弃。
  - RetentionPolicy.CLASS：注解在编译时保留，在类加载时被丢弃。
  - RetentionPolicy.RUNTIME：注解在运行时保留，可以通过反射机制读取。

- `@Target`: 用于指明注解可以修饰的目标，例如：

  - `@Target(ElementType.TYPE)`: 用于修饰类、接口、枚举
  - `@Target(ElementType.FIELD)`: 用于修饰成员变量
  - ...

- `@Inherited`: 被该注解标注的注解，在标记父类的时候，其子类也会继承该注解，例如：

  ```java
  @Inherited
  public @interface hasInherited {
  }
  ```

  ```java
  @hasInherited
  public class Father {...}
  ```

  ```java
  public class Child {...}
  ```

  此时虽然`Child`类没有被`@hasInherited`注解修饰，但如果通过反射`Child.class.getAnnotations()`方法获取到的注解数组中，会包含`@hasInherited`注解。

## lombok

### @Builder

`@Builder`注解主要是帮助省去了`set`方法，通过流式编程的方式为对象的每个成员变量赋值，例如

```java
// Studeten包含sno, sname, sage, sphone等成员变量
Student.builder()
       .sno( "001" )
       .sname( "admin" )
       .sage( 18 )
       .sphone( "110" )
       .build();
```

## 枚举

# 泛型

## 泛型类型

泛型包括：

- 泛型方法
- 泛型接口
- 泛型类

泛型方法主要指静态方法，例如：

```java
public static <E> void printArray(E[] inputArray) {
    for (E element : inputArray) {
        System.out.printf("%s ", element);
    }
    System.out.println();
}
```

泛型接口的实现方法可以指定或不指定接口中泛型的具体类型，例如：

```java
public interface Generator<T> {
    T next();
}
```

```java
public class GeneratorImpl implements Generator<T> {
    @Override
    // 实现的方法指定了具体类型为String
    public String next() {
        return null;
    }
}
```

```java
public class GeneratorImpl<T> implements Generator<T> {
    @Override
    // 不指定具体类型
    public T next() {
        return null;
    }
}
```

## 泛型擦除

泛型在编译期间，`T`会被擦除为`Object`，`T extends xxx`会被擦除为`xxx`。

如果不了解该机制，自己写`Overload`方法时可能会出现以下问题：

```java
public void method(List<String> list) {}
public void method(List<Integet> list) {}
```

上述的程式会报错，因为`String`和`Integer`最后都会被擦除为`Object`类型，因此两个方法的签名是一样的，会报错。

# IO

## 序列化

**为什么要序列化？**

  1. 为了保存对象的状态，以便将来恢复到当前状态
  2. 为了传输对象，通过序列化将对象转换为字节流，传输到其他地方，例如进行网络传输，或者保存到本地文件中；再通过反序列化将字节流转换为对象

**Java自带的序列化**

`java.io.Serializable`接口是Java自带的序列化，只要实现该接口即可（是个标识接口）。

通常会在实现了该接口的类中看到一个`serialVersionUID`的成员变量，这个变量的作用是版本控制，会在反序列化时检查和当前类的`serialVersionUID`是否一致，如果不一致则会抛出`InvalidClassException`异常。

**自带序列化的缺点**

在实际开发中很少用Java自带的序列化，因为存在以下问题：

- 不支持跨语言调用；
- 相比流行的序列化框架性能差，序列化后的字节数组体积大；
- 存在安全问题，

**目前常用的序列化框架**(待完善)

目前常用的框架有：Kryo、Protobuf、ProtoStuff、Hessian等，这里主要介绍Kryo

## 字节流

> **为什么字节流更快？**  

1. 字节流相比字符流有更大的缓冲区
2. 字节流直接操作底层字节，字符流还需要对字符进行编码和译码，耗费时间

**jdk9新增：**

- `readallbytes()` ：读取输入流中的所有字节，返回字节数组。
- `readnbytes(byte[] b, int off, int len)` ：阻塞直到读取 `len` 个字节。
- `transferto(outputstream out)` ： 将所有字节从一个输入流传递到一个输出流。

## 字符流

> **为什么要有字符流？**

    当文本中包含中文等字符时，其unicode编码包含2个字节(utf8包含3个字节)，字节流的读取一次只能读取一个字节，会发生乱码的问题

## 缓冲流

> **为什么要有缓冲流？**

    减少i/o次数，先将读取的字节或者字符存储在缓冲区(缓冲区是运行期间在内存中开辟，字节流通常是一个`8kb`的数组)，待缓冲区满或者调用`.flush`方法后将缓冲区数据发送，触发io中断

## Kryo

所有使用Kryo进行序列化的类，必须具有**无参构造方法**

## NIO

NIO: Non-blocking IO，即同步非阻塞I/O

### 三大组件

**Channel**

数据从`Channel`到来后，本地通过线程将`Channel`中的数据读取到`Buffer`中；同样如果需要发送数据给服务器/客户端，可以通过将数据先发送到`Buffer`，然后再写给`Channel`以此来发送数据。

**Buffer**

常用的Buffer：

- ByteBuffer
  - MappedByteBuffer
  - DirectByteBuffer
  - HeapByteBuffer

- 常用的方法：
  - `filp()`: 进入读模式
  - `clear()`: 进入写模式
  - `compact()`: 进入写模式，保留未读数据
  - `rewind()`: 将`position`置0，即重新读取数据
  - `mark()`: 标记当前`position`的位置
  - `reset()`: 将`position`回到`mark`的位置，配合`mark()`使用

**Buffer的结构**：

![20230427200355](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230427200355.png)

在Buffer里有三个要注意的成员变量：`position`，`limit`，`capacity`, `mark`。他们分别是：

- `position`：当前读写的位置，可以理解为一个指针；
- `limit`：表示当前缓冲区对指针移动范围的限制；
- `capacity`：表示缓冲区的大小。
- `mark`：当调用`mark()`方法时，用`mark`标记当前`position`的位置，即`mark=position`；当调用`reset`方法时，`position`会回到`mark`的位置

在写模式下，向`ByteBuffer`中写入数据后，其结构如下：

![0018](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/0018.png)

当我们通过`flip`函数将`Buffer`转换为读模式时，其结构如下：

![0019](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/0019.png)

再来看下`flip`的源码：

```java
public final Buffer flip() {
    limit = position;
    position = 0;
    mark = -1;
    return this;
}
```

当我们要转回读模式时，有两个方法`clear()`和`compact()`，他们的区别在于：

- `clear()`: 将`position`设置为0，`limit`设置为`capacity`。不用清空`Buffer`内的数据，因为会被之后写入的数据覆盖。其源码如下：

  ```java
  public final Buffer clear() {
      position = 0;
      limit = capacity;
      mark = -1;
      return this;
  }
  ```

- `compact()`: 目前只知道`Buffer`没有`compact()`方法，而是在`ByteBuffer`中的一个抽象方法。调用`compact()`会将未读完的数据移动到`Buffer`的开头，然后将`position`设置为未读完的数据的长度，`limit`设置为`capacity`。

> `clear()`和`compact()`不同在于，如果`Buffer`中有没有读完的数据，即`position`指针没有移动到`limit`处，调用`clear()`会忽略未读完的数据，而调用`compact()`则会保留未读完的数据

**Selector**

由于非阻塞程序会一直循环等待事件到来，频繁的触发用户态和内核态的切换会导致资源的浪费，白白占用CPU的时间片段。引入`Selector`的目的是解决非阻塞的资源浪费问题，同时一个线程配合`Selector`可以监控多个不同事件，例如绑定服务器的`ServerSocketChannel`用作监听请求创建链接事件`OP_ACCEPT`，绑定`ServerSocketChannel`生成的`SocketChannel`用作监听读写事件`OP_READ`, `OP_WRITE`

如何绑定`Selector`和`Channel`，如下：

```java
// 创建channel
ServerSocketChannel ssc = ServerSocketChannel.open();
// 创建selector
selector = Selector.open();
// 将channel和selector进行绑定
SelectionKey ssckey = ssc.register(selector, 0, null);
```

可以设置Channel用于监听什么类型事件，例如监听创建链接事件：

```java
// 可以在创建链接后指定
ssckey.interestOps(SelectionKey.OP_ACCEPT);
// 也可以在创建链接时指定
SelectionKey ssckey = ssc.register(selector, SelectionKey.OP_ACCEPT, null);
```

在程序开始时候通过`selector.select()`对程序进行阻塞直到监听到`Accept`事件时继续运行，此时`ServerSocketChannel`会通过`.accept()`方法返回`SocketChannel`来建立与客户端的通信信道。我们可以通过`Selector`将该信道绑定至监听读写事件，如下：

```java
ServerSocketChannel c = (ServerSocketChannel) key.channel();
SocketChannel sc = c.accept();
```

### 文件编程

**FileChannel**

`FileChannel`只能工作在`阻塞模式`下，可以通过`FileInputStream`, `FileOutputStream`和`RandomAccessFile`的`getChannel()`方法来获取，区别在于：

- `FileInputStream`: 获取的channel只能读；
- `FileOutputStream`: 获取的channel只能写；
- `RadomAccessFile`: 根据设置的读写模式决定，例如`rw`既可以读也可以写

**File Path**

对文件的操作通常是：移动、复制、删除、创建等。`Files`类提供了一些静态方法来操作文件，例如：

- `Files.move(Path source, Path target, CopyOption... options)`: 移动文件
- `Files.copy(Path source, Path target, CopyOption... options)`: 复制文件
  - `StandardCopyOption.REPLACE_EXISTING`：如果目标文件存在，替换目标文件
- `Files.delete(Path path)`: 删除文件
- `Files.createDirecotries(Path path)`: 创建多级目录; `Files.createDirectory(Path path)`: 创建单个目录

我们可以用这个API来做一些常见的操作，例如：

1. 删除多级目录，如果我们直接调用`Files.delete()`方法，当目录非空的时候，会抛出`DirectoryNotEmptyException`异常。我们需要借助`Files.walkFileTree()`方法来遍历目录，然后删除目录下的所有文件和目录，最后删除目录本身。代码如下：

```java
public static void deleteDirectory(Path path) throws IOException {
    Files.walkFileTree(path, new SimpleFileVisitor<Path>() {
        // SimpleFileVisitor提供了4个方法可以重写，分别是：
        // preVisitDirectory: 在访问目录前调用
        // postVisitDirectory: 在访问目录后调用
        // visitFile: 在访问文件时调用
        // visitFileFailed: 在访问文件失败时调用
        @Override
        public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
            Files.delete(file);
            return super.visitFile(file, attrs);
        }

        // 因此这里删除多级目录的思路是：访问到文件时，删除文件；访问到目录时因为没有重写preVisitDirectory方法，不进行任何操作
        // 在删除了目录下所有文件后，因为重写了postVisitDirectory方法，最后删除目录
        @Override
        public FileVisitResult postVisitDirectory(Path dir, IOException exc) throws IOException {
            Files.delete(dir);
            return super.postVisitDirectory(dir, exc);
        }
    });
}
```

## BIO

BIO: Blocking I/O，即同步阻塞IO

## AIO

Asynchronous I/O，即异步IO

# 反射

## 获取Class的方式

1. 通过具体的类名时:

  ```java
  Class class = ClassName.class;
  ```

2. 通过类路径:

    - 通过`Class.forName()`方法

      ```java
      Class class = Class.forName("com.example.ClassName");
      ```

    - 通过类加载器

      ```java
      ClassLoader.getSystemClassLoader().loadClass("cn.javaguide.TargetObject");
      ```

3. 通过实例对象:

  ```java
  ClassName instance = new ClassName();
  Class class = instance.getClass();
  ```

## 反射的作用

通过反射可以获取类的：

- 方法，Java中`Method`是发射包中的类
  - `method.invoke(Object instance, Object... args)`方法可以调用实例对象的方法
-

## 动态代理

创建动态代理实例`Proxy.newProxyInstance(classLoader, interfaces, invocationHandler)`三个参数分别代表如下：

- `classLoader`：目标对象的类加载起
- `interfaces`：目标对象实现的所有接口
- `invocationHandler`：如何重写目标对象的方法过程

其中`invocationHandler`是一个接口，要实现`invoke`方法来重写目标对象的方法过程。

### JDK代理

- 代理对象和目标对象实现同个接口

### cglib代理

- 代理对象继承自目标对象


# 网络编程

![20230426155332](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230426155332.png)

上图是通过Socket建立网络通信的流程图，可以理解为：

- 服务器端通过`ServerSocket`监听端口，等待客户端连接
- 客户端通过`Socket`连接服务器端，建立连接后，客户端和服务器端都会有一个`Socket`对象，通过该对象进行通信
- 客户端结束通信后，调用`Socket`的`close()`方法关闭连接，服务器端通过`accept()`方法接收到客户端关闭连接的消息，关闭服务器端的`Socket`对象

举个简单的例子：

1. 服务器建立Socket，绑定并监听指定端口：

```java
public class HelloServer {
    Logger logger = Logger.getLogger(HelloServer.class.getName());

    public void listenAndRecive(int port) {
        // 1. 建立服务器的socket，绑定并监听指定端口好
        try (ServerSocket serverSocket = new ServerSocket(port);) {
            Socket socket;
            while ((socket = serverSocket.accept()) != null) {
                // 2. 获取服务器socket的对象输入输出流，以便读取客户端发来的数据和发送数据到客户端
                ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());
                ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
                // 3. 读取客户端发来的数据
                Message message = (Message) objectInputStream.readObject();
                logger.info("receive message from client: " + message.getContent());
                // 4. 发送数据到客户端
                message.setContent("Hello client!");
                objectOutputStream.writeObject(message);
                // 刷新缓存让数据立即发送
                objectOutputStream.flush();
            }
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        HelloServer helloServer = new HelloServer();
        helloServer.listenAndRecive(6666);
    }

}
```

2. 客户端通过指定ip地址和端口号建立Socket，通过Socket的输入输出流实现与服务端的通信

```java
public class HelloClient {
    Logger logger = Logger.getLogger(HelloClient.class.getName());

    public Message send(Message message, String host, int port) {
        // 1. 建立客户端的Socket，绑定ip地址和端口号
        try (Socket socket = new Socket(host, port)) {
            // 2. 获取socket的输出流，以便向服务器写数据
            ObjectOutputStream objectOutputStream = new ObjectOutputStream(socket.getOutputStream());
            // 向服务器写数据，通过writeObject()方法传输Message对象的实例数据
            objectOutputStream.writeObject(message);
            logger.info("send message to server: " + message.getContent());
            // 3. 获取socket的输入流，以便从服务器读数据
            ObjectInputStream objectInputStream = new ObjectInputStream(socket.getInputStream());
            logger.info("receive message from server: " + ((Message) objectInputStream.readObject()).getContent());
            return (Message) objectInputStream.readObject();
        } catch (IOException | ClassNotFoundException e) {
            throw new RuntimeException(e);
        }
    }

    public static void main(String[] args) {
        HelloClient helloClient = new HelloClient();
        Message ms = new Message();
        ms.setContent("Hello server!");
        Message message = helloClient.send(ms, "127.0.0.1", 6666);
    }
}
```

> 这里的`Message`是自定义的POJO类，里面只有`String content`成员变量、构造方法和set/get方法

然而这样的网络编程是**阻塞形式**的，服务端在调用`serverSocket.accpet()`时会触发阻塞，直到接收到客户端的请求时才会继续执行后面的程序

# 多线程

## Unsafe类

主要功能有：

1. 内存操作
2. 内存屏障
3. 对象操作
4. 数据操作
5. CAS 操作
6. 线程调度
7. Class 操作
8. 系统信息

### 内存操作

```java
// 申请bytes字节的空间，返回long型的地址信息
public native long allocateMemory(long bytes);
//重新调整内存空间的大小
public native long reallocateMemory(long address, long bytes);
//将内存设置为指定值
public native void setMemory(Object o, long offset, long bytes, byte value);
//内存拷贝
public native void copyMemory(Object srcBase, long srcOffset,Object destBase, long destOffset,long bytes);
//清除内存
public native void freeMemory(long address);
```

本地方法申请的都是堆外内存，无法通过GC回收，需要手动通过`freeMemory`进行释放。

### 内存屏障(待完善)

```java
//内存屏障，禁止load操作重排序。屏障前的load操作不能被重排序到屏障后，屏障后的load操作不能被重排序到屏障前
public native void loadFence();
//内存屏障，禁止store操作重排序。屏障前的store操作不能被重排序到屏障后，屏障后的store操作不能被重排序到屏障前
public native void storeFence();
//内存屏障，禁止load、store操作重排序
public native void fullFence();
```

内存屏障可以看做一个同步点，会禁止屏障前的读/写操作**重排序**，保证同步点前所有读写操作执行完成后再执行后续程序

(待完善) load和store操作分别是什么

## 线程的生命周期

- `NEW`：新建状态，线程对象被创建，但是还没有调用`start()`方法
- `RUNNABLE`：可运行状态，线程对象调用`start()`方法后，线程进入可运行状态，但是并不一定立即执行，只是表示可以执行
- `BLOCKED`：阻塞状态，线程在运行过程中，可能由于资源竞争等原因被阻塞，此时线程进入阻塞状态，等待锁释放
- `WAITING`：等待状态，线程调用`wait()`方法、`join()`方法、`park()`方法等，此时线程进入等待状态，需要其他线程唤醒(通知或中断)
- `TIMED_WAITING`：超时等待状态，需要其他线程唤醒(通知或中断)，或者等待超时
- `TERMINATED`：终止状态，线程执行完毕或者因异常退出，此时线程进入终止状态

![20230515221206](https://raw.githubusercontent.com/PercivalYang/imgsSaving/main/imgs/20230515221206.png)

## 对象头

### mark word(待完善)

### 重量级锁

**线程饥饿**:指的是线程长时间竞争不到锁，无法获取CPU时间片和资源的情况

- 通常由`synchronized`实现，关键在于多线程竞争锁时，会产生阻塞和唤醒线程的开销
- 保证公平性，按照“先来先服务”原则对阻塞的线程排序

> 自旋和阻塞的区别在于：被阻塞的线程会被挂起，从而不占用CPU资源；自旋锁仍占用CPU资源，在这期间重复执行简单的操作并尝试获取锁
>

### 轻量级锁

- 通常由**CAS**机制实现，不会被阻塞，期间仍能利用CPU资源执行操作，效率高
- 轻量级锁自旋等待一定次数后会升级为重量级锁

### 偏向锁

### 无锁

## 线程池（待完善）

### Executors

是一个工具类，通常用于创建线程池，例如

### ExecutorService

**常用功能：**

- 创建线程池
  - `FixedThreadPool`：线程数固定的线程池；
  - `CachedThreadPool`：线程数根据任务动态调整的线程池；
  - `SingleThreadExecutor`：仅单线程执行的线程池。
  - `ScheduledThreadPool`：计划性线程池，可配置定时任务
- 提交任务
- 执行定时任务

```java
// 创建线程池，线程数为10
ExecutorService executor = Executors.newFixedThreadPool(10);
// 提交任务给线程池
executor.submit(new RunnableTask());  

// 创建有定时功能的线程池，线程数为10
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
// 延迟 1 秒执行
scheduledExecutorService.schedule(() -> {
    // 代码块
}, 1, TimeUnit.SECONDS);

// 延迟 1 秒后每隔 2 秒执行一次
scheduledExecutorService.scheduleAtFixedRate(() -> {
    // 代码块
}, 1, 2, TimeUnit.SECONDS);
```

**Future**

- `Future`是接口，`FutureTask`是其实现类，代表异步计算的结果
- 把`Runnable`或者`Callable`接口的实现类提交(`.submit()`)给线程池后，会返回一个`FutureTask`对象，如右

```java
Future<String> future = executor.submit(task);
String result = future.get(); // 等待任务执行并获取结果
```

## 线程组

每个线程**必然**存在于一个`ThreadGroup`中，如果没有显示指明，则该线程的线程组**默认为父线程的线程组。**例如在`main()`中创建一个`Thread`，其线程组名字则设置为其父线程`main`

`ThreadGroup`是**向下引用**的树结构，目的是**防止“上级”线程被“下级”引用而无法有效的GC**

## 线程通信模式

|                           |                                                 |
|-------------------|---------------------------------------|
| 消息传递并发模型          | 线程之间没有公共状态，通过发送消息进行线程间通信 |
| 共享内存并发模型（JVM采用） | 读写内存中的公共状态通信，需要互斥               |
|                           |                                                 |

### happens-before

- 具有**可见性**，即线程A happens-before 线程B，则其在公共区的执行结果对线程B是可见的
- 若执行结果一致，happens-before原则不影响JVM**重排序**

### 线程共享

#### volatile（待完善）

用来修饰**变量**，不能修饰方法

- 保证变量可见性
- 禁止指令重排序
- 不保证原子性
- 内存屏障
- 实现高效的单例模式、双重检查锁定等

## 线程同步

### 互斥锁

#### synchronized（待完善）

考点

#### ReentrantLock（待完善）

- 具备**尝试获取锁**的功能，例如下述代码尝试获取锁，等待1秒后获取不到则放弃

```java
if (lock.tryLock(1, TimeUnit.SECONDS)) {
    try {
        ...
    } finally {
        lock.unlock();
    }
}
```

#### condition（待完善）

#### ReadWriteLock（待完善）

#### StampedLock（待完善）

### 信号量（待完善）

> 待完善：如何实现互斥和同步，具体怎么操作
>

#### P、V操作

- `sem`表示资源的数量
- P：`sem`减1后，若`sem<0`则阻塞，否则线程继续
- V：`sem`加1后，若`sem<=0`则唤醒阻塞的线程
- 具有原子性


# SPI(待完善)

SPI全称为 `Service Provider Interface`，和API相同的是，他们都存在：

- 服务接口
- 服务实现类

不同的是：

- API的服务接口和实现类都是由服务实现方提供的，调用方仅是调用它们提供的接口；
- SPI的服务接口是由服务调用方提供，然后由实现方提供服务实现类，调用方通过接口调用实现类。
