 

# Apache Mina框架的使用

## **1.简介**

Apache Mina是一个网络通信应用框架。Mina提供了事件驱动、异步（Mina的异步IO默认使用的是JAVA NIO作为底层支持）

操作的编程模型。

## **2.关键对象**

### **a.接收器和连接器**

Acceptor(接收器)线程专门负责接受连接，在其上有一个selector，轮询是否有连接建立上来，当有连接建立上来，调用ServerSocketChannel.accept方法来接受连接，这个方法返回一个session对象，然后将这个session对象加入processor中，由processor遍历每个session来完成真正的IO操作。processor上也有一个 selector与一个Processor线程,selector用于轮询session，Processor线程处理每个session的IO操作。

> Acceptor有4种：
>  \* NioSocketAcceptor : the non-blocking Socket transport IoAcceptor TCP/IP接收器
>  \* NioDatagramAcceptor : the non-blocking UDP transport IoAcceptor UDP接收器
>  \* AprSocketAcceptor : the blocking Socket transport IoAcceptor, based on APR 基于Apache APR的socket接收器
>  \* VmPipeSocketAcceptor : the in-VM IoAcceptor 虚拟通道（VM）通信的接收器

Connector(连接器)用户客户端申请连接，和服务端要有Acceptor一样，客户端要有对应的Connector来进行连接控制。（在我实际测试中，服务端选用NioSocketAcceptor时，用telnet也可以作为客户端，并不需要额外编写客户端代码）。

>   Connector有以下几种:
>  \* NioSocketConnector : the non-blocking Socket transport Connector
>  \* NioDatagramConnector : the non-blocking UDP transport Connector
>  \* AprSocketConnector : the blocking Socket transport Connector, based on APR
>  \* ProxyConnector : a Connector providing proxy support
>  \* SerialConnector : a Connector for a serial transport
>  \* VmPipeConnector : the in-VM Connector

### **b.FilterChain**

I/O Filter Chain层是介于I/O Service层与I/O Handler层之间的一层，从它的命名上可以看出，这个层可以根据实际应用的需要，设置一组IoFilter来对I/O Service层与I/O Handler层之间传输数据进行过滤，任何需要在这两层之间进行处理的逻辑都可以放到IoFilter中。要实现一个自定义的IoFilter，一般是直接实现IoFilterAdapter类。同时，Mina也给出了几类常用的开发IoFilter的实现类，如下所示：

> LoggingFilter	记录所有事件和请求
>
> ProtocolCodecFilter	将到来的ByteBuffer转换成消息对象（POJO）
>
> CompressionFilter	压缩数据
>
> SSLFilter	增加SSL – TLS – StartTLS支持

### **c.handler**

处理程序Handler负责处理MINA触发的所有I/O事件，在事件穿越过滤器链之后，IoHandler接口将会接手所有的事件。它提供如下方法：

#### sessionCreated

​	方法sessionCreated会在一个新的连接被创建时调用，对于TCP协议的程序，当连接被接受时触发该；对于UDP协议的程序，当UDP数据包被接收时触发该事件。该方法可以用来初始化session属性，对于一个特定的连接，该方法只会执行一次。该方法在I/O处理器线程上下文中被调用，因此，该方法要使用消耗时间最小的方式来实现，同一个线程可以同时处理多个
session。

#### sessionOpened
   方法sessionOpened会在一个连接被打开时调用，它通常会在方法sessionCreated调用之后被调用。如果线程类型特殊配
置过，该方法会在其他的线程中被调用，而不一定是在I/O处理器线程中被调用。

#### sessionClosed
   方法sessionClosed会在一个连接关闭时调用，这里可以进行session清理动作。

#### sessionIdle
   方法sessionIdle会在一个session空闲时被调用。

#### exceptionCaught
   方法exceptionCaught会在有异常抛出时被调用，如果这个异常是一个IOException异常，连接会被关闭。

#### messageReceived
   方法messageReceived会在有消息被接收时被调用，这里是一个应用要处理的大部分任务的地方。你预计在这里会出现的所有的消息类型，都需要关注到。

