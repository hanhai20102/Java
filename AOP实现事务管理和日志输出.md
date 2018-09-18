# AOP实现事务管理和日志输出

## slf4j的使用

**为什么要使用slf4j**
1. slf4j是一个抽象日志接口，允许后台使用任何一个日志类库，依赖于具体的日志实现，类似于抽象类，可以使用slf4j和log4j一起使用。独立于任何一个特定的日志API
2. 占位符{}，运行时被某个特定的字符串替换
3. 如果使用具体的日志实现API，在更改日志的实现的时候就需要修改大量的代码，因此推荐使用抽象日志接口slf4j。

**logger必须作为静态变量使用的原因**

    log记录的是当前类的日志，不是每个实例的日志因此使用一个logger对象就可以了

**日志的等级**
- error    指出虽然发生错误事件，但仍然不影响系统的继续运行。
- warn    表明会出现潜在的错误情形。
- info      程序运行的步骤信息
- debug  程序的调试信息

**slf4j的使用**
1. 导入依赖的jar包
```java
  <!-- slf4j 依赖包 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.5</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-api</artifactId>
            <version>2.0-rc1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.logging.log4j</groupId>
            <artifactId>log4j-core</artifactId>
            <version>2.0-rc1</version>
        </dependency>
```
2. 配置文件使用Log4J的配置文件
```java
log4j.rootLogger=debug, C

log4j.appender.A=org.apache.log4j.ConsoleAppender
log4j.appender.A.layout=org.apache.log4j.PatternLayout
log4j.appender.A.layout.ConversionPattern=%d{yyyy-MM-dd HH:mm:ss} [%c]-[%p] %m%n

log4j.appender.B=org.apache.log4j.FileAppender  //日志输出到一个文件中，
log4j.appender.B.File=E:\\log.log
log4j.appender.B.layout=org.apache.log4j.SimpleLayout

log4j.appender.C=org.apache.log4j.RollingFileAppender //RollingFileAppender按log文件最大长度限度生成新文件当文件大小超过1000KB时，将原来的文件更名为portal.log.1，再使用portal.log接收新的日志记录。
log4j.appender.C.File=E:\\log.html
log4j.appender.C.MaxFileSize=1000KB
log4j.appender.C.MaxBackupIndex=10
log4j.appender.C.layout=org.apache.log4j.HTMLLayout
log4j.appender.C.encoding=gbk

log4j.appender.D=org.apache.log4j.DailyRollingFileAppender  //按日期生成新文件。比如今天是2010-01-13, 到明天这个文件将更名为portal.log2010-01-13.log
log4j.appender.D.File=E:\\log.log
log4j.appender.D.layout=org.apache.log4j.TTCCLayout
```
3. 在类中直接@slf4j导入logger对象，或者
```java
private static Logger log = LoggerFactory.getLogger(Slf4jTest.class);
```
4. 使用样例
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class Slf4jTest {
    private static Logger logger = LoggerFactory.getLogger(Slf4jTest.class);// slf4j日志记录器

    public static void main(String[] args) {

        // 普通的日志记录
        logger.debug("普通的日志记录");

        // {}占位符记录日志
        for (int i = 0; i < 3; i++) {
            logger.debug("这是第{}条记录", i);
        }

        // 用\转义{}
        logger.debug("Set \\{} differs from {}", "3"); // output:Set {} differs
                                                        // from 3

        // 两个参数
        logger.debug("两个占位符，可以传两个参数{}----{}", 1, 2);

        // 多个参数(可变参数)
        logger.debug("debug:多个占位符，{},{},{},{}", 1, 2, 3, 4);

        // 多个参数(可变参数)
        logger.info("info:多个占位符，{},{},{},{}", 1, 2, 3, 4);

        // 多个参数(可变参数)
        logger.error("error:多个占位符，{},{},{},{}", 1, 2, 3, 4);

    }

}
```
- 当捕捉到异常的时候Exception对象作为日志记录的最后一个参数（会显示具体的出错信息以及出错位置），而且要放在{}可以格式化的参数之外，防止被{}转为e.toString()

- 放到{}之外的非最后一个参数不会打印错误信息

```java
public class Slf4jTest {
    private static Logger log = LoggerFactory.getLogger(Slf4jTest.class);
    public static void main(String[] args) {
        openFile("xxxxxx");
    }
    public static void openFile(String filePath) {
        File file = new File(filePath);
        try {
            InputStream in = new FileInputStream(file);
        } catch (FileNotFoundException e) {
        //下面这两种效果一样
            log.error("can found file [{}]", filePath, e);
            log.error("can found file [{}],cause:{}", filePath, e.toString()); 

        }
    }
}
```
## AOP前置后置环绕通知
**JointPoint**
连接点类，通过这个对象可以访问连接点的细节
- getSignature 可以获得连接点的方法签名对象
- getTarget 可以获得连接点所在的目标对象
- getArgs（） 获取连接点方法运行时的参数列表
- getThis（）获取代理对象本身

前置通知：目标方法调用前进行通知
后置通知：目标方法调用后进行通知
环绕通知：编写的逻辑将被通知的方法包装起来，也可以不调用即阻塞环绕通知。
```java
 @Around("performance()")
        public void watchPerformance(ProceedingJoinPoint joinPoint){
            try {
                System.out.println("Silencing cell phones");
                System.out.println("Taking seats");
                joinPoint.proceed();  //调用被通知的方法
                System.out.println("Demanding a refund");
            } catch (Throwable throwable) {
                throwable.printStackTrace();
            }
        }
