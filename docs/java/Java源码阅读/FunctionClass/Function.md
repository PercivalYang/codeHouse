
# 基本概念

是一个功能性接口，需要实现apply方法，函数签名如下：

```java
R apply(T t);
```

## 其他方法

除开需要实现的`apply`方法，该接口还包含2个`default`和1个静态方法

**`default`**方法：

- `compose`:

    ```java
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }
    ```

- `andThen`:

    ```java
        default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }
    ```

> `compose`和`andThen`方法都是将两个`Function`的`apply`方法进行嵌套，区别在于执行顺序不同。

**静态方法**：

- `identity`: 返回一个实现了`apply`的`Function`，`apply`方法返回传入的参数，源码如下：

    ```java
    static <T> Function<T, T> identity() {
        return t -> t;
    }
    ```