#### messageSent
   方法messageSent会在有消息或者说响应被发送出去(调用IoSession.write()方法)后被调用。

## **3.举例**

这里编写了一个Mina服务端和客户端，服务端MinaTimerServer创建了一个NioSocketAcceptor来接收连接请求，接收到的信息由TimeServerHandler来处理。客户端MinaTimerClient创建了一个NioSocketConnector来发送连接请求，接收到的信息由TimeClientHandler来进行处理。

### **MinaTimerServer**

```java
package com.pbs.common.Server;

import org.apache.mina.core.service.IoAcceptor;
import org.apache.mina.core.session.IdleStatus;
import org.apache.mina.filter.codec.ProtocolCodecFilter;
import org.apache.mina.filter.codec.textline.TextLineCodecFactory;
import org.apache.mina.filter.logging.LoggingFilter;
import org.apache.mina.transport.socket.nio.NioSocketAcceptor;

import java.io.IOException;
import java.net.InetSocketAddress;
import java.nio.charset.Charset;

/**
 * Created by lifei7 on 2016/5/24.
 */
public class MinaTimerServer {
   private static final int PORT = 9123;

public static void main(String[] args) throws IOException {
     IoAcceptor acceptor = new NioSocketAcceptor();
     //这个过滤器用来记录所有的信息，比如创建session(会话)，接收消息，发送消息，关闭会话等
     acceptor.getFilterChain().addLast("logger",new LoggingFilter());
     //这个过滤器用来转换二进制或协议的专用数据到消息对象中
     acceptor.getFilterChain().addLast("codec",new ProtocolCodecFilter(
         new TextLineCodecFactory(Charset.forName("UTF-8"))));

//handler用于实时处理客户端的连接请求，这个handler类必须实现IoHandler这个接口。对于所有使用MINA的程序来说
     //主要的负荷都在这个类
     acceptor.setHandler( new TimeServerHandler());

//现在我们要添加一些NioSocketAcceptor 配置，这将允许我们设置特殊的socket设置来接收客户端的连接。
     acceptor.getSessionConfig().setReadBufferSize( 2048 ); //输入缓冲区的大小
     acceptor.getSessionConfig().setIdleTime( IdleStatus.BOTH_IDLE, 10 );//both read&write等待时间
     acceptor.bind( new InetSocketAddress(PORT) );
	}
}
```

### **TimeServerHandler**

```java
package com.pbs.common.Server;
import java.util.Date;
import org.apache.mina.core.service.IoHandler;
import org.apache.mina.core.service.IoHandlerAdapter;
import org.apache.mina.core.session.IdleStatus;
import org.apache.mina.core.session.IoSession;
import javax.script.ScriptEngine;
import javax.script.ScriptEngineManager;
import javax.script.ScriptException;
public class TimeServerHandler implements IoHandler {
	private ScriptEngine jsEngine = null;
	public TimeServerHandler(){
     ScriptEngineManager sfm = new ScriptEngineManager();
     jsEngine = sfm.getEngineByName("JavaScript");
     if (jsEngine == null) {
       throw new RuntimeException("找不到JavaScript引擎");
     }
   }
   /**
   * Trap exceptions.
   */
   @Override
   public void exceptionCaught( IoSession session, Throwable cause ) throws Exception
   {
       cause.printStackTrace();
   }
  /**
   * If the message is 'quit', we exit by closing the session. Otherwise,
   * we return the current date.
   */
   @Override
   public void messageReceived( IoSession session, Object message ) throws Exception
   {
     String str = message.toString();
     System.out.println("server接收信息:"+str+" to "+session.getRemoteAddress());
	if( str.trim().equalsIgnoreCase("quit") ) {
       // "Quit" ? let's get out ...
       session.close(true);
       return;
     }
     try{
       Object result = jsEngine.eval(str);
       session.write(result.toString());
     }catch (ScriptException e){
       session.write("非法字符串"+e.getMessage());
     }
}

   @Override
   public void messageSent(IoSession session, Object message) throws Exception {
     System.out.println("server发送信息:"+message.toString()+" to "+session.getRemoteAddress());
   }

   @Override
   public void inputClosed(IoSession session) throws Exception {
   }
   @Override
   public void sessionCreated(IoSession session) throws Exception {
     System.out.println("session created");
   }
   @Override
   public void sessionOpened(IoSession session) throws Exception {
     System.out.println("session opened "+session.getRemoteAddress().toString());
   }
   @Override
   public void sessionClosed(IoSession session) throws Exception {
     System.out.println("session closed "+session.getRemoteAddress().toString());
   }
   /**
   * On idle, we just write a message on the console
   */
   @Override
   public void sessionIdle( IoSession session, IdleStatus status ) throws Exception
   {
     System.out.println( "IDLE " + session.getIdleCount( status ));
   }
 }
```

