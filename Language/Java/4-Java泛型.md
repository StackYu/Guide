# 1、泛型

**什么是泛型：** JDK1.5引入的新特性，他提供了程序编译时类型安全检测机制，该机制允许程序员在编译时检查非法类型。

**泛型本质：** 泛型本质是参数化类型，即指定类，方法或接口只能操作某一种数据类型。

**为什么用泛型：** 泛型提供了编译时类型安全检测机制，允许在编译时检测非法数据类型。

**泛型作用：**

* 安全：编译时就进行类型检测，避免在运行时出错。
* 消除强转：规定了参数类型，避免多余的类型强制转换。
* 避免冗余拆装箱：当存入数值时，存入Object需要装箱，获得时又要拆箱。
* 代码重用

# 2、使用

## 2.1 泛型类

```java
// 示例一
public class Generic<T> {
    // key 这个成员变量的数据类型为 T, T 的类型由外部传入  
    private T key;

    // 泛型构造方法形参 key 的类型也为 T，T 的类型由外部传入
    public Generic(T key) {
        this.key = key;
    }

    // 泛型方法 getKey 的返回值类型为 T，T 的类型由外部指定
    public T getKey() {
        return key;
    }
}

// 示例二
public class Generic<K, V> {
    private K key;
    private V value;

    public Generic(K key, V value) {
        this.key = key;
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }
}

```

> **泛型类泛型使用位置：**
> 1. 非静态成员变量
> 2. 非静态成员方法（构造方法）形参
> 3. 非静态成员方法返回值类型


<font color=#FF0000>注意：</font>不可用于静态成员方法或静态成员变量，泛型是在编译时期确定，静态属性是在类加载是确定

```java
// 当创建带有泛型类时，不提供泛型，系统默认为Object
List list = new ArrayList<>();
// 相当于：List<Object> list = new ArrayList<>();
```

## 2.2 泛型接口

```java
// 基本语法
public interface IClass<K, V> {
    //...
}

// 接口继承泛型接口
public interface AClass extends IClass<String, Object> {
    // ...
}

// 不提供泛型，系统默认为Object
public interface AClass extends IClass {
    // ...
}

// 继承接口也不确定泛型类型
public interface AClass<K, V> extends IClass<K, V> {
    // ...
}

// 类实现泛型接口
class BClass implements IClass<String, Object> {
    // ...
}
```

> **泛型接口泛型使用位置：**
> 1. 普通方法、抽象方法、默认方法形参和返回值可以使用
> 2. 成员属性不可以（成员属性为静态）

## 2.3 泛型方法

```
// 语法：
public <类型参数> 返回类型 方法名（类型参数 变量名） {
    ...
}
```

> 只有在方法中声明了<T>的才是泛型方法，使用泛型类型并不是泛型方法

```java
// 示例
public class TestMethod<U> {
    // 泛型方法
    public <T, S> T testMethod(T t, S s) {
        return null;
    }

    // 静态泛型方法
    public static <E> E show(E one) {
        return null;
    }
}
```

> 泛型方法和泛型类分别定义的类型都是相互独立的。

# 3. 泛型通配符

**什么是通配符：** 通配符用于表示某一范围内类型。

**通配符3种形式：**

* `<?>`：无边界通配符
* `<? extends T>`：有上界通配符
* `<? super T>`：有下界通配符

## 3.1 上界通配符

**`<? extends T>`:** 表示泛型只能是T及T子类

**特点：** _可以读，不能写_

---

```java
// 编译错误
ArrayList<? extends Number> list = new ArrayList<Float>();

ArrayList<? extends Number> list = new ArrayList<>();
list.

add(new Integer(47));// 编译错误
        list.

add(new Float(3.14));// 编译错误
```

> `<? extends Number>`表示为任意一个Number及其子类集合，但是我们不能手动指定具体类型。类似接口，当为形参时，接收的是接口实现类，但是不能直接实例化接口传递过去。
>
> _**通配符规定了类型上线，也就是说，传入的子类必须要明确。是什么子类类型需要提前明确。语义规定上线，实则是明确具体子类。**_

```java
// 正确使用姿势
public void addList(List<? extends Number> list) {
    // ...
}

ArrayList<Integer> lists = new ArrayList<>();

// 调用
addList(lists);
```

