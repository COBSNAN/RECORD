---
title: 2018-3-6未命名文件 
tags: java
grammar_cjkRuby: true
---


# 搭建Tomcat源码项目
- 下载tomcat源代码 [下载地址][1]
- 编写一个pom.xml 文件，给Tomcat使用。
	```xml
	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0"
		xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
		xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

		<modelVersion>4.0.0</modelVersion>
		<groupId>org.apache.tomcat</groupId>
		<artifactId>Tomcat8.0</artifactId>
		<name>Tomcat8.0</name>
		<version>8.0</version>

		<build>
			<finalName>Tomcat8.0</finalName>
			<sourceDirectory>java</sourceDirectory>
			<testSourceDirectory>test</testSourceDirectory>
			<resources>
				<resource>
					<directory>java</directory>
				</resource>
			</resources>
			<testResources>
				<testResource>
					<directory>test</directory>
				</testResource>
			</testResources>
			<plugins>
				<plugin>
					<groupId>org.apache.maven.plugins</groupId>
					<artifactId>maven-compiler-plugin</artifactId>
					<version>2.3</version>
					<configuration>
						<encoding>UTF-8</encoding>
						<source>1.8</source>
						<target>1.8</target>
					</configuration>
				</plugin>
			</plugins>
		</build>

		<dependencies>
			<dependency>
				<groupId>junit</groupId>
				<artifactId>junit</artifactId>
				<version>4.12</version>
				<scope>test</scope>
			</dependency>
			<dependency>
				<groupId>org.easymock</groupId>
				<artifactId>easymock</artifactId>
				<version>3.4</version>
			</dependency>
			<dependency>
				<groupId>ant</groupId>
				<artifactId>ant</artifactId>
				<version>1.7.0</version>
			</dependency>
			<dependency>
				<groupId>wsdl4j</groupId>
				<artifactId>wsdl4j</artifactId>
				<version>1.6.2</version>
			</dependency>
			<dependency>
				<groupId>javax.xml</groupId>
				<artifactId>jaxrpc</artifactId>
				<version>1.1</version>
			</dependency>
			<dependency>
				<groupId>org.eclipse.jdt.core.compiler</groupId>
				<artifactId>ecj</artifactId>
				<version>4.5.1</version>
			</dependency>
		</dependencies>
	</project>
	```
- 在idea中导入Tomcat工程后，需要给Bootstrap运行时的CATALINA_HOME目录，在Tomcat项目的平级目录下建一个catalina-home目录，这时需要下载一个Binary 包，解压后把一些文件拷贝到catalina-home目录中。
  > bin
	conf
	lib
	logs
	temp
	webapps
	work

- 配置Bootstrap运行环境
	> 在Man class:中填入，org.apache.catalina.startup.Bootstrap
		在VM options:中填入设置catalina-home路径,我的路径是`-Dcatalina.home="E:\tomcat\catalina-home"`
		
- 这是启动时，会报一些架包不存在错误，可以先删除test路径的配置


# 工程目录
![tomcat目录][2]
- javax：Java中标准的Servlet规范。
- catalina：Tomcat中的Servlet容器。
- coyote：Tomcat中的连接器。
- el：正则表达式解析规范。
- jasper：JSP文件解析规范。
- juli：Tomcat的日志系统。
- naming：JNDI的实现。
- tomcat：Tomcat的工具包。


# Tomcat中三种部署项目的方法
- 在tomcat中的conf目录中，在server.xml中<host/>节点下添加Context节点
- 将web项目文件件拷贝到webapps 目录中
- 在conf\Catalina\localhost目录下建一个文件，文件名和项目名一致，主要是在安全模式下部署

# 配置文件
- server.xml: Tomcat的主配置文件，包含Service, Connector, Engine, Realm, Valve, Hosts主组件的相关配置信息；
- web.xml：遵循Servlet规范标准的配置文件，用于配置servlet，并为所有的Web应用程序提供包括MIME映射等默认配置信息；
- tomcat-user.xml：Realm认证时用到的相关角色、用户和密码等信息；Tomcat自带的manager默认情况下会用到此文件；
- catalina.policy：Java相关的安全策略配置文件，在系统资源级别上提供访问控制的能力；
- catalina.properties：Tomcat内部package的定义及访问相关的控制，也包括对通过类装载器装载的内容的控制；
- logging.properties: Tomcat内部实现的JAVA日志记录器来记录操作相关的日志，此文件即为日志记录器相关的配置信息，可以用来定义日志记录的组件级别以及日志文件的存在位置等；
- context.xml：所有host的默认配置信息；

