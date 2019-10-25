# Hadoop介绍

### Hadoop的组成

**hadoop分为四部分：**

- **Hadoop Common**: The common utilities that support the other Hadoop modules.

  （支持其他Hadoop模块的常用实用程序。 ）

- **Hadoop Distributed File System (HDFS™)**: A distributed file system that provides high-throughput access to application data.

  （一种分布式文件系统，可提供对应用程序数据的高吞吐量访问。 ）

- **Hadoop YARN**: A framework for job scheduling and cluster resource management.

  （作业调度和集群资源管理的框架。 ）

- **Hadoop MapReduce**: A YARN-based system for parallel processing of large data sets.

  （基于YARN的系统，用于并行处理大型数据集。 ）

  下面两部分是新增加的：

- **Hadoop Ozone**: An object store for Hadoop.

- **Hadoop Submarine**: A machine learning engine for Hadoop.

### hadoop的集群模式

```
[Local (Standalone) Mode] ： 运行在单个jvm中，适用于bug调试。(操作linux本地文件系统中的文件或者目录)
[Pseudo-Distributed Mode] ： 将hadoop中的各个服务都运行在一台服务器上，(操作hdfs文件系统中的文件或者目录)
[Fully-Distributed Mode] ： 将hadoop中的各个服务分别运行在多台服务器中。
```

#### 本地模式

~~~
本地模式-默认模式。
-不对配置文件进行修改。
-使用本地文件系统，而不是分布式文件系统。
-Hadoop不会启动NameNode、DataNode、ResourceManager、NodeManager等守护进程，Map()和Reduce()任务作为同一个进程的不同部分来执行的。-用于对MapReduce程序的逻辑进行调试，确保程序的正确。
~~~

#### 伪分布式

~~~
伪分布式
-在一台主机模拟多主机。
-Hadoop启动NameNode、DataNode、ResourceManager、NodeManager这些守护进程都在同一台机器上运行，是相互独立的Java进程。
-在这种模式下，Hadoop使用的是分布式文件系统，各个作业也是由ResourceManager服务，来管理的独立进程。在单机模式之上增加了代码调试功能，允许检查内存使用情况，HDFS输入输出，以及其他的守护进程交互。类似于完全分布式模式，因此，这种模式常用来开发测试Hadoop程序的执行是否正确。
-修改3个配置文件：core-site.xml（Hadoop集群的特性，作用于全部进程及客户端）、hdfs-site.xml（配置HDFS集群的工作属性）、mapred-site.xml（配置MapReduce集群的属性）
-格式化文件系统
~~~

#### 完全分布式

~~~
3.完全分布式
-Hadoop的守护进程运行在由多台主机搭建的集群上，是真正的生产环境。
-在所有的主机上安装JDK和Hadoop，组成相互连通的网络。
-在主机间设置SSH免密码登录，把各从节点生成的公钥添加到主节点的信任列表。
-修改3个配置文件：core-site.xml、hdfs-site.xml、mapred-site.xml，指定NameNode和ResourceManager的位置和端口，设置文件的副本等参数
-格式化文件系统
~~~

### Hadoop进程详解 (高可用)

~~~
Datanode
文件系统的工作节点 根据客户端 或者namenode的调度和检索数据 并且定期向 namenode发送他们所存
储的块的列表

QuorumPeerMain
zookeeper集群的启动入口类 加载配置QuorumPeer线程（分布式协调可持续服务）；
分布式协调可持续服务 ：zookeeper服务数量确定 统一选举 统一管理 协调服务

JournalNode
两个NameNode的数据同步 通过该进程进行通信 当active状态的Namenode命名空间有更改时会告知大部
分的JournalNode

NodeManager
运行在单个节点的代理，他管理Hadoop集群的单个计算节点的 包括和ResourceManager的通信

ResourceManager
Yarn集群主控节点，负责协调和管理整个集群（所有NodeManager）的资源 协调计算资源

DFSZKFailoverController
Hadoop中HDFS NameNode HA实现的中心组件，它负责整体的故障转移控制等
~~~

### Hadoop的常见端口号

~~~
NameNode:50070 HDFS的webUI端口
         19888 JobHistory 
         8088  Yarn的WebUI端口   
~~~

### RPC 通信框架的编写 

#### 接口类 

~~~java
package SimulationHeart;/*
@author Yuniko
2019/10/24
*/
public interface RpcAgreement {
    //协议版本号
    public static final int versionID = 1;
    public String dialogue(String s);
}

~~~

#### NameNode

~~~
package SimulationHeart;/*
@author Yuniko
2019/10/24
*/


import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.ipc.RPC;

import java.io.IOException;

public class NameNode implements RpcAgreement {

    @Override
    public String dialogue(String s) {
        System.out.println("接收到dataNode发来的心跳了");
        return "receive";
    }

    public static void main(String[] args) throws IOException {

            //获取一个服务
            System.out.println("获取Server服务");
            RPC.Server server =  new RPC.Builder(new Configuration())
                    .setInstance(new NameNode())
                    .setProtocol(RpcAgreement.class) //设置协议
                    .setBindAddress("127.0.0.1")
                    .setPort(6666)
                    .build();
            //启动服务
            System.out.println("启动Server服务");
            server.start();
            System.out.println("server is started。。。。。");

    }
}

~~~

#### DataNode

~~~java
package SimulationHeart;/*
@author Yuniko
2019/10/24
*/

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.ipc.RPC;

import java.io.IOException;
import java.net.InetSocketAddress;

public class DataNode {


    public static void main(String[] args) throws IOException, InterruptedException {
        RpcAgreement rpcAgreement = RPC.getProxy(RpcAgreement.class,
                                                      1,
                                                 new InetSocketAddress("127.0.0.1", 6666),
                                                 new Configuration());
         while (true){
             rpcAgreement.dialogue("DataNode 发送心跳!!!");
             // 每隔3秒发送一次心跳
             Thread.sleep(3000);
         }
    }
}

~~~

