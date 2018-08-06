# Lambda表达式的使用

## 一、lambda表达式的介绍
lambda表达式出现在java8之后
+ 代替匿名内部类使代码不在啰嗦和繁杂
+ 使用lambda表达式对列表进行迭代
+ 使用lambda表达式进行事件处理


## 二、lambda表达式代替匿名内部类
参考文章：[https://www.cnblogs.com/CarpenterLee/p/5978721.html](https://www.cnblogs.com/CarpenterLee/p/5978721.html)

匿名内部类的写法（无参函数）
```java
// JDK7 匿名内部类写法
new Thread(new Runnable(){// 接口名
    @Override
    public void run(){// 方法名
        System.out.println("Thread run()");
    }
}).start();
```
lambda表达式的写法：
```java
//一行代码的时候可以省略大括号
new Thread(
        () -> System.out.println("Thread run()")// 省略接口名和方法名
).start();
//多行代码不省略大括号
// JDK8 Lambda表达式代码块写法
new Thread(
        () -> {
            System.out.print("Hello");
            System.out.println(" Hoolee");
        }
).start();
```

---
匿名内部类的写法（有参函数）
```java
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, new Comparator<String>(){// 接口名
    @Override
    public int compare(String s1, String s2){// 方法名
        if(s1 == null)
            return -1;
        if(s2 == null)
            return 1;
        return s1.length()-s2.length();
    }
});
```
lambda表达式的写法：
```java
List<String> list = Arrays.asList("I", "love", "you", "too");
Collections.sort(list, (s1, s2) ->{// 省略参数表的类型(如果只有一个参数可以() 只有一个return也可以省略return)
    if(s1 == null)
        return -1;
    if(s2 == null)
        return 1;
    return s1.length()-s2.length();
});
```
自定义函数接口：

>// 自定义函数接口
>@FunctionalInterface

加上此标注 编译器会检查接口是否符合函数接口规范就像@Override一样编译器会检查是否是重载函数
## 三、lambda表达式对列表迭代(使用聚合操作)
>参考文章：[http://how2j.cn/k/lambda/lambda-stream/700.html](http://how2j.cn/k/lambda/lambda-stream/700.html)

```java
//传统的根据条件对Heros列表进行遍历
for (Hero h : heros) {
   if (h.hp > 100 && h.damage < 50)
      System.out.println(h.name);
}
//使用lambda表达式进行有条件遍历
heros
	.stream()
	.filter(h -> h.hp > 100 && h.damage < 50)  
	.forEach(h -> System.out.println(h.name));
```
*Stream和管道的概念*

要了解聚合操作，首先要建立Stream和管道的概念
Stream 和Collection结构化的数据不一样，Stream是一系列的元素，就像是生产线上的罐头一样，一串串的出来。
管道指的是一系列的聚合操作。
操作分三部分：
+ 管道源：在这个例子里，源是一个List
+ 中间操作： 每个中间操作，又会返回一个Stream，比如.filter()又返回一个Stream, 中间操作是“懒”操作，并不会真正进行遍历。
结束操作：当这个操作执行后，流就被使用“光”了，无法再被操作。所以这必定是流的最后一个操作。 
+ 结束操作不会返回Stream，但是会返回int、float、String、 Collection或者像forEach，什么都不返回, 结束操作才进行真正的遍历行为，在遍历的时候，才会去进行中间操作的相关判断

*集合或者数组转换为管道源：*

把Collection切换成管道源很简单，调用stream()就行了。
heros.stream()
但是数组却没有stream()方法，需要使用
Arrays.stream(hs) 或者 Stream.of(hs)

*中间操作：*

+ 对元素进行筛选：
+ filter 匹配
+ distinct 去除重复(根据equals判断)
+ sorted 自然排序
+ sorted(Comparator<T>) 指定排序
+ limit 保留
+ skip 忽略

转换为其他形式的流
+ mapToDouble 转换为double的流
+ map 转换为任意类型的流

PS：
```java
package lambda;
  
import java.util.ArrayList;
import java.util.List;
import java.util.Random;
 
import charactor.Hero;
  
public class TestAggregate {
  
    public static void main(String[] args) {
        Random r = new Random();
        List<Hero> heros = new ArrayList<Hero>();
        for (int i = 0; i < 5; i++) {
            heros.add(new Hero("hero " + i, r.nextInt(1000), r.nextInt(100)));
        }
        //制造一个重复数据
        heros.add(heros.get(0));
        System.out.println("初始化集合后的数据 (最后一个数据重复)：");
        System.out.println(heros);
        System.out.println("满足条件hp>100&&damage<50的数据");
          
        heros
            .stream()
            .filter(h->h.hp>100&&h.damage<50)
            .forEach(h->System.out.print(h));
          
        System.out.println("去除重复的数据，去除标准是看equals");
        heros
            .stream()
            .distinct()
            .forEach(h->System.out.print(h));
        System.out.println("按照血量排序");
        heros
            .stream()
            .sorted((h1,h2)->h1.hp>=h2.hp?1:-1)
            .forEach(h->System.out.print(h));
          
        System.out.println("保留3个");
        heros
            .stream()
            .limit(3)
            .forEach(h->System.out.print(h));
          
        System.out.println("忽略前3个");
        heros
            .stream()
            .skip(3)
            .forEach(h->System.out.print(h));
          
        System.out.println("转换为double的Stream");
        heros
            .stream()
            .mapToDouble(Hero::getHp)
            .forEach(h->System.out.println(h));
          
        System.out.println("转换任意类型的Stream");
        heros
            .stream()
            .map((h)-> h.name + " - " + h.hp + " - " + h.damage)
            .forEach(h->System.out.println(h));
          
    }
}
```

*结束操作：*
+ forEach() 遍历每个元素
+ toArray() 转换为数组
+ min(Comparator<T>) 取最小的元素
+ max(Comparator<T>) 取最大的元素
+ count() 总数
+ findFirst() 第一个元素
```java
package lambda;
  
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.Random;
 
import org.omg.Messaging.SYNC_WITH_TRANSPORT;
 
import charactor.Hero;
  
public class TestAggregate {
  
    public static void main(String[] args) {
        Random r = new Random();
        List<Hero> heros = new ArrayList<Hero>();
        for (int i = 0; i < 5; i++) {
            heros.add(new Hero("hero " + i, r.nextInt(1000), r.nextInt(100)));
        }
        System.out.println("遍历集合中的每个数据");
        heros
            .stream()
            .forEach(h->System.out.print(h));
        System.out.println("返回一个数组");
        Object[] hs= heros
            .stream()
            .toArray();
        System.out.println(Arrays.toString(hs));
        System.out.println("返回伤害最低的那个英雄");
        Hero minDamageHero =
        heros
            .stream()
            .min((h1,h2)->h1.damage-h2.damage)
            .get();
        System.out.print(minDamageHero);
        System.out.println("返回伤害最高的那个英雄");
 
        Hero mxnDamageHero =
                heros
                .stream()
                .max((h1,h2)->h1.damage-h2.damage)
                .get();
        System.out.print(mxnDamageHero);     
         
        System.out.println("流中数据的总数");
        long count = heros
                .stream()
                .count();
        System.out.println(count);
 
        System.out.println("第一个英雄");
        Hero firstHero =
                heros
                .stream()
                .findFirst()
                .get();
         
        System.out.println(firstHero);
         
    }
}
```
## 四、学lambda表达式总结的知识

### (1)Random类
使用Random类生成随机数：
```java
Random r = new Random();
System.out.println(r.nextInt(1000));//随机生成1000以内的整数
```
+ public int nextInt(int n) 返回大于等于0且小于n的随机整数
+ public long nextLong() 返回一个随机长整型值
+ public boolean nextBoolean() 返回一个随机布尔型值
+ public float nextFloat() 返回一个随机浮点型值
+ public double nextDouble() 返回一个随机双精度型值
+ public double nextGaussian()返回一个概率密度为高斯分布的双精度值

### (2)Comparable<T>接口
Comparable定义接口的自然排序
>作用：如果数据类型或者List中的元素实现了该接口的话,我们就可以调用Collections.sort或者Arrays方法给他们排序。
>实现该接口要重写他的compareTo方法

重写compareTo方法的示例：即在调用Collections或者Arrays的方法进行该数据类型排序的时候就会调用该方法进行递增或者递减的排序
```java
class A implements Comparable<A>{
    private int rank;  
    @Override
    public int compareTo(A other) {  
        // TODO Auto-generated method stub  
        return Integer.compare(rank, other.rank);  //声明了用什么对该数据类型排序，至于递增或者递减是通过Collections和Arrays来决定的
    } 
}
```

### (3)java使用库函数进行排序
+ Arrays.sort(array); 
+ Collections.sort(list); 
+ Arrays.sort(nameArray,Comparator); 
+ Collections.sort(list,Comparator);
1. 数组递增排序： Arrays.sort(array );
2. 数组递减排序： Arrays.sort(array,Comparator.reverseOrder())
3. 集合递减顺序： Collections.sort(list,Collections.reverseOrder())
4. 集合递增排序： Collections.sort(list);

---

![美腻](https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=830613206,2867543965&fm=27&gp=0.jpg)