#### catalina.policy文件
> 标准权限

- java.util.PropertyPermission——控制对 JVM 属性的读/写，比如说 java.home。
- java.lang.RuntimePermission——控制一些系统/运行时函数的使用，比如 exit() 和 exec()。 另外也控制包的访问/定义。
- java.io.FilePermission——控制对文件和目录的读/写/执行。
- java.net.SocketPermission——控制网络套接字的使用。
- java.net.NetPermission——控制组播网络连接的使用。
- java.lang.reflect.ReflectPermission——控制类反射的使用。
- java.security.SecurityPermission——控制对 Security 方法的访问。
- java.security.AllPermission——允许访问任何权限，仿佛没有 SecurityManager。- 

```
// 策略文件项范例  
grant [signedBy <signer>,] [codeBase <code source>] {
  permission  <class>  [<name> [, <action list>]];
};
```
- codeBase 是通过URL的方式指定文件，可以使用变量java.home或者java.home或者{catalina.home}来表示JDK和tomcat的根目录。
- class 指定了相应的权限类型
- [name,[,action]] name指定具体的操作或者文件，action指定可选的动作（比如read write等等）

>进入安全模式
```
$CATALINA_HOME/bin/catalina.sh start -security    (Unix)
%CATALINA_HOME%\bin\catalina start -security  (Windows)
```

启动安全模式后，这时manage 项目起不起来，需要通过虚拟主机目录配置context.xml 文件才能启动。
```
<Context docBase="${catalina.home}/webapps/manager" path="/manager"
         privileged="true" antiResourceLocking="false" antiJARLocking="false">
</Context>
```

```
grant { 
    permission java.io.FilePermission "E:/testfile/nio-data.txt", "read";
    permission java.util.PropertyPermission "file.encoding", "read";
};
```
```
import="java.net.*,java.io.*"
<%
System.out.println("SecurityManager: " + System.getSecurityManager());
String result = "";
try{
    File file = new File("E:\\testfile\\nio-data.txt");
    BufferedReader br = new BufferedReader(new FileReader(file));
    String s = null;
    while((s = br.readLine())!=null){
        result = result + "\n" +s;
    }
    br.close();    
}catch(Exception e){
    e.printStackTrace();
}
System.out.println(result);
System.out.println(System.getProperty("file.encoding"));
%>
```
#### logging.properties 文件
日志输出级别：SEVERE (最高级别) > WARNING > INFO > CONFIG > FINE > FINER > FINEST (所有内容,最低级别)
禁用日志的输出设置为OFF,  输出所有的日志消息设置为ALL

`org.apache.catalina.core.ContainerBase.[Catalina].[localhost].[/host-manager].handlers = 4host-manager.org.apache.juli.AsyncFileHandler`

其中Catalina表示Engine 名，localhost表示Host 名，/host-manager 这个是请求路径

#web.xml文件

>JspServlet

- checkInterval 多长时间监测jsp文件是否修改，进行重新加载
- javaEncoding java文件编码格式
- maxLoadedJsps 应用程序最大加载jsp文件数，默认-1 没有限制。超过数量按最近未使用过jsp进行卸载
- jspIdleTimeout 多长时间未使用就卸载jsp。

# server.xml文件
![server][3]
```
<Server> <!-- 顶级元素 -->
  <Listener/> <!-- 组件 -->
  <GlobalNamingResources/> <!-- 组件 --> 
  <Service> <!-- 另一个顶级元素 -->
    <Connector/> <!-- 接头 -->
    <Engine> <!-- 容器 -->
      <Realm/> <!-- 组件 -->
      <Host> <!-- 容器 -->
        <Valve/> <!-- 组件 -->
          <Context></Context> <!-- 容器 -->
      </Host>
    </Engine>
  </Service>
</Server>
```
### Server(服务器)
> Tomcat的一个实例，通常一个JVM只能包含一个Tomcat实例；因此，一台物理服务器上可以在启动多个JVM的情况下在每一个JVM中启动一个Tomcat实例，每个实例分属于一个独立的管理端口。

Server有两种特有的组件，一个是GlobalNamingResources（全局命名资源），一个是Listener（监听器）可以作用于不同层次容器的组件。除此之外，还有Service（服务）

![StandardServer][4]

