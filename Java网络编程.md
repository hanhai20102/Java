# Java 网络编程

## IP与端口

+ 在网络中每台计算机都必须有一个的IP地址； 
32位，4个字节，常用点分十进制的格式表示，例如：192.168.1.100 
127.0.0.1 是固定ip地址，代表当前计算机，相当于面向对象里的 "this"
+ 两台计算机进行连接，总有一台服务器，一台客户端。

服务器和客户端之间的通信通过端口进行。如图：
![](http://stepimagewm.how2j.cn/881.png)
图解：ip地址是 192.168.1.100的服务器通过端口 8080
与ip地址是192.168.1.189的客户端 的1087端口通信

## 获取本地IP地址
```java
InetAddress host = InetAddress.getLocalHost();
String myIPAddress = host.getHostAddress();
```

## 使用java实现ping命令

```java
Process p = Runtime.getRuntime().exec("ping "+ip);
//通过输入字符流获取执行后的结果
BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
StringBuffer sb = new StringBuffer();
while(reader.readLine()!=null){
	sb.append(reader.readLine());
}

```

## 查询当前主机IP网段有多少可用的IP
```java
package JavaTest;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class SocketTest {

	/**
	 * @param args
	 * @throws UnknownHostException 
	 * @throws InterruptedException 
	 */
	public static void main(String[] args) throws UnknownHostException, InterruptedException {
		// TODO Auto-generated method stub
		InetAddress host = InetAddress.getLocalHost();
		String myIPAddress = host.getHostAddress();
		System.out.println(myIPAddress);
		//查找此IP的网段
		String netWork = myIPAddress.substring(0, myIPAddress.lastIndexOf("."));
		System.out.println("本网段为："+netWork);
		final List<String> ipList = Collections.synchronizedList(new ArrayList<String>());  //ArrayList是线程不安全的，转换为线程安全的类
		//创建线程池控制IP的检测任务
		ThreadPoolExecutor threadPool = new ThreadPoolExecutor(10, 20, 60, TimeUnit.MINUTES, new LinkedBlockingQueue<Runnable>());
		final AtomicInteger num = new AtomicInteger(0);
		for (int i = 0; i < 255; i++) {
			final String ip = netWork+"."+(i+1);
			threadPool.execute(new Runnable() {
				
				@Override
				public void run() {
					// TODO Auto-generated method stub
					try {
						if(judgeIPValid(ip)){
							ipList.add(ip);
						    System.out.println(ip);
						}
					} catch (IOException e) {
						e.printStackTrace();
					}
					synchronized (num) {
						System.out.println("已经完成："+num.incrementAndGet()+"个IP测试");
					}
					
				}
			});
		}
		//线程结束后关闭线程池 并等待一个小时等线程池中的线程把所有的任务都结束了
		threadPool.shutdown();
		if(threadPool.awaitTermination(1, TimeUnit.HOURS)){
			//循环输出所有的IP地址
			System.out.println("共有以下IP地址可以连接");
			for (String sb : ipList) {
				System.out.println(sb);
			}
		}
	}
	
	//使用java执行ping命令
	public static boolean judgeIPValid (String ip) throws IOException {
		//如果输出信息中含有TTL则连接成功
		Process p = Runtime.getRuntime().exec("ping "+ip);
		//通过输入字符流获取执行后的结果
		BufferedReader reader = new BufferedReader(new InputStreamReader(p.getInputStream()));
		StringBuffer sb = new StringBuffer();
		while(reader.readLine()!=null){
			sb.append(reader.readLine());
		}
		if(sb.toString().contains("TTL")){
			return true;
		}
		return false;
	}

}


```