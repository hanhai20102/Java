# Java 零碎知识
1. Java对象一定要初始化，不能只声明！
2. \r\n表示在Windows系统下的换行符
3. String的contains方法可以判断字符串是否包含了另一个字符串
4. StringBuffer 线程安全但是速度没有StringBuild快，StringBuild线程不安全
5. String的substring方法截取从beginIndex下标开始待endIndex下标的前一个字符之间的字符串
6. String的index方法返回指定字符串第一个字符的下标以及lastIndexOf返回最后一次出现的字符串的第一个字符的下标
7. *Collections.shufffle方*法对指定的列表进行随机替换    Collections.shuffle(list);
8. Map的遍历
```java
 Iterator<Map.Entry<String,String>> it=map.entrySet().iterator();
        while (it.hasNext()){
            Map.Entry<String,String> entry=it.next();
            System.out.println("the key is:"+entry.getKey()+",and the value is:"+entry.getValue());
        }
```
9. List或者其它集合只要初始化了之后就不可能为null，因此不能根据此来判断业务逻辑
10. Map在get()方法时get不到则对象为null。假设为Map<String,Object> get后强制转换为String，则String对象为null。
11. Intger等包装类的初始值为null
12. 数据流有readUTF()之类的方法、缓存流有readLine()方法、对象流有readObject()之类的方法 FileInputStram等只有read/writede 方法。
13. Java采用Unicode编码，即每个字符占用4个字节，即太浪费资源，因此会细化即出现了utf-8等编码，char类型在java中占用两个字节使用的是utf-16的方式，不论是中文还是英文都是占两个字节，而utf-8英文占用一个字节，中文占三个字节。在JVM的内部会细化

常见的编码有：
+ ISO-8859-1 使用AscII编码 所有字符均占一个字节 适用于数字和字母
+ GBK GB2312 BIG5 中文 GB2312 是简体中文，BIG5是繁体中文，GBK同时包含简体和繁体以及日文。
+ Unicode编码 万国码，统一码

14. 使用FileInputStream正确读取中文 读取到的字节数组可以以new String(byte,encode)的形式读取到中文
15. FileReader是不能手动设置编码方式的，为了使用其他的编码方式，只能使用InputStreamReader来代替，像这样：
```java

package stream;
 
import java.io.File;
import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.UnsupportedEncodingException;
import java.nio.charset.Charset;
 
public class TestStream {
 
    public static void main(String[] args) throws UnsupportedEncodingException, FileNotFoundException {
        File f = new File("E:\\project\\j2se\\src\\test.txt");
        System.out.println("默认编码方式:"+Charset.defaultCharset());
        //FileReader得到的是字符，所以一定是已经把字节根据某种编码识别成了字符了
        //而FileReader使用的编码方式是Charset.defaultCharset()的返回值，如果是中文的操作系统，就是GBK
        try (FileReader fr = new FileReader(f)) {
            char[] cs = new char[(int) f.length()];
            fr.read(cs);
            System.out.printf("FileReader会使用默认的编码方式%s,识别出来的字符是：%n",Charset.defaultCharset());
            System.out.println(new String(cs));
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        //FileReader是不能手动设置编码方式的，为了使用其他的编码方式，只能使用InputStreamReader来代替
        //并且使用new InputStreamReader(new FileInputStream(f),Charset.forName("UTF-8")); 这样的形式
        try (InputStreamReader isr = new InputStreamReader(new FileInputStream(f),Charset.forName("UTF-8"))) {
            char[] cs = new char[(int) f.length()];
            isr.read(cs);
            System.out.printf("InputStreamReader 指定编码方式UTF-8,识别出来的字符是：%n");
            System.out.println(new String(cs));
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
         
    }
}
```
16. Integer与String的相互转换，必须是数字类型不然会抛异常
17. 接口和抽象类都不能new，但是可以写成匿名内部类的形式假装是new的一个对象，当有很多种不同的角色有共同的功能和不同的功能即可使用抽象类进行扩展，不同的功能用抽象方法，相同的功能使用普通方法。而一些角色有的一些没得，可以在自己的类中实现此功能进行拓展，接口中全是抽象方法，因此实现接口必须全部重写接口中的抽象方法。

接口：常用与功能的扩展，实现常用的功能
抽象类： 公共类的角色，不适用于日后重新对里边的类进行修改