### Service(服务)
> 一个服务组件通常包含一个引擎和与此引擎相关联的一个或多个连接器。给服务命名可以方便管理员在日志文件中识别不同服务产生的日志。一个server可以包含多个service组件，但通常情下只为一个server只有一个service。



![StandardService][5]

### Connector(连接器)
>负责侦听一个具体的TCP端口，并通过该端口处理Engine与客户端之间的交互


![Connector][6]

**Connector属性**
- address：指定连接器监听的地址，默认为所有地址，即0.0.0.0；
- maxThreads：支持的最大并发连接数，默认为200；
- port：监听的端口，默认为0；
- redirectPort：如果某连接器支持的协议是HTTP，当接收客户端发来的HTTPS请求时，则转发至此属性定义的端口；
- connectionTimeout：等待客户端发送请求的超时时间，单位为毫秒，默认为60000，即1分钟；
- enableLookups：是否通过request.getRemoteHost()进行DNS查询以获取客户端的主机名；默认为true；
- acceptCount：设置等待队列的最大长度；通常在tomcat所有处理线程均处于繁忙状态时，新发来的请求将被放置于等待队列中；
- URIEncoding:指定对所有GET方式请求进行统一的重新编码（解码）的编码
- useBodyEncodingForURI:是否用request.setCharacterEncoding参数对URL提交的数据和表单中GET方式提交的数据进行重新编码，在默认情况下，该参数为false
- protocol：连接器使用的协议，默认为HTTP/1.1，定义AJP协议时通常为AJP/1.3；
有两种协议`HTTP/1.1`和`AJP/1.3`两种协议
HTTP是主要是否则接受来自客户端的请求，AJP主要是和一个web容器进行交互，比如Apache服务器，目的在于静态资源和动态资源分离。
```
//生产
keytool -genkeypair -alias "tomcat" -keyalg "RSA" -keystore "d:\ssl\tomcat.keystore"
<Connector
           protocol="org.apache.coyote.http11.Http11NioProtocol"
           port="8443" maxThreads="200"
           scheme="https" secure="true" SSLEnabled="true"
           keystoreFile="E:\tomcat\catalina-home\conf\tomcat.keystore" keystorePass="123456"
           clientAuth="false" sslProtocol="TLS"/>
```
![示意图][7]

### Engine(引擎)
>Engine是Service的请求处理引擎，负责处理所有Connector发过来的请求，并将内部处理完毕的结果返回给Connector。

![StandardEngine][8]
**Engine元素属性**
- Engine.name - 引擎的名称
- Engine.defaultHost - 默认采用哪一个子容器Host来处理请求
- Engine.jvmRoute - 一个用于负载均衡场景下的唯一标识符
	- 默认的负载均衡场景下，Tomcat采用一种被称作粘性session的机制管理session信息。

### Realm
> 用于用户的认证和授权；在配置一个应用程序时，管理员可以为每个资源或资源组定义角色及权限，而这些访问控制功能的生效需要通过Realm来实现。Realm的认证可以基于文本文件、数据库表、LDAP服务等来实现。Realm的效用会遍及整个引擎或顶级容器，因此，一个容器内的所有应用程序将共享用户资源。同时，Realm可以被其所在组件的子组件继承，也可以被子组件中定义的Realm所覆盖。

- JDBCRealm 用户授权信息存储于某个关系型数据库中，通过JDBC驱动获取信息验证
- DataSourceRealm 用户授权信息存储于关于型数据中，通过JNDI配置JDBC数据源的方式获取信息验证
- JNDIRealm  用户授权信息存储在基于LDAP的目录服务的服务器中，通过JNDI驱动获取并验证
- UserDatabaseRealm 默认的配置方式，信息存储于XML文档中 conf/tomcat-users.xml
- MemoryRealm 用户信息存储于内存的集合中，对象集合的数据来源于xml文档 conf/tomcat-users.xml
- JAASRealm 通过JAAS框架访问授权信息

### Host(主机)

![StandardHost][9]

**Host元素属性**
- appBase：存放非归档的web应用程序的目录或归档后的WAR文件的目录路径；可以使用基于$CATALINA_HOME的相对路径；
- autoDeploy：在Tomcat处于运行状态时放置于appBase目录中的应用程序文件是否自动进行deploy；默认为true；
- unpackWars：在启用此webapps时是否对WAR格式的归档文件先进行展开；默认为true；
- xmlBase：虚拟主机目录。  如果没有指定默认的conf/<engine_name>/<host_name>
- deployXML：是否部署项目下Context.xml 文件(位于/META-INF/context.xml)。安全模式下设置为false

