# Java8时间类



### LocalDate

本地日期，不包含具体时间 例如：2014-01-14 可以用来记录生日、纪念日、加盟日等。



```java
@Test
public void testLocalDate() {
    LocalDate today = LocalDate.now();
    System.out.println("今天的日期为：" + today);
    System.out.println("年：" + today.getYear());
    System.out.println("月：" + today.getMonth());
    System.out.println("日：" + today.getDayOfMonth());
    System.out.println("明天的日期为" + today.plus(1, ChronoUnit.DAYS));
    System.out.println("明年的日期为" + today.plusYears(1));
    System.out.println("2周后的时间：" + today.plusWeeks(2));
    System.out.println("上个月的日期为" + today.minusMonths(1));
    System.out.println("今天刚开始的时间：" + today.atStartOfDay());
    System.out.println("今天星期几：" + today.getDayOfWeek());
    System.out.println("今年是否是闰年：" + today.isLeapYear());

    LocalDate date1 = LocalDate.of(2018, 01, 14);
    System.out.println("指定的的日期为：" + date1);
    System.out.println("指定的的日期是否在今天之前：" + date1.isBefore(today));
    System.out.println("指定的的日期是不是今天：" + date1.equals(today));
    LocalDate dateOfBirth = LocalDate.of(2016, 01, 14);
    MonthDay birthday = MonthDay.of(dateOfBirth.getMonth(), dateOfBirth.getDayOfMonth());
    MonthDay currentMonthDay = MonthDay.from(date1);
    System.out.println("是否是同月同日：" + birthday.equals(currentMonthDay));
}
```

输出：

```
今天的日期为：2018-12-17
年：2018
月：DECEMBER
日：17
明天的日期为2018-12-18
明年的日期为2019-12-17
2周后的时间：2018-12-31
上个月的日期为2018-11-17
今天刚开始的时间：2018-12-17T00:00
今天星期几：MONDAY
今年是否是闰年：false
指定的的日期为：2018-01-14
指定的的日期是否在今天之前：true
指定的的日期是不是今天：false
是否是同月同日：true
```





### LocalTime



获取时间使用的是LocalTime类，一个只有时间没有日期的LocalDate近亲。可以调用静态工厂方法now()来获取当前时间。默认的格式是hh:mm:ss:nnn。

```java
@Test
public void testLocalTime() {
    LocalTime time = LocalTime.now();
    System.out.println("现在的时间是：" + time);
    System.out.println("几点：" + time.getHour());
    System.out.println("几分：" + time.getMinute());
    System.out.println("几秒：" + time.getSecond());
    System.out.println("几纳秒【1纳秒(ns)=1e-6毫秒(ms)】：" + time.getNano());
    System.out.println("下一个分钟的时间是：" + time.plusMinutes(1));
    System.out.println("上一钟头的时间是：" + time.minus(1, ChronoUnit.HOURS));
    //构造时间
    LocalTime time1 = LocalTime.of(11, 20, 55);
    System.out.println("构造的时间是：" + time1);
    System.out.println("构造的时间是否在现在时间之前：" + time1.isBefore(time));
}
```

输出：

```
现在的时间是：21:48:03.914
几点：21
几分：48
几秒：3
几纳秒【1纳秒(ns)=1e-6毫秒(ms)】：914000000
下一个分钟的时间是：21:49:03.914
上一钟头的时间是：20:48:03.914
构造的时间是：11:20:55
构造的时间是否在现在时间之前：true
```



### Clock

Java 8增加了一个Clock时钟类用于获取当时的时间戳，或当前时区下的日期时间信息。以前用到System.currentTimeInMillis()和TimeZone.getDefault()的地方都可用Clock替换。

```java
@Test
public void testClock() {
    Clock clock = Clock.systemUTC();
    System.out.println("获取时区：" + clock.getZone());
    System.out.println("获取时间戳：" + clock.millis());
}
```

输出：

```
获取时区：Z
获取时间戳：1545054503504
```



### LocalDateTime

LocalDateTime：组合了日期和时间，但不包含时差和时区信息。

```java
@Test
public void testLocalDateTime() {
    LocalDateTime now = LocalDateTime.now();
    System.out.println("当前具体时间：" + now);

    LocalDateTime datetime = LocalDateTime.of(2018, Month.JANUARY, 14, 19, 30, 8, 11);
    System.out.println("构造时间：" + datetime);
    System.out.println("当前具体时间是否在构造时间之前：" + now.isBefore(datetime));
    System.out.println("根据LocalDateTime获取时间戳：" + now.toInstant(ZoneOffset.of("+8")).toEpochMilli());

}
```

输出

```
当前具体时间：2018-12-17T21:48:48.049
构造时间：2018-01-14T19:30:08.000000011
当前具体时间是否在构造时间之前：false
根据LocalDateTime获取时间戳：1545054528049
```





### Instant 