```
定义通知的时候在通知方法中添加了入参：ProceedingJoinPoint。在创建环绕通知的时候，这个参数是必须写的。因为在需要在通知中使用ProceedingJoinPoint.proceed()方法调用被通知的方法。当然也可以不调用被通知的方法。

**切面的通知顺序**：可以让切面类实现ordered接口，重写getOrder（）方法，返回排序值，order值越小就先通知。

## 使用自定义注解实现日志输出

1. 自定义注解只是为了在切面中书中注解总的信息
```java
 5 @Target({ElementType.PARAMETER, ElementType.METHOD})  
 6 @Retention(RetentionPolicy.RUNTIME)  
 7 @Documented  
 8 public @interface Log {
 9 
10     /** 要执行的操作类型比如：add操作 **/  
11     public String operationType() default "";  
12      
13     /** 要执行的具体操作比如：添加用户 **/  
14     public String operationName() default "";
15 }
```
2. 编写切面类
```java
1 package com.gcx.annotation;
  2 
  3 import java.lang.reflect.Method;
  4 import java.util.Date;
  5 import java.util.UUID;
  6 
  7 import javax.annotation.Resource;
  8 import javax.servlet.http.HttpServletRequest;
  9 import javax.servlet.http.HttpSession;
 10 
 11 import org.aspectj.lang.JoinPoint;
 12 import org.aspectj.lang.ProceedingJoinPoint;
 13 import org.aspectj.lang.annotation.After;
 14 import org.aspectj.lang.annotation.AfterReturning;
 15 import org.aspectj.lang.annotation.AfterThrowing;
 16 import org.aspectj.lang.annotation.Around;
 17 import org.aspectj.lang.annotation.Aspect;
 18 import org.aspectj.lang.annotation.Before;
 19 import org.aspectj.lang.annotation.Pointcut;
 20 import org.slf4j.Logger;
 21 import org.slf4j.LoggerFactory;
 22 import org.springframework.stereotype.Component;
 23 
 24 import com.gcx.entity.SystemLog;
 25 import com.gcx.entity.User;
 26 import com.gcx.service.SystemLogService;
 27 import com.gcx.util.JsonUtil;
 28 
 35 
 36 @Aspect  //标记为一个切面类（需要在配置文件中开启基于注解的切面支持<aop:aspectj-autoProxy proxy-target-class="true">）
 37 @Component  //标记该类交由spring管理，就不用使用xml文件配置该bean了
 38 public class SystemLogAspect {
 39 
 40     //注入Service用于把日志保存数据库  
 41     @Resource  //这里我用resource注解，一般用的是@Autowired，他们的区别如有时间我会在后面的博客中来写
 42     private SystemLogService systemLogService;  
 43     
 44     private  static  final Logger logger = LoggerFactory.getLogger(SystemLogAspect. class);   //slf4j 抽象日志变量
 45     
 46     //Controller层切点  
 47     @Pointcut("execution (* com.gcx.controller..*.*(..))")   //切入点表达式这样写在后边的通知方法中就可以使用该方法不用反复的写该切入点表达式了。
 48     public  void controllerAspect() {  
 49     }  
 50     
 51     /** 
 52      * 前置通知 用于拦截Controller层记录用户的操作 
 53      * 
 54      * @param joinPoint 切点 
 55      */ 
 56     @Before("controllerAspect()")
 57     public void doBefore(JoinPoint joinPoint) { //传入连接点对象
 58         System.out.println("==========执行controller前置通知===============");
 59         if(logger.isInfoEnabled()){
 60             logger.info("before " + joinPoint);
 61         }
 62     }    
 63     
 64     //配置controller环绕通知,使用在方法aspect()上注册的切入点
 65       @Around("controllerAspect()")
 66       public void around(JoinPoint joinPoint){
 67           System.out.println("==========开始执行controller环绕通知===============");
 68           long start = System.currentTimeMillis();
 69           try {
 70               ((ProceedingJoinPoint) joinPoint).proceed();
 71               long end = System.currentTimeMillis();
 72               if(logger.isInfoEnabled()){
 73                   logger.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms!");
 74               }
 75               System.out.println("==========结束执行controller环绕通知===============");
 76           } catch (Throwable e) {
 77               long end = System.currentTimeMillis();
 78               if(logger.isInfoEnabled()){
 79                   logger.info("around " + joinPoint + "\tUse time : " + (end - start) + " ms with exception : " + e.getMessage());
 80               }
 81           }
 82       }
 83  
 84     /** 
 85      * 后置通知 用于拦截Controller层记录用户的操作 
 86      * 
 87      * @param joinPoint 切点 
 88      */  
 89     @After("controllerAspect()")  
 90     public  void after(JoinPoint joinPoint) {  
 91   
 92        /* HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();  
 93         HttpSession session = request.getSession();  */
 94         //读取session中的用户  
 95        // User user = (User) session.getAttribute("user");  
 96         //请求的IP  
 97         //String ip = request.getRemoteAddr();
 98         User user = new User();
 99         user.setId(1);
100         user.setName("张三");
101         String ip = "127.0.0.1";
102          try {  
103             
104             String targetName = joinPoint.getTarget().getClass().getName();  
105             String methodName = joinPoint.getSignature().getName();  
106             Object[] arguments = joinPoint.getArgs();  
107             Class targetClass = Class.forName(targetName);  //通过使用反射机制和连接点对象查找目标方法获取自定义注解的信息
108             Method[] methods = targetClass.getMethods();
109             String operationType = "";
110             String operationName = "";
111              for (Method method : methods) {  
112                  if (method.getName().equals(methodName)) {  
113                     Class[] clazzs = method.getParameterTypes();  
114                      if (clazzs.length == arguments.length) {  
115                          operationType = method.getAnnotation(Log.class).operationType();
116                          operationName = method.getAnnotation(Log.class).operationName();
117                          break;  
118                     }  
119                 }  
120             }
121             //*========控制台输出=========*//  
122             System.out.println("=====controller后置通知开始=====");  
123             System.out.println("请求方法:" + (joinPoint.getTarget().getClass().getName() + "." + joinPoint.getSignature().getName() + "()")+"."+operationType);  
124             System.out.println("方法描述:" + operationName);  
125             System.out.println("请求人:" + user.getName());  
126             System.out.println("请求IP:" + ip);  
127             //*========数据库日志=========*//  
128             SystemLog log = new SystemLog();  
129             log.setId(UUID.randomUUID().toString());
130             log.setDescription(operationName);  
131             log.setMethod((joinPoint.getTarget().getClass().getName() + "." + joinPoint.getSignature().getName() + "()")+"."+operationType);  
132             log.setLogType((long)0);  
133             log.setRequestIp(ip);  
134             log.setExceptioncode( null);  
135             log.setExceptionDetail( null);  
136             log.setParams( null);  
137             log.setCreateBy(user.getName());  
138             log.setCreateDate(new Date());  
139             //保存数据库  
140             systemLogService.insert(log);  
141             System.out.println("=====controller后置通知结束=====");  
142         }  catch (Exception e) {  
143             //记录本地异常日志  
144             logger.error("==后置通知异常==");  
145             logger.error("异常信息:{}", e.getMessage());  
146         }  
147     } 
148     
149     //配置后置返回通知,使用在方法aspect()上注册的切入点
150       @AfterReturning("controllerAspect()")
151       public void afterReturn(JoinPoint joinPoint){
152           System.out.println("=====执行controller后置返回通知=====");  
153               if(logger.isInfoEnabled()){
154                   logger.info("afterReturn " + joinPoint);
155               }
156       }
157     
158     /** 
159      * 异常通知 用于拦截记录异常日志 
160      * 
161      * @param joinPoint 
162      * @param e 
163      */  
164      @AfterThrowing(pointcut = "controllerAspect()", throwing="e")  
165      public  void doAfterThrowing(JoinPoint joinPoint, Throwable e) {  
166         /*HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.getRequestAttributes()).getRequest();  
167         HttpSession session = request.getSession();  
168         //读取session中的用户  
169         User user = (User) session.getAttribute(WebConstants.CURRENT_USER);  
170         //获取请求ip  
171         String ip = request.getRemoteAddr(); */ 
172         //获取用户请求方法的参数并序列化为JSON格式字符串  
173         
174         User user = new User();
175         user.setId(1);
176         user.setName("张三");
177         String ip = "127.0.0.1";
178         
179         String params = "";  
180          if (joinPoint.getArgs() !=  null && joinPoint.getArgs().length > 0) {  
181              for ( int i = 0; i < joinPoint.getArgs().length; i++) {  
182                 params += JsonUtil.getJsonStr(joinPoint.getArgs()[i]) + ";";  
183             }  
184         }  
185          try {  
186              
187              String targetName = joinPoint.getTarget().getClass().getName();  
188              String methodName = joinPoint.getSignature().getName();  
189              Object[] arguments = joinPoint.getArgs();  
190              Class targetClass = Class.forName(targetName);  
191              Method[] methods = targetClass.getMethods();
192              String operationType = "";
193              String operationName = "";
194               for (Method method : methods) {  
195                   if (method.getName().equals(methodName)) {  
196                      Class[] clazzs = method.getParameterTypes();  
197                       if (clazzs.length == arguments.length) {  
198                           operationType = method.getAnnotation(Log.class).operationType();
199                           operationName = method.getAnnotation(Log.class).operationName();
200                           break;  
201                      }  
202                  }  
203              }
204              /*========控制台输出=========*/  
205             System.out.println("=====异常通知开始=====");  
206             System.out.println("异常代码:" + e.getClass().getName());  
207             System.out.println("异常信息:" + e.getMessage());  
208             System.out.println("异常方法:" + (joinPoint.getTarget().getClass().getName() + "." + joinPoint.getSignature().getName() + "()")+"."+operationType);  
209             System.out.println("方法描述:" + operationName);  
210             System.out.println("请求人:" + user.getName());  
211             System.out.println("请求IP:" + ip);  
212             System.out.println("请求参数:" + params);  
213                /*==========数据库日志=========*/  
214             SystemLog log = new SystemLog();
215             log.setId(UUID.randomUUID().toString());
216             log.setDescription(operationName);  
217             log.setExceptioncode(e.getClass().getName());  
218             log.setLogType((long)1);  
219             log.setExceptionDetail(e.getMessage());  
220             log.setMethod((joinPoint.getTarget().getClass().getName() + "." + joinPoint.getSignature().getName() + "()"));  
221             log.setParams(params);  
222             log.setCreateBy(user.getName());  
223             log.setCreateDate(new Date());  
224             log.setRequestIp(ip);  
225             //保存数据库  
226             systemLogService.insert(log);  
227             System.out.println("=====异常通知结束=====");  
228         }  catch (Exception ex) {  
229             //记录本地异常日志  
230             logger.error("==异常通知异常==");  
231             logger.error("异常信息:{}", ex.getMessage());  
232         }  
233          /*==========记录本地异常日志==========*/  
234         logger.error("异常方法:{}异常代码:{}异常信息:{}参数:{}", joinPoint.getTarget().getClass().getName() + joinPoint.getSignature().getName(), e.getClass().getName(), e.getMessage(), params);  
235   
236     }  
237     
238 }
```

## 基于XML方式的切面定义
1. 编写切面类（普通类，在spring文件中注册为bean）
2. 使用
```java
<aop:config>
    <aop:aspect ref="sysLogAspect">
        <aop:pointcut expression="@annotation(com.wyj.annotation.SysLog)" id="sysLogPointcut"/>
        <aop:around method="aroud" pointcut-ref="sysLogPointcut"/>
    </aop:aspect>