### **MinaTimerClient**

```java
package com.pbs.common.Cilent;

import org.apache.mina.core.future.ConnectFuture;
 import org.apache.mina.core.service.IoConnector;
 import org.apache.mina.core.session.IoSession;
 import org.apache.mina.filter.codec.ProtocolCodecFilter;
 import org.apache.mina.filter.codec.textline.TextLineCodecFactory;
 import org.apache.mina.filter.logging.LoggingFilter;
 import org.apache.mina.transport.socket.nio.NioSocketConnector;

import java.net.InetSocketAddress;
 import java.nio.charset.Charset;
 import java.util.Scanner;

/**
 * Created by lifei7 on 2016/5/24.
 */
 public class MinaTimerClient {
   private static final int PORT = 9123;

 public static void main(String[] args) {
     IoConnector connector = new NioSocketConnector();
     connector.getFilterChain().addLast("logger",new LoggingFilter());
     connector.getFilterChain().addLast( "codec", 
     		new ProtocolCodecFilter(new TextLineCodecFactory(Charset.forName("UTF-8"))));
     connector.setHandler(new TimeClientHandler());
     ConnectFuture connectFuture = connector.connect(new InetSocketAddress("127.0.0.1",PORT));
     //等待建立连接
     connectFuture.awaitUninterruptibly();
     System.out.println("连接成功");
	 IoSession session = connectFuture.getSession();
	 Scanner sc = new Scanner(System.in);
     boolean quit = false;
     while(!quit){
		String str = sc.next();
		if(str.equalsIgnoreCase("quit")){
         quit = true;
        }
        session.write(str);
        }
	//关闭
     if(session!=null){
       if(session.isConnected()){
         session.getCloseFuture().awaitUninterruptibly();
       }
       connector.dispose(true);
     }
   }
 }
```

### **TimeClientHandler**

```java
package com.pbs.common.Cilent;
import org.apache.mina.core.service.IoHandler;
import org.apache.mina.core.session.IdleStatus;
import org.apache.mina.core.session.IoSession;

/**
 * Created by lifei7 on 2016/5/24.
 */
public class TimeClientHandler implements IoHandler {
  @Override
  public void sessionCreated(IoSession session) throws Exception {
  }

  @Override
  public void sessionOpened(IoSession session) throws Exception {
    System.out.println("client session open:服务端地址"+session.getRemoteAddress().toString());
  }

  @Override
  public void sessionClosed(IoSession session) throws Exception {
   System.out.println("client session closed:服务端地址"+session.getRemoteAddress().toString());
  }

  @Override
  public void sessionIdle(IoSession session, IdleStatus status) throws Exception {
    System.out.println( "IDLE " + session.getIdleCount( status ));
  }

  @Override
  public void exceptionCaught(IoSession session, Throwable cause) throws Exception {
    cause.printStackTrace();
  }

  @Override
  public void messageReceived(IoSession session, Object message) throws Exception {
    System.out.println("client接受信息:"+message.toString());
  }
 
  @Override
  public void messageSent(IoSession session, Object message) throws Exception {
    System.out.println("client发送信息:"+message.toString());
  }

  @Override
  public void inputClosed(IoSession session) throws Exception {
    System.out.println("input closed");
  }
}
```

