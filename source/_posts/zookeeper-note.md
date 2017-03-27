title: ZooKeeper 学习笔记
category: TVDCR
date: 2015-12-10
tags: [spring, transacion, dynamic reconfiguration]
author: Hooting
---
ZooKeeper是一个典型的分布式数据一致性的解决方案，分布式应用可以基于它实现诸如数据发布／订阅、负责均衡、命名服务、分布式协调／通知、
集群管理等功能。 ZooKeeper可以保证如下分布式一致性特征。

<!--more-->

---

## ZooKeeper是什么
ZooKeeper是一个典型的分布式数据一致性的解决方案，分布式应用可以基于它实现诸如数据发布／订阅、负责均衡、命名服务、分布式协调／通知、
集群管理等功能。 ZooKeeper可以保证如下分布式一致性特征。

### 顺序一致性
从同一个客户端发起的事务请求，最终将会严格地按照其发起顺序被应用到ZooKeeper中去

### 原子性
所有事务请求的处理结果在整个集群中所有机器上的应用情况是一致的，要么整个集群所有机器都成功应用了某一个事务，要么都没有应用

### 单一视图
无论客户端连接的是哪个ZooKeeper服务器，其看到的服务端数据模型都是一致的。

### 可靠性
一旦服务端成功地应用了一个事务，并完成对客户端的响应，那么该事务所引起的服务端状态变更将会被一直保留下来，除非有另一个事务又对其进行了变更。

### 实时性
ZooKeeper仅仅保证在一定的时间内，客户端最终一定能够从服务器端上读取到最新的数据状态。


## ZooKeeper提供了什么
简单的说，ZooKeeper=文件系统+通知机制。

### 文件系统
ZooKeeper维护一个类似文件系统的数据结构

![](/images/zookeeper-file-structure.png)

每个子目录项如 NameService 都被称作为znode，和文件系统一样，我们能够自由的增加、删除znode，在一个znode下增加、删除子znode，唯一的不同在于znode是可以存储数据的。有四种类型的znode：
* PERSISTENT-持久化目录节点
客户端与zookeeper断开连接后，该节点依旧存在
* PERSISTENT_SEQUENTIAL-持久化顺序编号目录节点
客户端与zookeeper断开连接后，该节点依旧存在，只是Zookeeper给该节点名称进行顺序编号
* EPHEMERAL-临时目录节。客户端与zookeeper断开连接后，该节点被删除
4、EPHEMERAL_SEQUENTIAL-临时顺序编号目录节点。客户端与zookeeper断开连接后，该节点被删除，只是Zookeeper给该节点名称进行顺序编号

### 通知机制
客户端注册监听它关心的目录节点，当目录节点发生变化（数据改变、被删除、子目录节点增加删除）时，ZooKeeper会通知客户端。

## 保证分布式数据原子性操作
ZooKeeper中为数据节点（znode）引入了版本的概念，每个数据节点都具有三种类型的版本信息，说明如下表。

版本类型           |  说明
-----------      | ------------------------------------------------------
version          | 当前数据节点数据内容的版本号 
cversion         | 当前数据节点子节点的版本号
aversion         | 当前数据节点ACL变更版本号

下文以其中的version版本类型为例说明。在一个数据节点`/zk`被创建完毕之后，节点的version值是0，表示的含义是“当前节点自创建以后，被更新过0次”。如果现在对该节点的数据内容进行更新操作，则version的值变为1。需要注意的是，即使前后两次变更并没有似的数据内容的值发生变化，version的值依然会变更。

