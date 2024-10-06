# 1. 异常

Java中异常父类为Throwable，子类为Error和Exception。

Exception：这是通常说的异常父类，子类主要分为RuntimeException和其他异常。出现后可处理。

Error：主要是虚拟机错误或者线程错误等，程序终结者。出现程序终结。

# 2. Exception

- 编译时异常（也称受查异常）
- 运行时异常（也称非受查异常(Unchecked Exception)）（RunTimeException以及其子类对应的异常，都称为运行时异常，比如上述的数组越界，空指针，算数异常等等）

# 3. 异常处理

## 3.1 异常抛出和捕获

异常抛出：`throw new 异常类("异常原因");`

```java
// 捕获异常
public class ExceptionTest {
    public static void main(String[] args) {
        try {
            // 可能出现异常代码
        } catch (Exception e) {
            // 匹配出现的代码
            // 出现匹配代码后进行处理
        } finally {
            // 始终会执行代码，常用于
        }
    }
}
```

**_PS：
catch匹配异常可以出现多个。
catch匹配多个时，父类异常在后(父亲兜底)_**

## 3.2 异常声明

> 当前方法存在异常，但是不进行处理，通过异常声明，提醒调用者进行处理。

异常声明：

```java
public String testMethod(String str) throws NullPointerException {
    // ...
}
```

# 4. 自定义异常

创建类继承Exception或RuntimeException即可。继承前者表示为检测异常，编写代码时必须进行处理。继承后者为运行时异常，无需提前处理。

```java
// 示例
public class IOException extends Exception {
    static final long serialVersionUID = 7818375828146090155L;

    public IOException() {
        super();
    }

    public IOException(String message) {
        super(message);
    }

    public IOException(String message, Throwable cause) {
        super(message, cause);
    }


    public IOException(Throwable cause) {
        super(cause);
    }
}
```