## 3.2 下限通配符

`<? super Number>`：下限通配符，表示泛型只能是T及其父类

**特点：** _可以写，不能读_

---

```java
List<? super Number> list = new ArrayList<>();
list.

add(new Integer(10));  // 编译通过
        list.

add(new Float(12.3f)); // 编译通过

        list.

add(new Object()); // 编译失败
```

> ArrayList<? super Number> 的下界是 ArrayList< Number > 。因此，我们可以确定 Number 类及其子类的对象自然可以加入 ArrayList<? super Number>
> 集合中；
> 而 Number 类的父类对象就不能加入 ArrayList<? super Number> 集合中了，因为不能确定 ArrayList<? super Number> 集合的数据类型。
>
> *
*_以上编译通过，这和上界通配符感觉颠倒了，实则并没有。通配符规定了下限，Number的子类添加进来后，都会被强制转换成Number类。只要是Number都可以成功。_
**
> **_但是添加父类就会失败。_**

```java
// 正确调用
public static void main(String[] args) {
    List<? super Number> list = new ArrayList<>();
    list.add(new Integer(10));
    list.add(new Float(12.3f));
}

// 错误调用
public static void main(String[] args) {
    List<Integer> list = new ArrayList<>();
    list.add(new Integer(10));

    fillNumList(list); // 编译错误
}

public static void fillNumList(ArrayList<? super Number> list) {
    list.add(new Integer(0));// 编译正确
    list.add(new Float(1.0));// 编译正确

    // 遍历传入集合中的元素，并赋值给 Number 对象；会编译错误
    for (Number number : list) {
        System.out.print(number.intValue() + " ");
        System.out.println();
    }
    // 遍历传入集合中的元素，并赋值给 Object 对象；可以正确编译
    // 但只能调用 Object 类的方法，不建议这样使用
    for (Object obj : list) {
        System.out.println(obj);
    }
}

```

## 3.3 无界通配符

`<?>`：无界通配符，表示元素可以是任何类型

---

```java
public static void main(String[] args) {
    List<?> list = new ArrayList<>();
    list.add(null); // 编译通过
    Object o = list.get(0); // 编译通过

    list.add(new Integer(10));  //编译不能通过
    
    // 类型不能确定，所以只能添加null，同样获得到的数据只能赋值给Object
}
```

> `<?>`和`<Object>`不一样，逻辑上前者是后者的父类。一般我们使用都会使用`<T>`表示。

## 3.4 上下界通配符比较

`<? extends T>`上界通配符：**只允许读操作，不允许写操作。即只可以取值，不可以设值。**

`<? super T>`下界通配符：**只允许写操作，不允许读操作。即只可以设值（比如 set 操作），不可以取值（比如 get 操作）。**

```java
// jdk中示例
public class Collections {
    // 把 src 的每个元素复制到 dest 中:
    public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        for (int i = 0; i < src.size(); i++) {
        	// 获取 src 集合中的元素，并赋值给变量 t，其数据类型为 T
            T t = src.get(i);
            // 将变量 t 添加进 dest 集合中 
            dest.add(t);// 添加元素进入 dest 集合中
        }
    }
}

// copy() 方法的作用是把一个 List 中的每个元素依次添加到另一个 List 中。
// 它的第一个形参是 List<? super T>，表示目标 List，
// 第二个形参是 List<? extends T>，表示源 List。
```

## 3.5 PECS原则

> 我们何时使用 extends，何时使用 super 通配符呢？为了便于记忆，我们可以用 PECS 原则：Producer Extends Consumer Super。

# 4. 类型擦除

> 泛型在编译时会对类进行检查，检查后就不在保留泛型的信息。这就是类型擦除，这也是泛型实现的一种机制。也就是说泛型只保留在编译期，运行时没有任何泛型信息。
> 
> 大部分情况下，泛型在编译后都是Object来替换的，使用有界泛型就会发生变化，示例如下。

```java
public class FandXingTest<T extends Number> {
   private T num;
}

// 编译后
public class Caculate {
    public Caculate() {}// 默认构造器
    private Number num; // 直接使用Number类代替
}
```