抽象类可以有main方法。接口可以继承接口，抽象类可以实现接口，抽象类也可以继承具体类。
18. **Listener的作用**是监听web应用的创建与销毁以及attribute的变化
web应用即为ServletContext对象，JSP的隐式对象application对象，
还能监听session和request对象的生命周期
spring的ContextLoaderListener就是实现了ServletContextListener接口，来监听web应用的创建与销毁
自定义listener后在web.xml文件的listrner标签配置即可
19. 服务器server.xml文件中context标签的reloadable属性控制web应用的自动重载，在某个servlet改变的时候自动重载
20. 对context的监听还可以监听context属性的改变，即实现ServletContextAttributeListener接口，重写attributeAdded、attributeRemoved、attributeReplaced三个方法即可在context对象的属性改变的时候触发。
21. 监听器对session的监听分为对session生命周期的监听和对session对象的属性监听。对session生命周期的监听实现HttpSessionListener接口，重写sessionDestroyed和sessionCreated方法即可而且可以获取到session的ID。通过HttpSessionEvent
```java
package listener;
 
import javax.servlet.http.HttpSessionEvent;
import javax.servlet.http.HttpSessionListener;
 
public class SessionListener implements HttpSessionListener {
 
    @Override
    public void sessionCreated(HttpSessionEvent e) {
 
        System.out.println("监听到 session 创建, sessionid 是： " + e.getSession().getId());
    }
 
    @Override
    public void sessionDestroyed(HttpSessionEvent e) {
 
        System.out.println("监听到 session 销毁, sessionid 是： " + e.getSession().getId());
    }
}
```
对session属性改变的监听：实现HttpSessionAttributeListener接口重写以下三个方法。就可以在session对象的属性值改变的时候触发监听器的事件处理。
```java
package listener;
 
import javax.servlet.http.HttpSessionAttributeListener;
import javax.servlet.http.HttpSessionBindingEvent;
 
public class SessionAttributeListener implements HttpSessionAttributeListener {
 
    @Override
    public void attributeAdded(HttpSessionBindingEvent e) {
 
        System.out.println("session 增加属性 ");
        System.out.println("属性是" + e.getName());
        System.out.println("值是" + e.getValue());
 
    }
 
    @Override
    public void attributeRemoved(HttpSessionBindingEvent e) {
        // TODO Auto-generated method stub
        System.out.println("session 移除属性 ");
 
    }
 
    @Override
    public void attributeReplaced(HttpSessionBindingEvent e) {
        // TODO Auto-generated method stub
        System.out.println("session 替换属性 ");
 
    }
}
```
22. 监听器对request对象的监听也是分为相似的两个部分。
而其实现的接口也大同小异ServletRequestListener, ServletRequestAttributeListener。用法一致。
23. String,Integer等包装类都是不可变类，因为其成员变量都是私有的且final，当给一个Integer赋值时，是不能改变该对象的成员变量的，改变对象的值相当于新建了一个新的Integer对象，String也一样。
24. **不可变类**的设计方法：
* 类用final修饰
* 成员私有且final
* 不提供改变的成员的方法，如setter方法
* 通过构造器初始化所有成员，进行深拷贝
```java
public final class MyImmutableDemo {  
    private final int[] myArray;  
    public MyImmutableDemo(int[] array) {  
        this.myArray = array.clone();   
    }   
}
```
25. 使用静态工厂方法代替构造器比如Integer的valueOf方法，使用预先创建好的实例，或者缓存起来
26. 静态的成员方法只能调用其他的静态方法和静态变量不能调用类中其他的非静态成员
27. 可以扩充数组的容量 使用Arrays.copyof(数组名，扩展的长度)
28. 接口中的属性都是公共的，静态的，final的
29. 计算机中使用补码表示负数。正数的原码补码和反码都一样，负数的反码是符号位不变，其余各个位取反，补码是反码+1符号位不变，简便算法：在原码的基础上找出两边第一个不为0的数，中间每位取反
30. Java只有左移、右移和无符号右移

- 无符号右移即为 右移左补0
- 正数的左移还是右移都是补0  
- 负数的左移右补0，右移左补1
- 左移扩大2倍
- 没有无符号左移是因为牵扯符号位 

31. leftJoin与rightJoin的区别就在leftjion返回左表的全部数据和右表满足条件的数据，rightJoin则相反
32. Arrays.copyOf()实现动态数组的关键，形参：原数组，原数组起始位置，新数组，新数组的起始位置，复制的长度
33. ArrayList的扩容是每次扩展为原数组的1.5倍
34. idea单步调试F7，F8，F9
35. char和int之间的转换需要用c-'0'就可以得到本来的int值了
36. main方法还可以用final、synchronized修饰
37. 
类修饰符：

