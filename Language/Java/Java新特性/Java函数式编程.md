# 1、简述

函数：只要输入相同，无论多少次调用，无论什么时间调用，输出相同。

> 函数像是一个无情的机器，只要相同的输入，无论如何都是给出相同的返回值。

## 1.1 函数对象化

> 函数是固定在某个类中，是被度定下来的规则。当我们想在不同的地方使用规则，创建对象调用即可。
> 
> 新特性中可以将函数对象化，让函数成为一个对象，进行传播，位置不在固定。

```java
public class LambdaTest {
    public static void main(String[] args) {
        int add = add(1, 2);// 固定函数
        System.out.println(add);

        Lambda lambda = (int a, int b) -> a + b;// 函数对象化
        int plus = lambda.plus(1, 2);
        System.out.println(plus);
    }

    static int add(int a, int b) {
        return a + b;
    }

    // 函数式接口
    interface Lambda {
        int plus(int a, int b);
    }
}
```

## 1.2 行为参数化

> 将一些行为当成参数进行传递。

```java



```