Instant用于“时间戳”的运算。它是以Unix元年(传统的设定为UTC时区1970年1月1日午夜时分)开始所经历的描述进行运算。

```java
@Test
public void testInstant() {
    Instant timestamp = Instant.now();
    System.out.println("当前具体时间：" + timestamp);
    System.out.println("时间戳:" + timestamp.toEpochMilli());
    Instant timestamp2 = timestamp.plusSeconds(200);
    System.out.println("时间差（秒）:" + (timestamp2.getEpochSecond() - timestamp.getEpochSecond()));
}
```

输出：

```
当前具体时间：2018-12-17T13:50:41.728Z
时间戳:1545054641728
时间差（秒）:200
```





### Duration

Duration:用于计算两个“时间”间隔

```java
@Test
public void testDuration() {
    Instant inst1 = Instant.now();
    System.out.println("时间1:" + inst1);
    Instant inst2 = inst1.plus(Duration.ofSeconds(10));
    System.out.println("时间2:" + inst2);

    System.out.println("相差的时间(ms):" + Duration.between(inst1, inst2).toMillis());

    System.out.println("相差的时间(s):" + Duration.between(inst1, inst2).getSeconds());
}
```

输出：

```
时间1:2018-12-17T13:51:16.085Z
时间2:2018-12-17T13:51:26.085Z
相差的时间(ms):10000
相差的时间(s):10
```





###Period

 Period用于计算两个“日期”间隔

```java
@Test
public void testPeriod() {
    LocalDate today = LocalDate.now();
    System.out.println("Today : " + today);
    LocalDate birthDate = LocalDate.of(1993, Month.OCTOBER, 19);
    System.out.println("BirthDate : " + birthDate);

    Period p = Period.between(birthDate, today);
    System.out.printf("年龄 : %d 年 %d 月 %d 日", p.getYears(), p.getMonths(), p.getDays());
}
```

输出：

```
Today : 2018-12-17
BirthDate : 1993-10-19
年龄 : 25 年 1 月 28 日
```



### ChronoUnit

ChronoUnit计算时间间隔的类

```java
@Test
public void testChronoUnit() {
    LocalDate startDate = LocalDate.of(1993, Month.OCTOBER, 19);
    System.out.println("开始时间  : " + startDate);
    LocalDate endDate = LocalDate.of(2017, Month.JUNE, 16);
    System.out.println("结束时间 : " + endDate);
    long daysDiff = ChronoUnit.DAYS.between(startDate, endDate);
    System.out.println("两天之间的差在天数   : " + daysDiff);
}
```

输出：

```
开始时间  : 1993-10-19
结束时间 : 2017-06-16
两天之间的差在天数   : 8641
```



### 时间转换

1.将LocalDateTime转为自定义的时间格式的字符串

```java
public static String getDateTimeAsString(LocalDateTime localDateTime, String format) {
    DateTimeFormatter formatter = DateTimeFormatter.ofPattern(format);
    return localDateTime.format(formatter);
}
```



2.将long类型的timestamp转为LocalDateTime

```java
public static LocalDateTime getDateTimeOfTimestamp(long timestamp) {
    Instant instant = Instant.ofEpochMilli(timestamp);
    ZoneId zone = ZoneId.systemDefault();
    return LocalDateTime.ofInstant(instant, zone);
}
```



3.将LocalDateTime转为long类型的timestamp

```java
public static long getTimestampOfDateTime(LocalDateTime localDateTime) {
    ZoneId zone = ZoneId.systemDefault();
    Instant instant = localDateTime.atZone(zone).toInstant();
    return instant.toEpochMilli();
}
```




4.将某时间字符串转为自定义时间格式的LocalDateTime

```java
public static LocalDateTime parseStringToDateTime(String time, String format) {
    DateTimeFormatter df = DateTimeFormatter.ofPattern(format);
    return LocalDateTime.parse(time, df);
}
```



5.把一个String类型的时间转换成Date类型（会有异常抛出）

```java
try {
    String times = "2016-11-18";
    SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
    Date date = sdf.parse(times);
    System.out.println(date);
} catch (ParseException e) {
    // TODO Auto-generated catch block
    e.printStackTrace();
}
```



6.转换详细时间(老API)

```java
SimpleDateFormat df = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");//设置日期格式
System.out.println(df.format(new Date()));// new Date()为获取当前系统时间
```



7.LocalDateTime与String相互转化

```java
    @Test
    public void t1() {
        DateTimeFormatter df = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
        LocalDateTime time = LocalDateTime.now();
        String localTime = df.format(time);
        LocalDateTime ldt = LocalDateTime.parse("2017-09-28 17:07:05",df);
        System.out.println("LocalDateTime转成String类型的时间："+localTime);
        System.out.println("String类型的时间转成LocalDateTime："+ldt);
    }
```

```
LocalDateTime转成String类型的时间：2018-12-17 22:01:01
String类型的时间转成LocalDateTime：2017-09-28T17:07:05
```

