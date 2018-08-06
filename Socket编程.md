# Socket编程

## Socket简介
>参考文章：[http://how2j.cn/k/socket/socket-socket/400.html#nowhere](http://how2j.cn/k/socket/socket-socket/400.html#nowhere)
![Socket](http://stepimagewm.how2j.cn/882.png)
利用Socket建立客户端和服务器端的连接

1. 服务器端使用ServerSocket开启监听端口，等待客户端的建立连接的请求
2. 客户端知道服务器端的IP以及端口，通过Socket创建Socket对象，建立与服务器端的连接
3. 服务器端收到建立连接的请求，在服务器端创建一个Socket对象ServerSocket ss = new ServerSocket(8888);Socket s =  ss.accept();通过Socket对象与客户端的Socket对象实现客户端与服务器端的通信。

## 利用Socket实现收发字符串
**Client端：** 负责向服务器端发送消息
```java
package socket;
 
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.net.UnknownHostException;
import java.util.Scanner;
 
public class Client {
 
    public static void main(String[] args) {
 
        try {
            Socket s = new Socket("127.0.0.1", 8888);
 
            OutputStream os = s.getOutputStream();
 
            //把输出流封装在DataOutputStream中
            DataOutputStream dos = new DataOutputStream(os);
            //使用writeUTF发送字符串
            dos.writeUTF("Legendary!");
            dos.close();
            s.close();
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```
**Server端：** 服务器端负责读取客户端传递的消息
```java
package socket;
 
import java.io.DataInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.ServerSocket;
import java.net.Socket;
 
public class Server {
 
    public static void main(String[] args) {
        try {

            ServerSocket ss = new ServerSocket(8888);
            System.out.println("监听在端口号:8888");
            Socket s = ss.accept();
 
            InputStream is = s.getInputStream();
            //把输入流封装在DataInputStream
            DataInputStream dis = new DataInputStream(is);
            //使用readUTF读取字符串
            String msg = dis.readUTF();
            System.out.println(msg);
            dis.close();
            s.close();
            ss.close();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
 
    }
}
```

## 利用Socket实现客户端和服务器端的交流(数据库版)
1. 新建数据存存储id、receive、response字段，供服务器端回复客户端消息使用
2. 创建实体类存放数据库表中的信息以及创建DAO类，写查询方法根据receive查询response字段，如果查询不到，在自定义的列表中随机返回一条错误信息。
3. Server端写死循环，在接受到客户端消息之后给出相应的回复信息;客户端在发出信息之后接受服务器端的信息也是一个死循环

Server端：
```java
package socket;
 
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;
 
public class Server {
 
    private static List<String> cannotUnderstand= new ArrayList<>();
    static{
        cannotUnderstand.add("听求不懂啊");
        cannotUnderstand.add("说人话");
        cannotUnderstand.add("再说一遍？");
        cannotUnderstand.add("大声点");
        cannotUnderstand.add("老子在忙，一边玩儿去");
    }
    public static void main(String[] args) {
        try {
 
            ServerSocket ss = new ServerSocket(8888);
 
            System.out.println("监听在端口号:8888");
            Socket s = ss.accept();
 
            InputStream is = s.getInputStream();
            DataInputStream dis = new DataInputStream(is);
            OutputStream os = s.getOutputStream();
            DataOutputStream dos = new DataOutputStream(os);
 
            while (true) {
                String msg = dis.readUTF();
                System.out.println(msg);
                 
                List<Dictionary> ds= new DictionaryDAO().query(msg);
                String response = null;
                if(ds.isEmpty()){
                    Collections.shuffle(cannotUnderstand);
                    response = cannotUnderstand.get(0);
                }
                else{
                    Collections.shuffle(ds);
                    response = ds.get(0).response;
                }
                dos.writeUTF(response);
            }
 
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
 
    }
}
```
Client端：
```java
package socket;
  
import java.io.DataInputStream;
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStream;
import java.net.Socket;
import java.net.UnknownHostException;
import java.util.Scanner;
  
public class Client {
  
    public static void main(String[] args) {
  
        try {
            Socket s = new Socket("127.0.0.1", 8888);
  
            OutputStream os = s.getOutputStream();
            DataOutputStream dos = new DataOutputStream(os);
            InputStream is = s.getInputStream();
            DataInputStream dis = new DataInputStream(is);
              
            while(true){
                Scanner sc = new Scanner(System.in);
                String str = sc.nextLine();
                dos.writeUTF(str);
                String msg = dis.readUTF();
                System.out.println(msg);
                System.out.println();
 
            }
              
        } catch (UnknownHostException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        } catch (IOException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }
}
```
## 利用Socket实现客户端和服务器端的交流(多线程)
在主线程中实现交流只能等待收到信息之后才能继续交流
因此接受信息和发送信息可以放在两个线程中实现主线程负责开启这两个线程，依旧不关闭客户端和服务器端的连接用死循环，这样就不用等待信息接收之后才可以再发信息给对面。

**接受线程：**
```java
import java.io.DataInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.net.Socket;

public class ReceiveThread extends Thread {
    private Socket s;
    public ReceiveThread(Socket s) {
        this.s = s;
    }
    @Override
    public void run() {
        try {
            InputStream in = s.getInputStream();
            DataInputStream dis = new DataInputStream(in);
            while(true){
                String msg = dis.readUTF();
                System.out.println("沐雨昔 "+msg);
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

}

```
**发送线程：**
```java
import java.io.DataOutputStream;
import java.io.IOException;
import java.io.OutputStream;
import java.net.Socket;
import java.util.Scanner;

public class SendThread extends Thread {
	private Socket s;
	public SendThread(Socket s) {
		this.s = s;
	}
	@Override
	public void run() {
		
		try {
			OutputStream out = s.getOutputStream();
			DataOutputStream dos = new DataOutputStream(out);
			while(true){
				Scanner sc = new Scanner(System.in);
                String response = sc.nextLine();
                dos.writeUTF(response);
			}
			
		} catch (IOException e) {
			e.printStackTrace();
		}
	
	}

}
```
## Socket中使用的输入输出流
InputStream和DataInputStream
OutputStream和DataOutputStream
```java
InputStream in = s.getInputStream();  //s为Socket对象
DataInputStream dis = new DataInputStream(in);
String msg = dis.readUTF();
System.out.println("沐雨晴 "+msg);
```
```java
OutputStream out = s.getOutputStream(); //s为Socket对象
DataOutputStream dos = new DataOutputStream(out);
Scanner sc = new Scanner(System.in);
String response = sc.nextLine();
dos.writeUTF(response);
```
PS：因为字节流所以在使用InputStream的write方法传递整型的范围是-256-255.

![heihei](https://ss2.bdstatic.com/70cFvnSh_Q1YnxGkpoWK1HF6hhy/it/u=2193258640,853506226&fm=27&gp=0.jpg)