### Context(上下文)
> Context组件是最内层次的组件，它表示Web应用程序本身。配置一个Context最主要的是指定Web应用程序的根目录，以便Servlet容器能够将用户请求发往正确的位置。

![StandardContext][10]

**Context元素属性**
- docBase:应用程序的路径或者是WAR文件存放的路径
- path:此web应用程序的上下文路径
- reloadable:是否支持热部署

### Valve(阀门)
> Valve的中文含义是阀门，可以简单地理解为Tomcat的拦截器。它负责在请求发送到应用之前拦截HTTP请求，可以定义在任何容器中。默认配置中定义了一个AccessLogValve，负责拦截HTTP请求，并写入到日志文件中。

### Listener（监听组件）
- AprLifecycleListener 检测apr模式是否支持。useAprConnector 这个属性来表示是否开启apr模式
- GlobalResourcesLifecycleListener 作用于全局资源，保证 JNDI 对资源的可达性，比如数据库
- VersionLoggerListener 监听tomcat启动日志信息 logProps属性表示是否打印系统属性

# 生命周期

> 每一个组件都会存在很多的状态，如初始化、开始、停止等等。而在状态之间的切换需要做很多相应的工作。我们把Tomcat组件的各个状态以及状态之间的转换统称为Tomcat的生命周期。在Tomcat中参与生命周期管理的主要有以下几个类或接口：

- Lifecycle：表示生命周期概念的接口。
- LifecycleListener：用来监听组件状态变化并触发相应事件的监听器。
- LifecycleEvent：组件状态变化时触发的事件。
- LifecycleException：生命周期相关异常。
- LifecycleState：组件的有效状态枚举。
- LifecycleSupport：用来帮助传送消息给监听器的辅助类。
- LifecycleBase：Lifecycle接口的基础实现类，主要实现了制定启动和停止状态相关的转换规则。

## Lifecycle

这个接口声明了能对组件施加影响的生命周期事件类型常量，可以分为在状态中、状态前和状态后（不是每个状态都有这相应的三种情况）。包含的状态有初始化、启动、停止、销毁、配置，以及一个特殊的阶段性状态，具体常量如下：

```
    public static final String BEFORE_INIT_EVENT = "before_init";
    public static final String AFTER_INIT_EVENT = "after_init";
    public static final String START_EVENT = "start";
    public static final String BEFORE_START_EVENT = "before_start";
    public static final String AFTER_START_EVENT = "after_start";
    public static final String STOP_EVENT = "stop";
    public static final String BEFORE_STOP_EVENT = "before_stop";
    public static final String AFTER_STOP_EVENT = "after_stop";
    public static final String AFTER_DESTROY_EVENT = "after_destroy";
    public static final String BEFORE_DESTROY_EVENT = "before_destroy";
    public static final String PERIODIC_EVENT = "periodic";
    public static final String CONFIGURE_START_EVENT = "configure_start";
    public static final String CONFIGURE_STOP_EVENT = "configure_stop";
```

Lifecycle对外提供的方法主要分为两大类，一类是对监听器的管理，另一类是生命周期的各个状态，需要根据状态来做相应动作和触发事件。

```
    //监听器的添加、查找与删除操作
    public void addLifecycleListener(LifecycleListener listener);
    public LifecycleListener[] findLifecycleListeners();
    public void removeLifecycleListener(LifecycleListener listener);

    //触发各个状态相关的生命周期事件来进行准备和善后处理
    public void init() throws LifecycleException;
    public void start() throws LifecycleException;
    public void stop() throws LifecycleException;
    public void destroy() throws LifecycleException;

    //获取组件当前状态
    public LifecycleState getState();
    public String getStateName();
```

实现了Lifecycle接口的非常之多，具体可以去看它继承树，这里给出一些实现或继承该接口的关键类或接口。

