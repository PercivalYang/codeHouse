
## 目录

- [Java基础](./Java%E5%9F%BA%E7%A1%80.md)
- [设计模式](./%E8%AE%BE%E8%AE%A1%E6%A8%A1%E5%BC%8F.md)
- [Java源码阅读](./Java%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB/README.md)
- [MySQL](./MySQL.md)
- Java开发
  - [Spring源码解析](./spring%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
  - [Mybatis](./mybatis.md)

## 待分类的笔记

### 实体类和DTO类的关系，为什么要用DTO？

1. DTO类可以对实体类中的数据进行隐藏，例如在Service层我们通过手机号查询到客户的信息，此时返回的实体类中包含有客户的隐私信息例如密码、手机号等。我们不希望将这些信息都返回给前端，避免隐私泄漏的风险，因此可以使用DTO类将实体类中的数据进行隐藏，只返回给前端需要的数据。

具体操作：因为DTO和实体类之间没有继承关系，不能通过强转的方式将实体类转换为DTO类，因此需要使用BeanUtils工具类中的copyProperties方法将实体类中的数据复制到DTO类中。可以新建一个类如下：

```java
public static UserDTO transerToDTO(User user) {
    UserDTO userDTO = new UserDTO();
    BeanUtils.copyProperties(user, userDTO);
    return userDTO;
}
```
