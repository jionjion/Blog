---
title: Java基础-网络通信
abbrlink: 86ef4956
date: 2020-01-30 16:13:20
categories:
  - Java
  - Socket
tags: [Java, Socket]
---

> Java 实现网络通信及其示例代码

<!--more-->



# JAVA中的网络通信

## 简介
Java中对互联通信的实现,使用TCP/UDP分别完成数据传输.

## 相关知识
### 网路通信
互联网中,使用IP4/IP6,TCP等协议结合IP地址+端口号确定互联网中唯一的一台电脑进行通信.

### TCP/IP模型
- **应用层** http(超文本传输协议),FTP(文件传输协议),SMTP(简单邮件传送协议),Telnet(远程登录服务)
- **传输层** TCP/IP协议
- **网络层**
- **数据链路层**
- **物理层** 各种硬件设备

### Socket
IP和端口号组成,是网络通信的链路的终结点.

### JAVA中的网络支持
- **InetAddress**:标识网络上的硬件资源.IP地址
- **URL**:统一资源定位符,通过其可以直接读取或者写入网络上的数据
- **Sockets**:使用TCP协议实现网络通信的Socket相关的类
- **Datagram**:使用UDP协议,将数据保存在数据报中,通过网络进行通信

## 示例代码

[参考代码](https://github.com/jionjion/JAVA_WorkSpace/tree/master/JavaBase/src/networkCommunication)

* `TCPNetwork`  TCP通信模型
* `UDPNetwork`  UDP通信模型
* `InetAddressDemo` IP地址和端口号
* `URLDemo` URL实例



### `URL`

传入`http`协议的URL连接,创建`URL`对象,完成资源的访问.

``` java
public static void main(String[] args) throws Exception{
		
		//创建一个URL实例
		URL baidu = new URL("http://www.baidu.com");
		//创建子URL
		URL url = new URL(baidu,"/s?wd=hello#index");
		
		System.out.println("协议类型:"+url.getProtocol());
		System.out.println("主机IP:"+url.getHost());
		System.out.println("端口号:"+url.getPort());
		System.out.println("文件的路径:"+url.getPath());
		System.out.println("文件名:"+url.getFile());
		System.out.println("相对路径:"+url.getRef());
		System.out.println("查询字符串:"+url.getQuery());
		
		//查询网页
		InputStream inputStream = url.openStream();
		InputStreamReader inputStreamReader = new InputStreamReader(inputStream,"UTF-8");
		BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
		//读取数据
		String data = bufferedReader.readLine();
		while(data != null){
			System.out.println(data);
			data = bufferedReader.readLine();
		}
		bufferedReader.close();
		inputStreamReader.close();
		inputStream.close();
	}
```

### `InetAddress`
通过IP地址,或者主机名,或者别称获得IP对象

``` java
public static void main(String[] args) throws Exception{
	//获取实例
	InetAddress address = InetAddress.getLocalHost();
	System.out.println("直接获得当前的IP和端口号:"+address.toString());
	System.out.println("当前计算机名称:"+address.getHostName());
	System.out.println("当前计算机IP:"+address.getHostAddress());
	System.out.println("字节数组形式的IP:"+Arrays.toString(address.getAddress()));

	//根据机器名获得InetAddress实例
	InetAddress address2 = InetAddress.getByName("DESKTOP-QLSNNQ9");
	System.out.println("计算机名:"+address2.toString());
	System.out.println("IP地址:"+address2.getHostAddress());

	//根据IP获得InetAddress实例
	InetAddress address3 = InetAddress.getByName("192.168.0.1");
	System.out.println("计算机名:"+address3.toString());
	System.out.println("IP地址:"+address3.getHostAddress());
}
```



### TCP通信协议

**TCP**
面向连接,可靠的,有序的,以字节流的方式发送数据
**基于TCP协议实现网络通信的类**
客户端的Socket类
服务端的ServerSocket类
**过程:**
服务器创建socket进行侦听,客户端创建socket进行连接,服务器收到请求后建立连接;随后以IO流进行通信;
在通信结束后,双方关闭相关的socket及相应的资源
**步骤:**
1.创建ServerSocket和Socket
2.打开连接到Socket的输入/输出流
3.按照协议对Socket进行读/写操作
4.关闭输入输出流,关闭Socket
**多线程服务器:**
1.服务端创建ServerSocket,循环调用accept(),等待客户端连接
2.客户端创建一个socket,并请求和服务器端连接
3.服务器端接收客户请求,创建socket与该客户建立专线连接
4.建立连接的两个socket在一个单独的线程上对话
5.服务器端继续等待新的连接



### TCP通信实现

#### 线程类实现服务端监听

 1. 使用构造器传入一个`Socket`通信对象
 2. 重写多线程`run()`方法
 3. 从`Socket`对象中获得输入流,获得传入信息
 4. 写入输出流,传入`Socket`中
 5. 关闭资源

``` java
/**	服务器端,进行处理类
 1. 	传入一个Socket,交由多线程任务同步进行,类似于回调函数,可以处理多并发请求*/
public class ServerThread extends Thread {

	//和本线程相关的socket
	Socket socket = new Socket();
	
	public ServerThread(Socket socket) {
		this.socket = socket;
	}
	
	/**重写线程方法,调用一个线程,相应客户端的请求*/
	@Override
	public void run() {
		
		BufferedReader bufferedReader = null;
		PrintWriter printWriter = null;
		
		try {
			//3.获取输入流,读取客户端信息
			bufferedReader = new BufferedReader(	//开启输入流
										new InputStreamReader(
											socket.getInputStream(), "UTF-8"));
			String info = null;		//读取输入流
			while((info=bufferedReader.readLine()) != null){
				System.out.println("侦听到来自客户端的:"+info);
			}
			socket.shutdownInput();	//关闭输入流
			//4.获取输出流,相应客户端的请求
			printWriter = new PrintWriter(socket.getOutputStream());
			printWriter.write("欢迎你,客户端.....");
			printWriter.flush();
			socket.shutdownOutput();
		} catch (Exception e) {
			System.err.println("客户端读取相应出现异常...");
			e.printStackTrace();
		} finally {
			try {
				//5.关闭资源
				printWriter.close();
				bufferedReader.close();
				socket.close();
			} catch (IOException e) {
				System.err.println("客户端关闭出现异常...");	
				e.printStackTrace();
			}
		}
	}
}
```
#### 开启服务端监听

 1. 创建服务器端的ServerSocket,并侦听端口
 2. 在一个死循环中开始侦听,等待连接,并创建一个子线程进行调用

``` java
public static void main(String[] args) throws Exception {
	int count = 0;	
	//1.创建服务器端的ServerSocket,并侦听端口
	ServerSocket serverSocket = new ServerSocket(8888);
	//2.开始侦听,等待连接
	System.out.println("-----循环侦听,等待客户度连接....------");
	while(true){
		Socket socket = serverSocket.accept();
		//创建一个新的线程,启动
		ServerThread thread = new ServerThread(socket);
		//启动线程  
		thread.start();
		count++;					//统计连接过的服务器
		System.out.println("当前您为第"+count+"个用户");
	}	
}
```

#### 模拟客户端请求

1. 创建`Socket`.对象,指定IP和端口
2. 创建传输内容,写入输出流
3. 获得服务器响应,放入输入流
4. 关闭资源

``` java
public static void main(String[] args) throws Exception{
	//1.创建Socket,指定服务器地址和端口
	Socket socket = new Socket("127.0.0.1", 8888);
	//2.获取输出流,发送信息
	PrintWriter printWriter = new PrintWriter(socket.getOutputStream());
	printWriter.write("你好,我是客户端!");
	printWriter.flush();
	socket.shutdownOutput();
	//3.读取服务器端的相应
	BufferedReader reader = new BufferedReader(
										new InputStreamReader(
											socket.getInputStream(),"UTF-8"));
	String info = null;		//读取输入流
	while((info=reader.readLine()) != null){
		System.out.println("侦听到来自服务器的:"+info);
	}
	//4.关闭资源
	printWriter.close();
	socket.close();
}
```



### UDP通信协议

**UDP**
进行数据传输时,首先将要传输的数据定义成为数据报(Datagram),在数据包中指明要达到的Socket(主机地址和端口),然后再将数据报发送出去.
**基于UDP协议实现网络通信的类**
DatagramPacket:表示数据报包
DatagramSocket:表示端到端的通信类
**发送步骤**
1.创建DatagramSocket,指定端口号	
2.创建DatagramPacket
3.接收客户端发送的数据信息	
4.读取数据	
**接收步骤**
1.定义发送信息
2.创建DatagramPacket,包含将要发送的信息
3.创建DatagramSocket
4.发送数据



### UDP通信实现

#### 开启服务器监听
1. 指定地址和端口
2. 创建数据报和内容
3. 创建数据报通信对象
4. 向其发送数据报文
5. 接收服务器返回信息,并获取数据
6. 接收数据
7. 读取数据
8. 关闭资源
``` java
public static void main(String[] args) throws Exception {
	//1.指定发送的地址,端口号,数据
	InetAddress address = InetAddress.getByName("127.0.0.1");
	int port = 9999;
	byte[] request = new String("(｡･∀･)ﾉﾞ嗨").getBytes();
	//2.创建数据报,包含发送的数据信息
	DatagramPacket packet = new DatagramPacket(request, request.length, address, port);
	//3.创建DatagramSocket对象
	DatagramSocket socket = new DatagramSocket();
	//4.向服务器端发送数据报
	socket.send(packet);

	//5.接收服务器端信息,获取数据报
	byte[] respose = new byte[1024];
	DatagramPacket packet2 = new DatagramPacket(respose, respose.length);
	//6.接收服务器的相应数据
	socket.receive(packet2);
	//7.读取数据
	String info = new String(respose, 0, packet2.getLength());
	System.out.println("服务器响应为:"+info);
	//8.关闭资源
	socket.close();
}
```

#### 模拟客户端请求

 1. 创建数据报通信对象
 2. 获得输入信息
 3. 接收客户端信息
 4. 读取数据
 5. 从报文中获取来源信息,并返回响应
 6. 创建数据报
 7. 发送,响应客户端
 8. 关闭资源

``` java
public static void main(String[] args) throws Exception {
	//1.创建服务端的DatagramSocket,指定端口
	DatagramSocket socket = new DatagramSocket(9999);
	//2.创建数据报,接收客户端的数据报信息
	byte[] request = new byte[1024];	//创建字节数组,指定数据报的大小
	DatagramPacket packet = new DatagramPacket(request, request.length);
	//3.接收客户端发送的数据
	System.out.println("---------服务器正在监听-----------");
	socket.receive(packet);			//该方法在接收到数据报之前会一直阻塞
	//4.读取数据
	String info = new String(request, 0, packet.getLength());
	System.out.println("服务器收到数据报:"+info);



	//5.响应请求,从数据报中发送来的客户端地址,端口
	InetAddress address = packet.getAddress();
	int port = packet.getPort();
	byte[] response = new String("欢迎您,嘤嘤...").getBytes();
	//6.创建数据报,包含响应的数据信息
	DatagramPacket packet2 = new DatagramPacket(response, response.length, address, port);
	//7.响应客户端
	socket.send(packet2);
	//8.关闭资源
	socket.close();
}
```