</aop:config>
```
使用如上的配置方式实现切面的绑定

AOP原理就是 动态代理的原理

CGlib在运行时动态生成字节码，继承被代理的类，重写方法，植入通知，动态生成字节码并运行，使用方法拦截技术。

```java
//代理类 输出日志
import java.lang.reflect.Method;

import net.sf.cglib.proxy.Enhancer;
import net.sf.cglib.proxy.MethodInterceptor;
import net.sf.cglib.proxy.MethodProxy;

public class ShipProxy implements MethodInterceptor { //实现方法拦截器接口
//通过Enhancer 创建代理对象
    private Enhancer enhancer = new Enhancer(); //创建增强器对象Enhancer
    //通过Class对象获取代理对象
    public Object getProxy(Class c){  //返回代理对象 这个工作spring帮忙做了
        //设置创建子类的类
        enhancer.setSuperclass(c);  //设置生成类的父类对象就是继承被代理类
        enhancer.setCallback(this);
        return enhancer.create(); //返回代理对象
    }

    @Override
    public Object intercept(Object obj, Method m, Object[] args, MethodProxy proxy) throws Throwable { //方法拦截 在此处进行拦截 在spring中相当于环绕通知
        // TODO Auto-generated method stub
        System.out.println("日志开始...");
        //代理类调用父类的方法
        proxy.invokeSuper(obj, args);  //调用被代理的方法（通过反射调用父类的方法）
        System.out.println("日志结束...");
        return null;
    }
}
```
总结：CgLib就是使用方法拦截技术，通过enhancer增强器生成代理类的子类，然后通过代理独享调用被代理方法是，通过重写的intercept方法进行被代理类的增强，然后通过反射机制invokeSuper调用父类中该方法的逻辑。