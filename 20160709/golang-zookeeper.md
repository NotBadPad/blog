#golang的zk客户端
最近打算写个简单的配置中心，考虑到实现便捷性，语言选择了go，由于其中计划用到zk，就调研了下golang的zk客户端，并实现了个简单的分布式server。最终找到了两个，地址如下： 

* gozk：https://wiki.ubuntu.com/gozk   
* go-zookeeper：https://github.com/samuel/go-zookeeper

由于gozk的文档不如后者，且代码没在gihub上，所以就直接选择了后者。go-zookeeper文档还是比较全面的：[文档](http://godoc.org/github.com/samuel/go-zookeeper/zk)

#基本操作测试
这里默认大家已经了解zk的用处和基本用法了，如果还不了解可以参看：[官方文档](https://cwiki.apache.org/confluence/display/ZOOKEEPER/Index)或[中文文档](http://zookeeper.majunwei.com/)
下边我们先来写个简单的例子来测试下基本的操作：

```  golang 
package main

/**
客户端doc地址：github.com/samuel/go-zookeeper/zk
**/
import (
	"fmt"
	zk "github.com/samuel/go-zookeeper/zk"
	"time"
)

/**
 * 获取一个zk连接
 * @return {[type]}
 */
func getConnect(zkList []string) (conn *zk.Conn) {
	conn, _, err := zk.Connect(zkList, 10*time.Second)
	if err != nil {
		fmt.Println(err)
	}
	return
}

/**
 * 测试连接
 * @return
 */
func test1() {
	zkList := []string{"localhost:2183"}
	conn := getConnect(zkList)

	defer conn.Close()
	conn.Create("/go_servers", nil, 0, zk.WorldACL(zk.PermAll))

	time.Sleep(20 * time.Second)
}

/**
 * 测试临时节点
 * @return {[type]}
 */
func test2() {
	zkList := []string{"localhost:2183"}
	conn := getConnect(zkList)

	defer conn.Close()
	conn.Create("/testadaadsasdsaw", nil, zk.FlagEphemeral, zk.WorldACL(zk.PermAll))

	time.Sleep(20 * time.Second)
}

/**
 * 获取所有节点
 */
func test3() {
	zkList := []string{"localhost:2183"}
	conn := getConnect(zkList)

	defer conn.Close()

	children, _, err := conn.Children("/go_servers")
	if err != nil {
		fmt.Println(err)
	}
	fmt.Printf("%v \n", children)
}

func main() {
	test3()
}

```  

上边的代码里边我们测试了golang对zk的连接、创建节点、或取节点操作，在下边的分布式server中将用到这几个操作。

#简单的分布式server
目前分布式系统已经很流行了，一些开源框架也被广泛应用，如dubbo、Motan等。对于一个分布式服务，最基本的一项功能就是服务的注册和发现，而利用zk的EPHEMERAL节点则可以很方便的实现该功能。EPHEMERAL节点正如其名，是临时性的，其生命周期是和客户端会话绑定的，当会话连接断开时，节点也会被删除。下边我们就来实现一个简单的分布式server：    
server：

1. 服务启动时，创建zk连接，并在go_servers节点下创建一个新节点，节点名为"ip:port"，完成服务注册
2. 服务结束时，由于连接断开，创建的节点会被删除，这样client就不会连到该节点

client：

1. 先从zk获取go_servers节点下所有子节点，这样就拿到了所有注册的server
2. 从server列表中选中一个节点（这里只是随机选取，实际服务一般会提供多种策略），创建连接进行通信

这里为了演示，我们每次client连接server，获取server发送的时间后就断开。主要代码如下：
zkutil:用来处理zk操作

```  golang
func GetConnect() (conn *zk.Conn, err error) {
	conn, _, err = zk.Connect(hosts, timeOut*time.Second)
	if err != nil {
		fmt.Println(err)
	}
	return
}

func RegistServer(conn *zk.Conn, host string) (err error) {
	_, err = conn.Create("/go_servers/"+host, nil, zk.FlagEphemeral, zk.WorldACL(zk.PermAll))
	return
}

func GetServerList(conn *zk.Conn) (list []string, err error) {
	list, _, err = conn.Children("/go_servers")
	return
}
```

server: server端操作

``` golang
func main() {
	go starServer("127.0.0.1:8897")
	go starServer("127.0.0.1:8898")
	go starServer("127.0.0.1:8899")

	a := make(chan bool, 1)
	<-a
}

func starServer(port string) {
	tcpAddr, err := net.ResolveTCPAddr("tcp4", port)
	fmt.Println(tcpAddr)
	checkError(err)

	listener, err := net.ListenTCP("tcp", tcpAddr)
	checkError(err)

	//注册zk节点q
	conn, err := example.GetConnect()
	if err != nil {
		fmt.Printf(" connect zk error: %s ", err)
	}
	defer conn.Close()
	err = example.RegistServer(conn, port)
	if err != nil {
		fmt.Printf(" regist node error: %s ", err)
	}

	for {
		conn, err := listener.Accept()
		if err != nil {
			fmt.Fprintf(os.Stderr, "Error: %s", err)
			continue
		}
		go handleCient(conn, port)
	}

	fmt.Println("aaaaaa")
}

func handleCient(conn net.Conn, port string) {
	defer conn.Close()

	daytime := time.Now().String()
	conn.Write([]byte(port + ": " + daytime))
}
```
client: 客户端操作

``` golang

func main() {
	for i := 0; i < 100; i++ {
		startClient()

		time.Sleep(1 * time.Second)
	}
}

func startClient() {
	// service := "127.0.0.1:8899"
	//获取地址
	serverHost, err := getServerHost()
	if err != nil {
		fmt.Printf("get server host fail: %s \n", err)
		return
	}

	fmt.Println("connect host: " + serverHost)
	tcpAddr, err := net.ResolveTCPAddr("tcp4", serverHost)
	checkError(err)
	conn, err := net.DialTCP("tcp", nil, tcpAddr)
	checkError(err)
	defer conn.Close()

	_, err = conn.Write([]byte("timestamp"))
	checkError(err)

	result, err := ioutil.ReadAll(conn)
	checkError(err)
	fmt.Println(string(result))

	return
}

func getServerHost() (host string, err error) {
	conn, err := example.GetConnect()
	if err != nil {
		fmt.Printf(" connect zk error: %s \n ", err)
		return
	}
	defer conn.Close()
	serverList, err := example.GetServerList(conn)
	if err != nil {
		fmt.Printf(" get server list error: %s \n", err)
		return
	}

	count := len(serverList)
	if count == 0 {
		err = errors.New("server list is empty \n")
		return
	}

	//随机选中一个返回
	r := rand.New(rand.NewSource(time.Now().UnixNano()))
	host = serverList[r.Intn(3)]
	return
}
```
我们先启动server，可以看到有三个节点注册到zk：    
![](https://raw.githubusercontent.com/NotBadPad/blog/master/pic/20160709-1.png)    
再启动client，可以看到每次client都会随机连接到一个节点进行通信：    
![](https://raw.githubusercontent.com/NotBadPad/blog/master/pic/20160709-2.png)    
至此，我们的分布式server就实现了，够简单吧。当然，实际项目中这样的实现是远远不够的，有兴趣的可以研究下一些开源的框架。    
相关代码：[github](https://github.com/NotBadPad/go-learn/tree/master/zk) 
