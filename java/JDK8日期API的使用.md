# JDK8日期API的使用

### 老版日期API的弊端：

java面世之初，引入java.util.Date和java.util.Calendar这两个类处理时间。但现在其中很多的方法很早之前就都已被废弃，且存在很多的问题。

1、毫秒值与日期直接转换比较繁琐，通过毫秒值来计算时间的差额步骤较多，且可能存在误差。

2、SimpleDateFormat类是线程不安全的，在多线程情况下，全局共享一个SimpleDateFormat类中的Calendar对象有可能出现异常。（可通过同步方式处理）

3、使用规范问题。在Date和Calendar类之前，枚举类（ENUM）还没有出现，所以在字段中使用整数常量导致整数常量都是可变的，且存在魔法数字的情况，不是线程安全的。

### 新版日期API的使用

常用类的概述与功能介绍

| 类                | 介绍                                                         |
| ----------------- | ------------------------------------------------------------ |
| Instant           | 对时间轴上的单一瞬时点建模，可以用于记录应用程序中的时间时间戳，在其他时间类型的转换中，均可以使用此类作为中间类完成转换。 |
| Duration          | 表示秒或纳秒时间间隔，适合处理较短的时间，需要更高的精确度。 |
| Period            | 表示一段时间的年、月、日。                                   |
| LocalDate         | 一个不可变的日期时间对象，表示日期，通常被视为年月日。       |
| LocalTime         | 一个不可变的日期时间对象，代表一个时间，通常被看作是时-分-秒，时间表示为纳秒精度。 |
| LocalDateTime     | 一个不可变的日期时间对象，代表日期时间，通常被视为年-月-日-时-分-秒。 |
| ZonedDateTime     | 具有时区的一个不可变的日期时间对象，此类存储所有日期和时间字段，精度为纳秒，时区为区域偏移量，用于处理模糊的本地日期时间。 |
| Year              | 表示年                                                       |
| YearMonth         | 表示年月                                                     |
| MonthDay          | 表示月日                                                     |
| DateTimeFormatter | 自定义格式化转换器                                           |

now方法：根据当前日期或时间创建实例。

of方法：根据给定的参数生成对应的日期/时间对象，基本上每个时间类都有of方法，且重载形式多变。

plus方法：根据现有时间对象进行日期的增加，并返回一个新的日期对象。

with方法：直接修改日期。

### Date和LocalDateTime互相转化

```java
public class DateTimeExchangeUtil {

    public static LocalDateTime parseToLocalDateTime(Date date) {
        // 方法一
        return date.toInstant().atZone(ZoneId.systemDefault()).toLocalDateTime();
        // 方法二
        // LocalDateTime.ofInstant(date.toInstant(), ZoneId.systemDefault());
    }

    public static Date parseToDate(LocalDateTime localDateTime) {
        return Date.from(localDateTime.atZone(ZoneId.systemDefault()).toInstant());
    }

}
```

