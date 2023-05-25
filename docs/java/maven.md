# 坐标解释

- groupId: 组织标识，一般第一段为域，第二段为公司，第三段为项目名，第四段为模块名
  - 常见域：org: 非盈利; com: 商业公司; cn: 中国
- artifactId: 项目唯一标识符

# 依赖范围

Maven 在编译、测试、运行有着三种`classpath`，可以理解为：

- 编译classpath: 编译主代码有效
- 测试classpath: 编译、运行测试代码有效
- 运行classpath: 项目运行时有效

以下各种范围的目的就是为了控制依赖在这三种`classpath`中的有效性：

- compile(默认): 编译、测试、运行都有效
- test: 只在测试时有效
- provided: 编译、测试有效，运行无效
- runtime: 测试、运行有效，编译无效
- system: 与provided类似，但需要通过systemPath指定依赖jar包的路径，不依赖Maven仓库解析

# 依赖冲突

依赖冲突指同个groupId和artifactId的依赖在不同版本下的冲突，例如项目存在如下依赖关系：

```
依赖链路一：A -> B -> C -> X(1.0)
依赖链路二：A -> D -> X(2.0)
```

## 最短路径优先

即优先选择依赖链路最短的版本，如上面会选择2.0版本的X

## 声明顺序优先

当链路长度相同时，根据`pom.xml`文件中声明顺序，谁在前选择谁

## 通过排除依赖解决冲突

例如项目存在如下依赖关系：

```
依赖链路一：A -> B -> C -> X(1.5) // dist = 3
依赖链路二：A -> D -> X(1.0) // dist = 2
```

通过最短路径原则排除了1.5版本的X，这样C中的某个类/方法在1.0版本中不存在，就会出现`NoClassDefFoundError`或`NoSuchMethodError`异常。这时可以通过排除依赖解决冲突，例如：

```xml
<dependencyD>
    ......
    <exclusions>
      <exclusion>
        <artifactId>x</artifactId>
        <groupId>org.apache.x</groupId>
      </exclusion>
    </exclusions>
</dependency>
```

这样D就不会再引入1.0版本的X依赖，避免了依赖冲突，也解决了引入低版本不兼容的依赖

# 生命周期

## default

是Maven的主要生命周期，包含有23个阶段：

```xml
<phases>
  <!-- 验证项目是否正确，并且所有必要的信息可用于完成构建过程 -->
  <phase>validate</phase>
  <!-- 建立初始化状态，例如设置属性 -->
  <phase>initialize</phase>
  <!-- 生成要包含在编译阶段的源代码 -->
  <phase>generate-sources</phase>
  <!-- 处理源代码 -->
  <phase>process-sources</phase>
  <!-- 生成要包含在包中的资源 -->
  <phase>generate-resources</phase>
  <!-- 将资源复制并处理到目标目录中，为打包阶段做好准备。 -->
  <phase>process-resources</phase>
  <!-- 编译项目的源代码  -->
  <phase>compile</phase>
  <!-- 对编译生成的文件进行后处理，例如对 Java 类进行字节码增强/优化 -->
  <phase>process-classes</phase>
  <!-- 生成要包含在编译阶段的任何测试源代码 -->
  <phase>generate-test-sources</phase>
  <!-- 处理测试源代码 -->
  <phase>process-test-sources</phase>
  <!-- 生成要包含在编译阶段的测试源代码 -->
  <phase>generate-test-resources</phase>
  <!-- 处理从测试代码文件编译生成的文件 -->
  <phase>process-test-resources</phase>
  <!-- 编译测试源代码 -->
  <phase>test-compile</phase>
  <!-- 处理从测试代码文件编译生成的文件 -->
  <phase>process-test-classes</phase>
  <!-- 使用合适的单元测试框架（Junit 就是其中之一）运行测试 -->
  <phase>test</phase>
  <!-- 在实际打包之前，执行任何的必要的操作为打包做准备 -->
  <phase>prepare-package</phase>
  <!-- 获取已编译的代码并将其打包成可分发的格式，例如 JAR、WAR 或 EAR 文件 -->
  <phase>package</phase>
  <!-- 在执行集成测试之前执行所需的操作。 例如，设置所需的环境 -->
  <phase>pre-integration-test</phase>
  <!-- 处理并在必要时部署软件包到集成测试可以运行的环境 -->
  <phase>integration-test</phase>
  <!-- 执行集成测试后执行所需的操作。 例如，清理环境  -->
  <phase>post-integration-test</phase>
  <!-- 运行任何检查以验证打的包是否有效并符合质量标准。 -->
  <phase>verify</phase>
  <!-- 	将包安装到本地仓库中，可以作为本地其他项目的依赖 -->
  <phase>install</phase>
  <!-- 将最终的项目包复制到远程仓库中与其他开发者和项目共享 -->
  <phase>deploy</phase>
</phases>
```

## clean

```xml
<phases>
  <!--  执行一些需要在clean之前完成的工作 -->
  <phase>pre-clean</phase>
  <!--  移除所有上一次构建生成的文件 -->
  <phase>clean</phase>
  <!--  执行一些需要在clean之后立刻完成的工作 -->
  <phase>post-clean</phase>
</phases>
<default-phases>
  <clean>
    org.apache.maven.plugins:maven-clean-plugin:2.5:clean
  </clean>
</default-phases>
```

## site


```xml
<phases>
  <!--  执行一些需要在生成站点文档之前完成的工作 -->
  <phase>pre-site</phase>
  <!--  生成项目的站点文档作 -->
  <phase>site</phase>
  <!--  执行一些需要在生成站点文档之后完成的工作，并且为部署做准备 -->
  <phase>post-site</phase>
  <!--  将生成的站点文档部署到特定的服务器上 -->
  <phase>site-deploy</phase>
</phases>
<default-phases>
  <site>
    org.apache.maven.plugins:maven-site-plugin:3.3:site
  </site>
  <site-deploy>
    org.apache.maven.plugins:maven-site-plugin:3.3:deploy
  </site-deploy>
</default-phases>
```