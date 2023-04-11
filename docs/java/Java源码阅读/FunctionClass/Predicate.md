
# 基本概念

`Predicate`是一个接口，他需要实现的方法是`bool test(T t)`。可以看出它接受一个泛形参数，输出是bool值。

我们可以用`lambda`表达式来实现这个接口。

```java
Predicate<String> predicate = (s) -> s.length() > 0;
```

也可以通过匿名内部类来实现这个接口。

```java
Predicate<String> predicate = new Predicate<String>() {
    @Override
    public boolean test(String s) {
        return s.length() > 0;
    }
};
```

调用：

```java
predicate.test("");     // 返回false
predicate.test("a");    // 返回true
```

# 其他方法

`Predicate`内部还有3个默认方法和1个静态方法，这4个方法的返回值类型都是`Predicate`

default方法：

- `and`:

    ```java
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) && other.test(t);
    }
    ```

- `or`:

    ```java
    default Predicate<T> or(Predicate<? super T> other) {
        Objects.requireNonNull(other);
        return (t) -> test(t) || other.test(t);
    }
    ```

- `negate`:

    ```java
    default Predicate<T> negate() {
        return (t) -> !test(t);
    }
    ```

静态方法：

- `isEqual`：返回一个实现了`test`的`Predicate`，`test`能判断两个对象是否相等，源码如下：

    ```java
    static <T> Predicate<T> isEqual(Object targetRef) {
        return (null == targetRef)
                ? Objects::isNull
                : object -> targetRef.equals(object);
    }
    ```
