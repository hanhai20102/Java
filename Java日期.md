# Java日期类

## 一、Date
java.util.Date不是java.sql.date(给数据库访问的时候用的)

### 创建日期对象
+ 通过new Date()对象创建当前时间的Date
+ 通过从1970年1月1号8点到现在的毫秒数new Date(System.currentTimeMillis())创建一个Date对象
```java
import java.util.Date;
public class TestDate {
 
    public static void main(String[] args) {
        // 当前时间
        Date d1 = new Date();
        System.out.println("当前时间:");
        System.out.println(d1);
        System.out.println();
        // 从1970年1月1日 早上8点0分0秒 开始经历的毫秒数
        Date d2 = new Date(5000);
        System.out.println("从1970年1月1日 早上8点0分0秒 开始经历了5秒的时间");
        System.out.println(d2);
    }
}
```
### 格式化日期
使用SimpleDateFormat格式化日期
| 关键字 | 简介 |
| :-: | :-: |
| format | 日期转字符串 |
| parse | 字符串转日期 |
**格式化日期每个字符代表的含义：**
+ y 代表年
+ M 代表月
+ d 代表日
+ H 代表24进制的小时
+ h 代表12进制的小时
+ m 代表分钟
+ s 代表秒
+ S 代表毫秒
示例代码：
```java
 SimpleDateFormat sdf =new SimpleDateFormat("yyyy-MM-dd HH:mm:ss SSS" );
        Date d= new Date();
        String str = sdf.format(d);
        System.out.println("当前时间通过 yyyy-MM-dd HH:mm:ss SSS 格式化后的输出: "+str);
         
        SimpleDateFormat sdf1 =new SimpleDateFormat("yyyy-MM-dd" );
        Date d1= new Date();
        String str1 = sdf1.format(d1);
        System.out.println("当前时间通过 yyyy-MM-dd 格式化后的输出: "+str1);
```
```java
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
public class TestDate {
  
    public static void main(String[] args) {
        SimpleDateFormat sdf =new SimpleDateFormat("yyyy/MM/dd HH:mm:ss" );
  
        String str = "2016/1/5 9:2:12";
          
        try {
            Date d = sdf.parse(str);
            System.out.printf("字符串 %s 通过格式  yyyy/MM/dd HH:mm:ss %n转换为日期对象: %s",str,d.toString());
        } catch (ParseException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
          
    }
}
```
## Calendar

### Calendar类与Date类相互转换
```java
Calendar c= Calendar.getInstance();
Date d = c.getTime();
Date d1 = new Date(0);
c.setTime(d1);  //把这个日历，调成日期 : 1970.1.1 08:00:00
```

### 翻日历
+ add方法，在原日期上增加年/月/日
+ set方法，直接设置年/月/日
```java
import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
 
public class TestDate {
 
    private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
 
    public static void main(String[] args) {
        Calendar c = Calendar.getInstance(); //创建了当前时间的日历
        Date now = c.getTime();
        // 当前日期
        System.out.println("当前日期：\t" + format(c.getTime()));
 
        // 下个月的今天
        c.setTime(now);
        c.add(Calendar.MONTH, 1);
        System.out.println("下个月的今天:\t" +format(c.getTime()));
 
        // 去年的今天
        c.setTime(now);
        c.add(Calendar.YEAR, -1);
        System.out.println("去年的今天:\t" +format(c.getTime()));
 
        // 上个月的第三天
        c.setTime(now);
        c.add(Calendar.MONTH, -1);
        c.set(Calendar.DATE, 3);
        System.out.println("上个月的第三天:\t" +format(c.getTime()));
 
    }
    private static String format(Date time) {
        return sdf.format(time);
    }
}
```
