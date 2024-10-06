# 1. 枚举

枚举是一种特殊的数据类型，他是一组预定义的常量，每个常量包括数据和名称。

> 枚举无法继承其他类，默认继承Enum类，可以实现其它接口。
> 
> 枚举常量命名全大写，多单词下划线分割。

# 2. 创建枚举

```java
public enum Week {
    Monday("星期一", "Mon", 1),
    Tuesday("星期二", "Tues", 2),
    Wednesday("星期三", "Wed", 3),
    Thursday("星期四", "Thur", 4),
    Friday("星期五", "Fri", 5),
    Saturday("星期六", "Sat", 6),
    Sunday("星期日", "Sun", 7);

    // 枚举属性
    private String value;
    private String code;
    private int num;

    // 枚举构造，必须私有
    private Week(String value, String code, int num) {
        this.value = value;
        this.code = code;
        this.num = num;
    }

    public String getValue() {
        return value;
    }

    public String getCode() {
        return code;
    }

    public int getNum() {
        return num;
    }

    // 枚举方法
    public boolean isWeekend() {
        return this == Saturday || this == Sunday;
    }
}
```

```java
// 需要实现接口
interface Printable {
    void print();
}

// 实现接口
public enum Week implements Printable {
    Monday("星期一", "Mon", 1), Tuesday("星期二", "Tues", 2), Wednesday("星期三", "Wed", 3), Thursday("星期四", "Thur", 4), Friday("星期五", "Fri", 5), Saturday("星期六", "Sat", 6), Sunday("星期日", "Sun", 7);

    // 枚举属性
    private String value;
    private String code;
    private int num;

    // 枚举构造，必须私有
    private Week(String value, String code, int num) {
        this.value = value;
        this.code = code;
        this.num = num;
    }

    public String getValue() {
        return value;
    }

    public String getCode() {
        return code;
    }

    public int getNum() {
        return num;
    }

    // 枚举方法
    public boolean isWeekend() {
        return this == Saturday || this == Sunday;
    }

    @Override
    public void print() {
        System.out.println("Today is " + this.value);
    }
}
```

# 3. 枚举使用

```java
public static void main(String[] args) {
    // 访问枚举常量
    Week monday = Week.Monday;
    // 访问枚举常量属性
    System.out.println(monday.getValue());
    System.out.println(Week.Monday.getValue());
    // 访问枚举方法
    System.out.println(monday.isWeekend());
    System.out.println(Week.Monday.isWeekend());
    Week.Sunday.print();
}
```

# 4. 枚举默认方法

`values()`:获得所有枚举常量数组。

```java
public static void main(String[] args) {
    Week[] values = Week.values();
    for (Week value : values) {
        System.out.println(value);
    }
}

// Monday
// Tuesday
// Wednesday
// Thursday
// Friday
// Saturday
// Sunday
```

`valueOf(String)`:通过字符串，获得对应枚举常量。

```java
public static void main(String[] args) {
    Week sunday = Week.valueOf("Sunday");
    System.out.println(sunday.getValue());
}
```