public（访问控制符），将一个类声明为公共类，他可以被任何对象访问，一个程序的主类必须是公共类。

abstract，将一个类声明为抽象类，没有实现的方法，需要子类提供方法实现。

final，将一个类生命为最终（即非继承类），表示他不能被其他类继承。

friendly，默认的修饰符，只有在相同包中的对象才能使用这样的类。

当类不适用修饰符时，为默认访问模式，此类只能在本包中访问或引用

成员变量修饰符：

public（公共访问控制符），指定该变量为公共的，他可以被任何对象的方法访问。

private（私有访问控制符）指定该变量只允许自己的类的方法访问，其他任何类（包括子类）中的方法均不能访问。

protected（保护访问控制符）指定该变量可以别被自己的类和子类访问。在子类中可以覆盖此变量。

friendly ，在同一个包中的类可以访问，其他包中的类不能访问。

final，最终修饰符，指定此变量的值不能变。

static（静态修饰符）指定变量被所有对象共享，即所有实例都可以使用该变量。变量属于这个类。

transient（过度修饰符）指定该变量是系统保留，暂无特别作用的临时性变量。

volatile（易失修饰符）指定该变量可以同时被几个线程控制和修改。

  

方法修饰符：

public（公共控制符）

private（私有控制符）指定此方法只能有自己类等方法访问，其他的类不能访问（包括子类）

protected（保护访问控制符）指定该方法可以被它的类和子类进行访问。

final，指定该方法不能被重载。

static，指定不需要实例化就可以激活的一个方法。

synchronize，同步修饰符，在多个线程中，该修饰符用于在运行前，对他所属的方法加锁，以防止其他线程的访问，运行结束后解锁。

native，本地修饰符。指定此方法的方法体是用其他语言在程序外部编写的。

**Java接口**

- 接口修饰
public abstract
- 方法
public abstract
- 成员变量 
public static final 必须赋初值

38. 普通函数可以与构造函数有相同的名字

### 内部类
内部类可以用public,protected,default,private，static,final修饰
- 静态内部类
不依赖外部类的实例化而被实例化，不能与外部类有相同的名字。只能访问外部类的静态成员和静态方法。
- 成员内部类
成员内部类不能定义静态的变量和方法，但可以自由的引用外部的成员和方法。依靠外部类实例化而实例化
- 局部内部类
代码块的内部类。只能访问方法中定义为final类型的变量。一般使用匿名内部类代替
- 匿名内部类
没有类名的内部类，没有构造函数，必须集成其他类或者实现其他接口。一般在继承抽象类或者实现接口时使用并赋值给接口变量。匿名内部类一定要在new的后边 。因为是在代码块中实现，所以不能使用修饰限定符修饰。

### 关于super关键字的理解

1. super主要用来调用父类中被子类覆盖的方法。
2. 在子类的构造函数的应用
当父类中定义了有参的构造函数时，子类的构造函数必须显示的调用父类的构造函数

当父类没有定义有参的构造函数，编译器默认加入一个无参构造函数，子类的构造函数中super();写不写都无所谓的。实例化一个子类的对象时，调用父类的构造函数主要是为了初始化父类中的成员。

### Java跳出多重循环

1. 使用标识
```java
 public static void main(String[] args) {
        out:
        for (int i=0;i<10;i++){
            for (int j=0;j<10;j++){
                if(j==5){
                    break out;
                }
            }
        }
    }
```
2. 抛出异常强制跳出
3. 外层循环的条件受内层循环控制，在外层循环的循环条件中添加标志位

### final的用法

- final修饰变量
变量必须被初始化，可以在定义的时候初始化，final成员变量可以在初始化块初始化但不能在静态块中初始化，静态final类型可以在静态块中初始化，但不能在初始化块初始化，可以在类的构造器中初始化，但是静态final成员不能在构造函数中初始化。

指的是引用的不可变性，对象的属性可以改变。
- final修饰方法（子类不允许重写）
- final修饰类（类不能被继承）

### strictfp关键字 
被strictfp关键字修饰的类，方法java编译器会完全按照IEEE二进制浮点数算数标准执行。

### volatile关键字的两个语义

1. 防止指令重排序
2. 可见性
3. 不能保证原子性