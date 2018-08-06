# Java异常处理

## 一、异常介绍
导致程序的正常流程被中断的事件，叫做**异常**--Exception

Exception继承与Throwable类

![http://how2j.cn/k/exception/exception-custom/337.html#nowhere](http://how2j.cn/frontstepImage?stepid=742)

*Exception*：
+ 编译时异常 必须进行处理抛出或者通过try-catch-finally块处理 如：IOException、SQLException
+ 运行时异常 RunTimeException一般由程序的逻辑错误引起的，一般不进行处理，处理会显得代码太冗杂，如：NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)

*Error*
+ AWTError
+ VirtualMachineeRROR JVM选择终止线程 如：内存资源不足

**异常和错误的区别：异常能被程序本身可以处理，错误是无法处理。**

## 二、异常处理

### (1)通过try-catch-finally块处理
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
  
public class TestException {
  
    public static void main(String[] args) {
          
        File f= new File("d:/LOL.exe");
          
        try{
            System.out.println("试图打开 d:/LOL.exe");
            new FileInputStream(f);
            System.out.println("成功打开");
        }
         
        catch(Exception e){
            System.out.println("d:/LOL.exe不存在");
            e.printStackTrace();
        }
          
    }
}

```
+ 使用try-catch块捕捉异常，可以捕获具体类的异常也可以捕获处理父类的异常，也可以都处理，但是要先处理子类异常，再处理父类异常。
+ e.printStackTrace()函数 会打印异常发生时函数处理的痕迹，显示异常发生时的信息，方便程序员调试。
+ 使用try-catch块捕获多种异常时，一般写一个try-后跟多个catch块即可，然后分别对相应的异常对象做处理
+ 使用try-catch块捕获多种异常的另一种方法如下代码所示，在处理对应异常时要判断异常对象是哪一个异常类的实例再分别进行处理，此方式是java7之后才有的。
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.text.ParseException;
import java.text.SimpleDateFormat;
import java.util.Date;
 
public class TestException {
 
    public static void main(String[] args) {
 
        File f = new File("d:/LOL.exe");
 
        try {
            System.out.println("试图打开 d:/LOL.exe");
            new FileInputStream(f);
            System.out.println("成功打开");
            SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd");
            Date d = sdf.parse("2016-06-03");
        } catch (FileNotFoundException | ParseException e) {
            if (e instanceof FileNotFoundException)
                System.out.println("d:/LOL.exe不存在");
            if (e instanceof ParseException)
                System.out.println("日期格式解析错误");
 
            e.printStackTrace();
        }
 
    }
}
```
使用*finally*块时应注意：
(一)finally块中的代码一定可以执行但是有两种情况：
+ 在finally块中的代码执行之前执行了System.exit();
+ 在try块执行之前已经产生了异常则不会执行try-catch-finally块中的异常
(二)finally块的return相关
+ 在finally块执行之前return语句中的表达式的值已经计算好并保存
执行finally块的代码不能改变将要return的值
+ finally块最好不要return，容易使程序提前退出

---
### (2)通过throws或者throw抛出
#### throws
代码解释：method2中需要进行异常处理但是method2不打算处理，而是把这个异常通过throws抛出去那么method1就会接到该异常。 处理办法也是两种，要么是try catch处理掉，要么也是抛出去。
method1选择本地try catch住 一旦try catch住了，就相当于把这个异常消化掉了，主方法在调用method1的时候，就不需要进行异常处理了。

**因此：只有方法try-catch块处理了异常，上一层就不用抛出或者处理异常了，也就是这个异常消失了或者说是被消化掉了。**
```java
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
 
public class TestException {
 
    public static void main(String[] args) {
        method1();
 
    }
 
    private static void method1() {
        try {
            method2();
        } catch (FileNotFoundException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
 
    }
 
    private static void method2() throws FileNotFoundException {
 
        File f = new File("d:/LOL.exe");
 
        System.out.println("试图打开 d:/LOL.exe");
        new FileInputStream(f);
        System.out.println("成功打开");
 
    }
}

```
#### throw
throw用于方法体中用于手动抛出异常。在使用throw是要在方法体通过throws向上抛出，让上一级的方法继续向上抛出或者处理消化掉这个异常。

throws与throw这两个关键字接近，不过意义不一样，有如下区别：
1. throws 出现在方法声明上，而throw通常都出现在方法体内。
2. throws 表示出现异常的一种可能性，并不一定会发生这些异常；throw则是抛出了异常，执行throw则一定抛出了某个异常对象。

## 三、throwable类
在捕获异常时可以用父类代替，因此也可以捕获throwable类的异常，但是不能具体判断出现了何种异常!
>代码示例：
```java
import java.io.File;
import java.io.FileInputStream;
 
public class TestException {
 
    public static void method() throws Throwable {
        File f = new File("d:/LOL.exe");
        new FileInputStream(f);
    }
 
    public static void main(String[] args) {
        try {
            method();
        } catch (Throwable e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```

## 四、自定义异常
可以通过继承Exception类自定义异常
>代码示例：
```java
class EnemyHeroIsDeadException extends Exception{
     
    public EnemyHeroIsDeadException(){
         
    }
    public EnemyHeroIsDeadException(String msg){
        super(msg);
    }
}
```
在使用时可以通过throw new EnemyHeroIsDeadException(msg);来抛出自定义的这个异常，再由上一级方法捕获此异常进行处理或者抛出
>参考文章：[http://how2j.cn/k/exception/exception-custom/337.html#nowhere](http://how2j.cn/k/exception/exception-custom/337.html#nowhere)