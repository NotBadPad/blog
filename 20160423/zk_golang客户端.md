#go-zookeeper
最近打算写个小项目，会用到zookeeper，语言为了方便就选用golang了，于是就找了个go实现的zk客户单。    

* github: [https://github.com/samuel/go-zookeeper](https://github.com/samuel/go-zookeeper)  
* doc: [ http://godoc.org/github.com/samuel/go-zookeeper/zk]( http://godoc.org/github.com/samuel/go-zookeeper/zk)  

由于是doc是英文，看着有些不方便，于是就翻译了下。  

##import
```
import "github.com/samuel/go-zookeeper/zk"
```
##目录

##package文件
* [conn.go](https://github.com/samuel/go-zookeeper/blob/master/zk/conn.go)
* [constants.go](https://github.com/samuel/go-zookeeper/blob/master/zk/constants.go)
* [dnshostprovider.go](https://github.com/samuel/go-zookeeper/blob/master/zk/dnshostprovider.go)
* [flw.go](https://github.com/samuel/go-zookeeper/blob/master/zk/flw.go)
* [lock.go](https://github.com/samuel/go-zookeeper/blob/master/zk/lock.go)
* [server_help.go](https://github.com/samuel/go-zookeeper/blob/master/zk/server_help.go)
* [server_java.go](https://github.com/samuel/go-zookeeper/blob/master/zk/server_java.go)
* [structs.go](https://github.com/samuel/go-zookeeper/blob/master/zk/structs.go)
* [tracer.go](https://github.com/samuel/go-zookeeper/blob/master/zk/tracer.go)
* [util.go](https://github.com/samuel/go-zookeeper/blob/master/zk/util.go)

##常量
``` golang
const (
    EventNodeCreated         = EventType(1)
    EventNodeDeleted         = EventType(2)
    EventNodeDataChanged     = EventType(3)
    EventNodeChildrenChanged = EventType(4)

    EventSession     = EventType(-1)
    EventNotWatching = EventType(-2)
)
```
``` golang
const (
    StateUnknown           = State(-1)
    StateDisconnected      = State(0)
    StateConnecting        = State(1)
    StateAuthFailed        = State(4)
    StateConnectedReadOnly = State(5)
    StateSaslAuthenticated = State(6)
    StateExpired           = State(-112)

    StateConnected  = State(100)
    StateHasSession = State(101)
)
```  
```
const (
    FlagEphemeral = 1
    FlagSequence  = 2
)
```
```
const (
    PermRead = 1 << iota
    PermWrite
    PermCreate
    PermDelete
    PermAdmin
    PermAll = 0x1f
)
```
```
const (
    DefaultServerTickTime                 = 2000
    DefaultServerInitLimit                = 10
    DefaultServerSyncLimit                = 5
    DefaultServerAutoPurgeSnapRetainCount = 3
    DefaultPeerPort                       = 2888
    DefaultLeaderElectionPort             = 3888
)
```
```
const (
    DefaultPort = 2181
)
```
##变量
```
var (
    ErrConnectionClosed        = errors.New("zk: connection closed")
    ErrUnknown                 = errors.New("zk: unknown error")
    ErrAPIError                = errors.New("zk: api error")
    ErrNoNode                  = errors.New("zk: node does not exist")
    ErrNoAuth                  = errors.New("zk: not authenticated")
    ErrBadVersion              = errors.New("zk: version conflict")
    ErrNoChildrenForEphemerals = errors.New("zk: ephemeral nodes may not have children")
    ErrNodeExists              = errors.New("zk: node already exists")
    ErrNotEmpty                = errors.New("zk: node has children")
    ErrSessionExpired          = errors.New("zk: session has been expired by the server")
    ErrInvalidACL              = errors.New("zk: invalid ACL specified")
    ErrAuthFailed              = errors.New("zk: client authentication failed")
    ErrClosing                 = errors.New("zk: zookeeper is closing")
    ErrNothing                 = errors.New("zk: no server responsees to process")
    ErrSessionMoved            = errors.New("zk: session moved to another server, so operation is ignored")
)
```
```
var (
    // ErrDeadlock:当lock未被释放而又被lock时返回改err
    ErrDeadlock = errors.New("zk: trying to acquire a lock twice")
    // ErrNotLocked：当未获取lock而去释放时返aw回err
    ErrNotLocked = errors.New("zk: not locked")
)
```
```
var (
    ErrUnhandledFieldType = errors.New("zk: unhandled field type")
    ErrPtrExpected        = errors.New("zk: encode/decode expect a non-nil pointer to struct")
    ErrShortBuffer        = errors.New("zk: buffer too small")
)
```
```
var ErrInvalidPath = errors.New("zk: invalid path")
```
ErrInvalidPath表明设置的path是无效的的
```
var ErrNoServer = errors.New("zk: could not connect to a server")
```
ErrNoServer表明list中设置的server地址都连接失败
##方法
* AuthACL  
根据指定权限生成ACL（访问控制列表）  
```
func AuthACL(perms int32) []ACL  
```  
* DigestACL  
根据用户名密码生成ACL  
```
func DigestACL(perms int32, user, password string) []ACL
```   
* FLWCons
FLWCons是一个四字帮助方法，通常其从每个服务器获取响应信息。对于每个FLWSrvr，bool返回值表示响应是否有问题
```
func FLWCons(servers []string, timeout time.Duration) ([]*ServerClients, bool)
```
* FLWRuok
```
func FLWRuok(servers []string, timeout time.Duration) []bool
```
* 