![lifecycle-hierarchy](http://upload-images.jianshu.io/upload_images/1980824-b102ae4510f4a35f?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## LifecycleListener

LifecycleListener表示对某一个生命周期事件的监听。这个接口非常简单，只有一个方法。

```
    //定义某一事件发生时需要的行为
    public void lifecycleEvent(LifecycleEvent event);
```

## LifecycleEvent

LifecycleEvent继承自Java的事件对象EventObject，是一个简单的类，由事件相关组件、事件的类型和事件相关数据组成。

```
public final class LifecycleEvent extends EventObject {
    private static final long serialVersionUID = 1L;
    private final Object data;
    private final String type;

    public LifecycleEvent(Lifecycle lifecycle, String type, Object data) {
        super(lifecycle);
        this.type = type;
        this.data = data;
    }

    public Object getData() {
        return (this.data);
    }

    public Lifecycle getLifecycle() {
        return (Lifecycle) getSource();
    }

    public String getType() {
        return (this.type);
    }
}
```

## LifecycleException

LifecycleException表示一个生命周期相关的异常，继承自Java的Exception类，提供了多种不同的构造函数，此外不能自定义它的子类。

```
public final class LifecycleException extends Exception {
    private static final long serialVersionUID = 1L;

    public LifecycleException() {
        super();
    }

    public LifecycleException(String message) {
        super(message);
    }

    public LifecycleException(Throwable throwable) {
        super(throwable);
    }

    public LifecycleException(String message, Throwable throwable) {
        super(message, throwable);
    }
}
```

## LifecycleState

LifecycleState枚举出了一个组件合法的生命周期状态，并对外提供两个参数，一个是获取生命周期事件，即在该状态下应该调用哪类生命周期事件，另一个是在该状态下能否调用组件除了getter/setter和生命周期方法外的其他public方法。

```
public enum LifecycleState {
    NEW(false, null),
    INITIALIZING(false, Lifecycle.BEFORE_INIT_EVENT),
    INITIALIZED(false, Lifecycle.AFTER_INIT_EVENT),
    STARTING_PREP(false, Lifecycle.BEFORE_START_EVENT),
    STARTING(true, Lifecycle.START_EVENT),
    STARTED(true, Lifecycle.AFTER_START_EVENT),
    STOPPING_PREP(true, Lifecycle.BEFORE_STOP_EVENT),
    STOPPING(false, Lifecycle.STOP_EVENT),
    STOPPED(false, Lifecycle.AFTER_STOP_EVENT),
    DESTROYING(false, Lifecycle.BEFORE_DESTROY_EVENT),
    DESTROYED(false, Lifecycle.AFTER_DESTROY_EVENT),
    FAILED(false, null),
    MUST_STOP(true, null),
    MUST_DESTROY(false, null);

    private final boolean available;
    private final String lifecycleEvent;

    private LifecycleState(boolean available, String lifecycleEvent) {
        this.available = available;
        this.lifecycleEvent = lifecycleEvent;
    }

    //能否调用其他公共分方法
    public boolean isAvailable() {
        return available;
    }

    public String getLifecycleEvent() {
        return lifecycleEvent;
    }
}
```
## LifecycleSupport

LifecycleSupport是用来将事件通知给一个组件的所有监听器的辅助类。它包含了组件和该组件所有的监听器。其中监听器用了线程安全的CopyOnWriteArrayList来存储，主要的事件通知函数如下：

```
    public void fireLifecycleEvent(String type, Object data) {
        LifecycleEvent event = new LifecycleEvent(lifecycle, type, data);
        for (LifecycleListener listener : listeners) {
            listener.lifecycleEvent(event);
        }
    }
```

## LifecycleBase

LifecycleBase是实现了Lifecycle的抽象类。它通过持有一个LifecycleSupport来管理监听器。具体的状态函数使用了模板方法模式，LifecycleBase规定了个状态之间的转换规则（是否合法以及状态间的自动转换等，具体参看LifecycleState中的状态转换图），而让用户继承的子类来实现具体的操作。由于代码较多，举一个start方法的例子。


# 启动
![启动图][11]

# 设计模式

### 外观模式

> 外观模式封装了子系统的具体实现，提供统一的外观类给外部系统，这样当子系统内部实现发生变化的时候，不会影响到外部系统。

![外观模式][12]

RequestFacade包装了Request，它们都实现了HttpServletRequest，当传递Request对象给应用的时候，其实是返回了RequestFacade对象，而RequestFacade内部可以根据是否自定义了安全管理器来进行相应的操作。

### 观察者模式
> 观察者模式是一种非常常用的模式，比如大家熟悉的发布-订阅模式，客户端应用中的事件监听器，以及通知等其实都属于观察者模式。观察者模式主要是在当系统中发生某个状态变更或者事件的时候，有另外一些组件或者对象对此次变化有兴趣，这个时候那些对变化感兴趣的对象就可以做为观察者对象来监听变化，而被观察对象要负责发生变化的时候触发通知操作。

![enter description here][13]

Tomcat抽象了一个LifecycleSupport的类，而所有需要生命周期管理的组件通过LifecycleSupport类通知对某个生命周期事件感兴趣的观察者，而所有的观察者都需要实现LifecycleListener。

### 责任链模式
> 通过名称我们应该就能知道责任链模式是解决啥问题的？当我们系统在处理某个请求的时候，请求需要经过很多个节点进行处理，每个节点只关注自己的应该做的工作，做完自己的工作以后，将工作转给下一个节点进行处理，直到所有节点都处理完毕。责任链模式在日常生活中例子挺多，比如快递，当你发一个从深圳到北京的快递的时候，你的包裹会从一个分拨中心传递到下一个分拨中心，直到目的地，这里面每个分拨中心都是链路上的一个节点，它做完自己的工作，然后将工作传递到下一个节点，还比如路由器中传递某个数据包其实也是同样的思路。

![enter description here][14]

每一个容器都会有一个Pipeline，而一个Pipeline又会具有多个Valve阀门，其中StandardEngine对应的阀门是StandardEngineValve，StandardHost对应的阀门是StandardHostValve，StandardContext对应的阀门是StandardContextValve，StandardWrapper对应的阀门是StandardWrapperValve。这里每一Pipeline就好比一个管道，而每一Valve就相当于一个阀门，一个管道可以有多个阀门，而对于阀门来说有两种，一种阀门在处理完自己的事情以后，只需要将工作委托给下一个和自己在同一管道的阀门即可，第二种阀门是负责衔接各个管道的，它负责将请求传递给下个管道的第一个阀门处理，而这种阀门叫Basic阀门，它是每个管道中最后一个阀门，上面的StandardValve都属于第二种阀门。

### 模板方法模式
>模板方法模式抽象出某个业务操作公共的流程，将流程分为几个步骤，其中有一些步骤是固定不变的，有一些步骤是变化的，固定不变的步骤通过一个基类来实现，而变化的部分通过钩子方法让子类去实现，这样就实现了对系统中流程的统一化规范化管理。在日常生活中其实也有类似的例子，比如我们知道的连锁加盟店，他们都是有固定的加盟流程，只不过每一家店开的时候，店的选址，装修等不同的而已，但是总体的加盟流程已经是确定的。

![enter description here][15]

Tomcat中关于生命周期管理的地方很好应用了模板方法模式，在一个组件的生命周期中都会涉及到init(初始化)，start（启动），stop(停止)，destory（销毁），而对于每一个生命周期阶段其实都有固定一些事情要做，比如判断前置状态，设置后置状态，以及通知状态变更事件的监听者等，而这些工作其实是可以固化的，所以Tomcat中就将每个生命周期阶段公共的部分固化，然后通过initInternal,startInternal,stopInternal,destoryInternal这几个钩子方法开放给子类去实现具体的逻辑.

# Connector的三种运行模式
- BIO : 一个线程处理一个请求。缺点：并发量高时，线程数较多，浪费资源。Tomcat7或以下，在Linux系统中默认使用这种方式。
- NIO：利用Java的异步IO处理，可以通过少量的线程处理大量的请求。Tomcat8默认使用这种方式。
- APR :即Apache Portable Runtime，从操作系统层面解决io阻塞问题。Tomcat8在Win7或以上的系统中可以这种方式。而在linux系统中需要额为安装APR库和OpenSSL相关库
![enter description here][16]


  [1]: https://tomcat.apache.org/download-80.cgi
  [2]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517640277301.jpg
  [3]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517732159336.jpg
  [4]: https://www.github.com/COBSNAN/ImageHub/raw/master/StandardServer.png "StandardServer"
  [5]: https://www.github.com/COBSNAN/ImageHub/raw/master/StandardService.png "StandardService"
  [6]: https://www.github.com/COBSNAN/ImageHub/raw/master/Connector.png "Connector"
  [7]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517743979552.jpg
  [8]: https://www.github.com/COBSNAN/ImageHub/raw/master/StandardEngine.png "StandardEngine"
  [9]: https://www.github.com/COBSNAN/ImageHub/raw/master/StandardHost.png "StandardHost"
  [10]: https://www.github.com/COBSNAN/ImageHub/raw/master/StandardContext.png "StandardContext"
  [11]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517789479176.jpg
  [12]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517876786509.jpg
  [13]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517877074256.jpg
  [14]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517877159404.jpg
  [15]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517877278817.jpg
  [16]: https://www.github.com/COBSNAN/ImageHub/raw/master/1517928617040.jpg