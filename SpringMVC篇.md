# SpringMVC篇

[TOC]

## SpringMVC的优点

- SpringMVC IOC、AOP的功能
- SpringMVC强大的约定大于配置的契约式变成
- 灵活的url到页面控制器的映射
- 灵活的进行数据验证、格式化和数据绑定
- 支持restful风格的URI

## SpringMVC与Struts的区别
- SpringMVC基于方法开发，Struts基于类开发

**struts**框架开发时，所有的方法使用的参数都是action类中的成员变量，请求的方法变得多时会使action类变得很乱很乱

**springMVC**将url与controller中的某个方法进行绑定，请求的参数作为该方法的形参，用户请求该路径时，springmvc生成一个handler对象，该对象包含一个方法，执行完后成形参数据自动销毁。

- springmvc可以使用单例开发，struts无法使用
- struts的处理速度比springmvc慢

## SpringMVC请求处理流程
![](https://img-blog.csdn.net/20170301213745588?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdWhnYWdudQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

SpringMVC的组件：前端控制器（DispatcherServlet）、处理器映射器（HandlerMapping）、处理器适配器（HandlerAdapter）、处理器（Handler）、视图解析器（ViewResolver）、视图（View）

DispatcherServlet-请求HandlerrMappering寻找处理该请求的Handler（带拦截器的Handler链）-前端控制器根据返回结果与HandlerAdapter进行匹配，找到处理此Handler的HandlerAdapter-HandlerAdapter调用自己的Handler方法，利用Java反射机制执行具体的COntroller方法-返回ModelAndView视图对象


## SpringMVC的配置
Servlet开发模式中，请求被映射到web.xml中，然后匹配对应的Servlet进行请求的处理

因此在SpringMVC中需要将符合条件的请求拦截到SpringMVC的前端控制器上，在web.xml文件配置SpringMVC的前端控制器

```java
<servlet>
    <servlet-name>mvc-dispatcher</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!--tomcat启动的时候  就加载这个数字-->
    <load-on-startup>1</load-on-startup>  //容器是否在web应用启动时就加在这个Servlet，实例化并调用init()方法，如果值为0或者负数，则该servlet在被请求时再加载
</servlet>

<servlet-mapping>
    <servlet-name>mvc-dispatcher</servlet-name>
    <url-pattern>*.action</url-pattern>
</servlet-mapping>
```
如果不配置<init-param></init-param>中**configuationLocation**属性，springMVC的配置文件就必须放在web-xml同级目录下，并取名为前端控制器的名称-servlet.xml,上例 mvc-dispatcher-servlet.xml
如果不想放在同级目录就在其configuationLocation中指明具体的位置

- 关于web.xml中，前端控制器配置了拦截请求的格式，当所有请求都被拦截的时候，关于服务器上的静态资源等也被拦截但是找不到匹配的Handler，因此使用<mvc:default-servlet-handler/>将不能处理的请求交给tomcat服务器，前提是静态资源不能放在web-inf目录下，放在web-app目录下
- <mvc:annnotation-driven> 包含了默认的处理器映射器和适配器

## 处理器映射器 HandlerMapping与处理器适配器 HandlerAdapter

### xml资源配置方式（处理器映射器）
- BeanNameUrlHandlerMapping  以Bean的name作为url进行查找请求处理的类
```java
<bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMaping">
<bean name="/queryFruits.action" class="com.muyuxi.controller.FruitsTest">
```
- SimpleUrlHanderMapping 内部参数配置请求的url和handler的映射关系，并加入拦截器
```java
<bean id="urlMapping" class="org.springframework.web.servlet.handler.SimpleUrlHandlerMapping">
<property name="interceptors">  //定义拦截器
<list>
<ref bean="someCheckInterceptor1"/>
<ref bean="someCheckInterceptor2">
</list>
</property>

<property name="mappings">
<props>
<prop key="/queryFruit">fruitsController</prop>
</props>
</property>
</bean>
```
- ControllerClassNameHandlerMapping 采用惯例优先的原则处理请求
请求的名称要与Controller（Handler）名称一致，比如xxxController会映射到"/xxx"的URL。

### xml资源配置方式 （处理器适配器）
- SimpleControllerHandlerAdapter 所有的Handler必须实现Controller接口
```java
<bean class="org.springframework.web.servlet.mvc.SimpleControllerHandlerAdapter"/>
```
- HttprequestHandlerAdapter 所有的Handler必须实现HttpRequestHandler接口 然后在controller类中实现handlerRequest方法返回ModelAndView视图

总之，处理器映射器就是根据url查找Handler，处理器是配置就是根基要求的规则 （handler instanceof xxx接口）执行这个handler，使用java反射机制执行该handler中的方法。

### 注解
以上方式，一个handler中只能编写一个处理请求的方法，对于大量请求处理逻辑勉为其难

使用<mvc:annotation-driven/>标签配置，自动提供处理器适配器和处理器映射器 ，还提供了数据绑定，数据校验，读写XML、Json类型转换器等功能

在Handler中只需要@Controler表名该类是一个控制器类就可以被注解的处理器适配器找到，为了让处理器映射器和适配器找到对应的Handler，必须把类扫描为bean加入springmvc的容器
- <bean ></bean>的方式
- 使用
> <context:component-scan base-package="com.muyuxi.Controller"></context:component-scan>

### 视图解析器

InternalResourceViewResolver Handler返回的视图解析为InternalResourceView对象，把模型保存在request对象中，经过前端控制器的视图渲染，转发到目标url

利用视图解析器可以将jsp文件放在web-inf下通过加上视图解析器的前后缀访问，但是静态资源不能放在web-inf下，jsp放在web-inf下浏览器不能通过url直接访问

```java
<!--配置视图解析器 方便页面返回-->
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/"></property>
    <property name="suffix" value=".jsp"></property>
</bean>
```
### 请求映射
@requestMapping
- value url的值
- method=RequestMethod.Get 请求的方法为Get方法
- param="userName" 请求的数据包含username才会被处理 
- headers="Content-Type:text/html;charset=UTF-8" 请求头的content-Type是这些内容时才会处理
- consumes="application/json" request的请求头包含application/json 才处理
- produces="application/json",request的请求头包含application/json 才处理，并且response的返回内容类型为application/json

### 参数绑定

#### 简单类型的参数绑定
形参与传入的参数一样 直接写 Integer id 即可

不一样的时候可以使用@RequestParam(value="",required=false) Integer ids;也可以指定默认值 defaultvalue=""

#### 包装类型的参数绑定
直接写形参 User user 请求的参数要与对象的成员变量一致
form表单中属性的名称需要与javaBean中的属性一一对应，在形参中可以放置request，response，session对象等等很是自由

#### 集合类型的参数绑定

**数组**

FruitsArray（Model model，int[] ids）数组的名称需要和jsp页面复选框相同的name值，这样才能接收此参数

**List类型**
必须使用一个包装类作为形参接受前台传入的List集合数据，在包装类中定义成员List<User> users;前台传入的List的值必须与包装类中成员的List的name相同

前台传List类型的数据方法：form表单中采用List名[index].属性名

**Map类型**
必须使用一个包装类封装成为类的成员变量Map<String,Object> fruitMap

前台传Map类型数据的方法：
form表单中使用Map名称["key"]的name值

### 校验器

#### Spring Validator接口校验

SpringMVC提供一个Validator接口，可以使用它来验证自己定义的JavaBean 
1. 编写一个Validator接口的实现类，并实现Validator接口的supports方法和validate方法，support方法判断当前Validator实现类是否支持校验当前需要校验的实体类，支持返回true否则为false
2. validate方法，具体的校验实体类中的属性**详细见SpringMVC+mybatis书196页**
```java
Errors errors;
errors.rejectValue(错误字段,错误码(英文),错误信息);
```
3. handler类中
```java
public Msg saveEmp(@Valid Employee employee, BindingResult result){
    if(result.hasErrors()){
        //校验失败在前台显示后台检验错误的信息
        List<ObjectError> errors = result.getAllErrors();
        Map<String,Object> errorMap = new HashMap<>();
        for(ObjectError objectError :errors){
            System.out.println("错误的字段名"+fieldError.getField());
            System.out.println("错误信息"+objectError.getDefaultMessage();
             System.out.println("错误码"+objectError.getCode();
            errorMap.put(fieldError.getField(),fieldError.getDefaultMessage());
        }
        return Msg.fail().add("errorField",errorMap);

    }else{
        employeeService.saveEmp(employee);
        return Msg.success();
    }
}
```
#### Bean Validation数据校验（JSR-303校验）
单独集成了Hibernate的检验框架，用于服务器端的数据校验

1. 引入hiberbnate-validator-4.3.0.jar Jboss-logging.jar validation-api-1.0.0.GA.jar
2. <mvc:annotation-driven validator="validator">
3. 在配置文件中添加validator的校验器配置
```java
<bean id="validator" class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean">
<property name="providerClass" value="org.hibernate.validator.HibernateValidator"/>
<property name="validationMessageSource" ref="messageSource">
</bean>
```
定义一个validator的校验器，指定提供的是Hibernatye的检验框架
validationMessageSource是校验使用的资源文件，如果不手动指定默认使用类路径下的ValidationMessage.properties
```java
<bean id="messageSource" class="">//自定义的检验资源文件
 <property naem="basenames">
 <list>
 <value>classPath:ProductValidationMessage</value>
 </list>
 </property>
 <property naem="fileEncoding" value="utf-8">
 <property naem="cacheSeconds" value="120">
</bean>
```
加载错误信息定义的类为ReLoadableResourceBundleMessgaeSource

basenames为指定的资源文件名 

fileEncoding 指定的编码格式

cacheSeconds资源文件内容的缓存时间单位为s

4. 编写错误信息资源文件ProductValidationMessage.properties
fruit.name.length.error = 请输入1到20个字符的商品名称
5. 在controller中使用@Validated注解以及BindingResult result步骤spring validator接口校验一样
6. 在需要校验的JavaBean属性上方使用注解校验 Message写错误码然后getDefaultMessage会根据错误码返回错误信息
也可以用正则表达式校验

### 异常处理
SpringMVC有一个全局的异常处理器，DAO层，Service，Action层都以throws Exception向上抛出，由前端控制器交给全局异常处理器处理
功能：
1. 解析异常类型，判断属于哪一种异常类
2. 如果是自定义异常，去除异常信息，在错误页面展示，如果不是自定义异常，构造一个自定义异常提示未知错误
3. 将异常信息绑定到界面的Model域中跳转到相应的异常信息界面

步骤：
1. 实现HandlerExceptionResolver接口
2. 实现resolverException方法
3. 返回ModelAndView对象
4. 异常信息不会硬编码到代码中需要一个资源配置文件exceptionMapping.properties
5. 实现读取配置文件的类
6. 在controller中抛出自定义异常的时候通过错误码获取错误信息抛出至全局异常处理器

读取配置文件
```java
Properties prop;
InputStream fis = Resource.getResourceAsStream("");
prop.load(fis);
fis.close;
String msg = prop.getProperty("key");
```
### 拦截器与拦截器链详细见SpringMVC+Mybatis书207页
拦截器链：
> preHandler（1）-perHandler（2）-controller-（返回ModelAndView前）postHandler（2）-postHandler（1）-（返回视图前）afterCompletion（2）-afterCompletion（1）

## Spring RestFul风格的URI
- 增 post
- 删 delete
- 改 put
- 查 get

1. 在web.xml文件中修改请求的拦截为全部拦截"/"
2. 可以在controller的@requestMapping的url路径中使用参数/{id}
3. 使用@Pathvariable("id") Integer id
4. 可以发送put、delete请求 在requestMapping中的method改为RequestMethod.PUT
5. 有的服务器不支持GET、POST以外的请求，需要配置过滤器（发put和delete的过滤器）

第一种解决方法：

发post请求，同时在请求参数中加"&_method=put" 过滤器会将post请求转换为method指定的请求方式。
```java
<!--使用rest风格的URI 将页面普通的post请求转换为指定的delete或者put请求-->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

第二种解决方法：

可以直接发put和delete请求，不能发多媒体数据
```java
<!--spring MVC 发put等等请求的过滤器-->
<filter>
    <filter-name>HttpPutFormContentFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>
    
<filter-mapping>
    <filter-name>HttpPutFormContentFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

```
## Spring Json数据
- @RequestBody --Json字符串转换为Java对象  当页面传输的数据是Json数据是，@RequestBody转换为实体类的原理就是请求数据的content-Type为需要转换的数据格式时，调用处理器适配器的类型转换器进行映射和转换
- @ResponseBody  --对象转换为Json字符串
需要Jackson的jar包

## SpringMVC上传文件
Page：220

1. 配置类型转换器
2. 前台上传
3. controller复制文件 file.transferTo(new File)
4. 浏览器使用虚拟路径访问服务器上存储在磁盘上的文件
##  关于器

前端控制器，过滤器，拦截器，视图解析器器，监听器

## 监听器

监听器对session的监听分为对session生命周期的监听和对session对象的属性监听。对session生命周期的监听实现HttpSessionListener接口，重写sessionDestroyed和sessionCreated方法即可而且可以获取到session的ID。通过HttpSessionEvent
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

监听器对request对象的监听也是分为相似的两个部分。
而其实现的接口也大同小异ServletRequestListener, ServletRequestAttributeListener。用法一致。

## 字符编码过滤器
就是对传输的格式进行统一的转码
```java
 <!--spring mvc的字符编码过滤器 放在所有过滤器之前-->
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>

    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>

    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
    
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```