ZooKeeper引入version的作用，主要是实现乐观锁，保证数据原子性操作。乐观锁把控制的事务分成三个阶段：数据读取、写入校验、数据写入，其中写入校验阶段是整个乐观锁控制的关键所在。如何进行写入校验呢？ZooKeeper采用的是JDK中最典型的乐观锁实现——[CAS(Compare and Swap)](https://en.wikipedia.org/wiki/Compare-and-swap)。简单的讲就是：对于值V，每次更新前都会比对其值是否是预期值A，只有符合预期，才会将V原子化地更新到新值B，其中是否符合预期便是乐观锁中的“写入校验”阶段。

下面以一个更新数据实例，演示如何用version保证一致性。假如一个客户端试图进行更新操作，它会携带上次获取到的version值进行更新。而如果在这段时间内，ZooKeeper服务器上该节点的数据已经被其他客户端更新了，那么数据版本一定已发生了变化，因此肯定与客户端携带的version无法匹配，于是便无法更新成功。

```java
import java.util.concurrent.CountDownLatch;

import org.apache.zookeeper.CreateMode;
import org.apache.zookeeper.KeeperException;
import org.apache.zookeeper.WatchedEvent;
import org.apache.zookeeper.Watcher;
import org.apache.zookeeper.Watcher.Event.EventType;
import org.apache.zookeeper.Watcher.Event.KeeperState;
import org.apache.zookeeper.ZooDefs.Ids;
import org.apache.zookeeper.ZooKeeper;
import org.apache.zookeeper.data.Stat;

public class ZKVersionSample implements Watcher{
    private static CountDownLatch connectedSemaphore = new CountDownLatch(1);
    private static ZooKeeper zk;

    public static void main(String[] args) throws Exception {
        String path = "/zk";
        zk = new ZooKeeper("127.0.0.1:2181", 5000, new ZKVersionSample());

        connectedSemaphore.await();
        zk.create(path, "123".getBytes(), Ids.OPEN_ACL_UNSAFE,
                CreateMode.EPHEMERAL);
        zk.getData(path, true, null);
        Stat stat = zk.setData(path, "456".getBytes(), -1);
        System.out.println(stat.getCzxid() + "," + stat.getMzxid() + ","
                + stat.getVersion());

        Stat stat2 = zk.setData(path, "456".getBytes(), stat.getVersion());
        System.out.println(stat2.getCzxid() + "," + stat2.getMzxid() + ","
                + stat2.getVersion());

        try {
            zk.setData(path, "456".getBytes(), stat.getVersion());
        } catch (KeeperException e) {
            System.out.println("Error: " + e.code() + "," + e.getMessage());
        }
    }
    
    @Override
    public void process(WatchedEvent event) {
        if (KeeperState.SyncConnected == event.getState()) {
            if (EventType.None == event.getType() && null == event.getPath()){
                connectedSemaphore.countDown();
            }
        }
    }
}

>>>output
17,18,1
17,19,2
Error: BADVERSION,KeeperErrorCode = BadVersion for /zk
```
上面的示例程序中，进行了三次更新操作，分别使用了不同的version。在第一次更新操作中，使用的版本是“－1”，并且更新成功。在ZooKeeper中，数据版本都是从0开始计数。如果客户端传入的版本参数是“－1”，就是告诉ZooKeeper服务端，客户端需要基于数据的最新版本进行更新操作。

在第一次更新操作成功后，ZooKeeper服务端回返回给客户端一个数据节点的节点状态信息对象：stat。在第二次更新操作中，我们在接口中传入了这个版本号，此时的数据版本变更为“2”。

在第三次操作的时候，程序依然使用了之前的版本“1”进行更新操作，于是更新失败了。

## Java客户端API使用
ZooKeeper作为一个分布式服务框架，主要用来解决分布式数据一致性问题，它提供了简单的分布式原语，并且对多种变种语言提供了API。下面只介绍ZooKeeper的Java客户端API使用方式

### 创建会话
客户端可以通过创建一个ZooKeeper实例来连接ZooKeeper服务器。ZooKeeper的四种构造方法如下。

```java
ZooKeeper(String connectString, int sessionTimeout, Watcher watcher);
ZooKeeper(String connectString, int sessionTimeout, Watcher watcher, boolean canBeReadOnly);
ZooKeeper(String connectString, int sessionTimeout, Watcher watcher, long sessionId, byte[] sessionPasswd);
ZooKeeper(String connectString, int sessionTimeout, Watcher watcher, long sessionId, byte[] sessionPasswd, boolean canBeReadOnly);
```
以下是ZooKeeper构造方法参数说明

 参数名           |  说明
-----------      | ------------------------------------------------------
connectString    |  指ZooKeeper服务器列表，由host:port字符串组成。也可以在connectString中设置客户端连接上ZooKeeper后的根目录。
sessionTimeout   | 指会话的超时时间，单位毫秒
watcher          | 事件通知器，可以为null
canBeReadOnly    | 用于标示当前会话是否支持"read－only"模式
sessionId        | 和sessionPasswd分别代表会话ID和会话秘钥。客户端使用这两个参数可以实现客户端会话复用

ZooKeeper客户端和服务器会话的建立是一个异步的过程。当会话真正创建完毕之后，ZooKeeper服务端会想会话对应的客户端发送一个事件通知。

创建一个最基本的ZooKeeper会话实例

```java
import java.io.IOException; 
import java.util.concurrent.CountDownLatch; 
 
import org.apache.zookeeper.CreateMode; 
import org.apache.zookeeper.KeeperException; 
import org.apache.zookeeper.WatchedEvent; 
import org.apache.zookeeper.Watcher; 
import org.apache.zookeeper.Watcher.Event.KeeperState; 
import org.apache.zookeeper.ZooDefs.Ids; 
import org.apache.zookeeper.ZooKeeper; 
 
 
public class ZKBootSample{ 
 
    private static final int SESSION_TIMEOUT = 10000; 
    private static final String CONNECTION_STRING = "127.0.0.1:2181"; 
    private static final String ZK_PATH = "/nileader"; 
    private ZooKeeper zk = null; 
     
 	private CountDownLatch connectedSemaphore = new CountDownLatch( 1 ); 
 
    /** 
     * 创建ZK连接 
     * @param connectString  ZK服务器地址列表 
     * @param sessionTimeout   Session超时时间 
     */ 
    public void createConnection( String connectString, int sessionTimeout ) { 
        try { 
            zk = new ZooKeeper( connectString, sessionTimeout, this ); 
            connectedSemaphore.await(); 
        } catch ( InterruptedException e ) { 
            System.out.println( "连接创建失败，发生 InterruptedException" ); 
            e.printStackTrace(); 
        } catch ( IOException e ) { 
            System.out.println( "连接创建失败，发生 IOException" ); 
            e.printStackTrace(); 
        } 
    } 
 
    public static void main( String[] args ) { 
 
        ZKBootSample sample = new ZKBootSample(); 
        sample.createConnection( CONNECTION_STRING, SESSION_TIMEOUT ); 
    } 
 
 
} 
```

### 创建节点
客户端可以通过ZooKeeper的API来创建一个数据节点，有如下两个接口：

```java
String create(final String path, byte data[], List<ACL> acl, CreateMode createMode);
void create(final String path, byte data[], List<ACL> acl, CreateMode createMode, StringCallback cb, Object ctx);
```

这两个接口分别以同步和异步方式创建节点，参数说明如下表。


参数名            |  说明
-----------      | ------------------------------------------------------
path             |  需要创建的数据节点的节点路径，例如／zk/foo
data[]           |  一个字节数组，是创建节点后的初始内容
acl              |  节点的ACL策略
createMode       | 节点类型，有持久、持久顺序、临时、临时顺序
cb               | 异步回调函数
ctx              | 用于传递一个对象，在回调方法执行的时候使用

无论是同步还是异步接口，ZooKeeper都不支持递归创建，即无法在父节点不存在的情况下创建一个子节点。如果一个节点已经存在，那么创建同名节点的时候，会抛出NodeExistsException异常。

ZooKeeper的节点内容只支持字节数组（byte[]）类型，不负责为节点内容进行序列化。对于字符串，可以简单的使用"string".getBytes()来生成一个字节数组，对于其他复杂对象，可以使用Hessian或是Kryo等序列化工具进行序列化。

使用同步API创建一个节点

```java
import java.io.IOException; 
import java.util.concurrent.CountDownLatch; 
 
import org.apache.zookeeper.CreateMode; 
import org.apache.zookeeper.KeeperException; 
import org.apache.zookeeper.WatchedEvent; 
import org.apache.zookeeper.Watcher; 
import org.apache.zookeeper.Watcher.Event.KeeperState; 
import org.apache.zookeeper.ZooDefs.Ids; 
import org.apache.zookeeper.ZooKeeper; 
 

public class ZKBootSample implements Watcher { 
 
    private static final int SESSION_TIMEOUT = 10000; 
    private static final String CONNECTION_STRING = "127.0.0.1:2181"; 
    private static final String ZK_PATH = "/nileader"; 
    private ZooKeeper zk = null;

    private CountDownLatch connectedSemaphore = new CountDownLatch( 1 );  
     
    /** 
     *  创建节点 
     * @param path 节点path 
     * @param data 初始数据内容 
     * @return 
     */ 
    public boolean createPath( String path, String data ) { 
        try { 
            System.out.println( "节点创建成功, Path: " 
                    + this.zk.create( path, // 
                                              data.getBytes(), // 
                                              Ids.OPEN_ACL_UNSAFE, // 
                                              CreateMode.EPHEMERAL ) 
                    + ", content: " + data ); 
        } catch ( KeeperException e ) { 
            System.out.println( "节点创建失败，发生KeeperException" ); 
            e.printStackTrace(); 
        } catch ( InterruptedException e ) { 
            System.out.println( "节点创建失败，发生 InterruptedException" ); 
            e.printStackTrace(); 
        } 
        return true; 
    } 
 
    public static void main( String[] args ) { 
 
        ZKBootSample sample = new ZKBootSample(); 
        sample.createConnection( CONNECTION_STRING, SESSION_TIMEOUT ); 

        if ( sample.createPath( ZK_PATH, "我是节点初始内容" ) ) { 
            //do something
        } 
 
    } 
} 
```
### 删除节点
客户端可以通过ZooKeeper的API来删除一个节点，有如下两个接口。

```java
public void delete(final String path, int version)
public void delete(final String path, int version, VoidCallBack cb, Object ctx)
```

参数名            |  说明
-----------      | ------------------------------------------------------
path             |  需要删除的数据节点的节点路径，例如／zk/foo
version          |  指定节点的数据版本，即表明本次删除操作是针对该数据版本进行的
cb               | 异步回调函数
ctx              | 用于传递一个对象，在回调方法执行的时候使用

```java
import java.io.IOException; 
import java.util.concurrent.CountDownLatch; 
 
import org.apache.zookeeper.CreateMode; 
import org.apache.zookeeper.KeeperException; 
import org.apache.zookeeper.WatchedEvent; 
import org.apache.zookeeper.Watcher; 
import org.apache.zookeeper.Watcher.Event.KeeperState; 
import org.apache.zookeeper.ZooDefs.Ids; 
import org.apache.zookeeper.ZooKeeper; 
 

public class ZKBootSample implements Watcher { 
 
    private static final int SESSION_TIMEOUT = 10000; 
    private static final String CONNECTION_STRING = "127.0.0.1:2181"; 
    private static final String ZK_PATH = "/nileader"; 
    private ZooKeeper zk = null;

    private CountDownLatch connectedSemaphore = new CountDownLatch( 1 );  

    /** 
     * 删除指定节点 
     * @param path 节点path 
     */ 
    public void deleteNode( String path ) { 
        try { 
            this.zk.delete( path, -1 ); 
            System.out.println( "删除节点成功，path：" + path ); 
        } catch ( KeeperException e ) { 
            System.out.println( "删除节点失败，发生KeeperException，path: " + path  ); 
            e.printStackTrace(); 
        } catch ( InterruptedException e ) { 
            System.out.println( "删除节点失败，发生 InterruptedException，path: " + path  ); 
            e.printStackTrace(); 
        } 
    }  
 
    public static void main( String[] args ) { 
 
        ZKBootSample sample = new ZKBootSample(); 
        sample.createConnection( CONNECTION_STRING, SESSION_TIMEOUT ); 

        if ( sample.createPath( ZK_PATH, "我是节点初始内容" ) ) { 
            //do something
            sample.deleteNode( ZK_PATH ); 
        } 
 
    } 
} 
```


### 读取数据
读取数据， 包括自节点列表的获取和节点数据的获取。ZooKeeper分别提供了不同的API来获取数据。

#### getChildren
客户端可以通过ZooKeeper的API来获取一个节点的所有子节点，有如下4个接口可以使用

```java
List<String> getChildren(final String path, Watcher watcher);
List<String> getChildren(final String path, boolean watch);
void getChildren(final String path, Watcher watcher, ChildrenCallback cb, Object ctx);
List<String> getChildren(final String path, Watcher watcher, Stat stat);
```

参数名            |  说明
-----------      | ------------------------------------------------------
path             |  指定数据节点的节点路径，例如／zk/foo
watcher          |  注册的Watcher。一旦本次子节点获取之后，当子节点列表发生变更，那么就会向客户端发送通知。该参数允许传入null
watch            |  表明是否需要一个Watcher。如果这个参数是true，那么ZooKeeper客户端会自动使用程序上文中提到的那个默认Watcher
cb               | 异步回调函数
ctx              | 用于传递一个对象，在回调方法执行的时候使用
stat             | 指定数据节点的节点状态。用法是在接口中传入一个旧的stat变量，该stat变量会在方法执行过程中，被来自服务端响应的新stat对象替换

stat用于描述节点状态信息，该对象纪录了一个节点的基本属性信息，例如节点创建时的事务ID、最后一次修改的事务ID和节点数据内容的长度等。

#### getData
客户端可以通过ZooKeeper的API来获取一个节点的所有子节点，有如下4个接口可以使用

```java
byte[] getData(final String path, Watcher watcher, Stat stat);
byte[] getData(final String path, boolean watch, Stat stat);
void getData(final String path, Watcher watcher, DataCallback cb, Object ctx);
void getData(final String path, boolean watch, DataCallback cb, Object ctx);
```

参数说明痛getChildren。一个简单的读取数据实例：

```java
import java.io.IOException; 
import java.util.concurrent.CountDownLatch; 
 
import org.apache.zookeeper.CreateMode; 
import org.apache.zookeeper.KeeperException; 
import org.apache.zookeeper.WatchedEvent; 
import org.apache.zookeeper.Watcher; 
import org.apache.zookeeper.Watcher.Event.KeeperState; 
import org.apache.zookeeper.ZooDefs.Ids; 
import org.apache.zookeeper.ZooKeeper; 
 

public class ZKBootSample implements Watcher { 
 
    private static final int SESSION_TIMEOUT = 10000; 
    private static final String CONNECTION_STRING = "127.0.0.1:2181"; 
    private static final String ZK_PATH = "/nileader"; 
    private ZooKeeper zk = null;

    private CountDownLatch connectedSemaphore = new CountDownLatch( 1 );  
 
 	/** 
     * 读取指定节点数据内容 
     * @param path 节点path 
     * @return 
     */ 
    public String readData( String path ) { 
        try { 
            System.out.println( "获取数据成功，path：" + path ); 
            return new String( this.zk.getData( path, false, null ) ); 
        } catch ( KeeperException e ) { 
            System.out.println( "读取数据失败，发生KeeperException，path: " + path  ); 
            e.printStackTrace(); 
            return ""; 
        } catch ( InterruptedException e ) { 
            System.out.println( "读取数据失败，发生 InterruptedException，path: " + path  ); 
            e.printStackTrace(); 
            return ""; 
        } 
    } 
 
    public static void main( String[] args ) { 
 
        ZKBootSample sample = new ZKBootSample(); 
        sample.createConnection( CONNECTION_STRING, SESSION_TIMEOUT ); 

        if ( sample.createPath( ZK_PATH, "我是节点初始内容" ) ) { 
            //do something
            System.out.println( "数据内容: " + sample.readData( ZK_PATH ) + "\n" ); 
        } 
 
    } 
} 
```

