title: "实现一个自己的 Docker 日志驱动"
date: 2016-11-04 18:38:08 +0800
update:
author: me
cover:
categories:
    - docker
tags:
    - log
    - hyper
    - golang
preview: 这本来是一个内部文档，不过似乎发布出来也不错，HyperContainer 支持原生的 Docker Logger，所以这篇日志讲得既是 Docker 的 Logger，也是 Hyper 的 Logger
draft: false

---

这本来是一个内部文档，不过似乎发布出来也不错，HyperContainer 支持原生的 Docker Logger，所以这篇日志讲得既是 Docker 的 Logger，也是 Hyper 的 Logger。

> 按，本文的参考代码主要位于 `"github.com/docker/docker/daemon/logger"` ，用于参考的 json-file logger 的代码位于 `"github.com/docker/docker/daemon/logger/jsonfilelog"` 。

## 关于 Docker Logger

Docker 的日志设计实际很简单，container 进程的标准输出和标准错误输出，就直接作为 container 的正常日志和错误日志。日志系统实际相当于两条管道（对于使用 tty 的 container，是一条），一端作为 `stdout` 和 `stderr` ， attach 到 container 上，接收 container 输出的信息，另一端写入日志系统。

这个日志系统本身是一个具有可扩展性的框架，不仅内建支持了一些日志输出方式，比如 `json-file`, `syslog`，也很容易扩展自己的日志系统，只需要实现几个接口即可。

## 日志消息

日志消息是日志中信息的基本单位，不论是记日志，还是读日志，基本单位都是 `Message` ，形状是这样的

```
// Message is datastructure that represents record from some container.
type Message struct {
	ContainerID string
	Line        []byte
	Source      string
	Timestamp   time.Time
}
```

字段的名字都很有描述性，基本不需要介绍， `Source` 字段是指 `stdout` 或 `stderr`，而消息的内容 (`Line`) 是不包含结尾换行的日志记录，如果写文件的话，需要日志驱动加上换行。

## 日志的记录

记录日志的东西就是 `Logger` ，它只需要实现这个接口：

```
type Logger interface {
	Log(*Message) error
	Name() string
	Close() error
}
```

从简单的说起， `Name()` 就是返回日志驱动的名字，比如 `json-file` 日志驱动就会返回 `"json-file"` ，除此之外毫无信息量。而 `Close()` 用来停止记录日志，在 Container 结束的时候调用来结束写入。

记录日志的主要函数是 `Log(*Message) error`, 当然这里也没有什么特别的，还是非常直观易懂，唯一需要注意的是，如果写入有并发的话，需要使用 `chan` 或加锁同步一下，避免写出奇怪的东西来。

## 日志的读取

对于 Docker Logger 来说，同一个 Logger 不紧要支持日志的写入，还要支持日志的读取，读取者要实现 `LogReader` 接口

```
type LogReader interface {
	// Read logs from underlying logging backend
	ReadLogs(ReadConfig) *LogWatcher
}
```

这个接口只有一个方法，但实际上要比写入复杂，`ReadLogs()` 函数是有一个 `ReadConfig` 类型的参数的：

```
type ReadConfig struct {
	Since  time.Time
	Tail   int
	Follow bool
}
```

这些是用户在读日志的时候可以指定的参数：日志开始时间 (`Since`)，读取日志的最后几行的行数 (`Tail`，为 0 则是输出所有日志)，以及是否 `Follow`，和 `tail` 命令一样，如果 `Follow` 为 `true`，那么，就需要一直等着程序的进一步输出，而不是显示完所有日志就完了。

`ReadLogs()` 并不直接返回日志内容，而是立刻返回一个 `LogWatcher`

```
type LogWatcher struct {
	// For sending log messages to a reader.
	Msg chan *Message
	// For sending error messages that occur while while reading logs.
	Err           chan error
	closeNotifier chan struct{}
}
```

通过这几个 `chan` 来给读者（一般是 logs API 的后端）异步返回日志信息、错误消息以及关闭提醒。这个 `LogWatcher` 有两个方法：

- `Close()` 关闭这个 `LogWatcher`
- `WatchClose()` ，返回一个 `<-chan`，用于监听 `LogWatcher` 事件，用法大致如下

  ```
select {
case <-watcher.WatchClose():
    //closed...
}
```

日志是允许多个 reader 同时读的，并且，当有 `Follow` 标记的时候，

## 日志工厂 (Creator)

Docker (或者说 `hyperd`) 会为每个 container 创建一个 `Logger`, 所以，日志驱动实际是通过一个 `Creator` 来创建日志的，每个日志驱动都要提供一个 `Creator` 函数

```
// Creator builds a logging driver instance with given context.
type Creator func(Context) (Logger, error)
```

这其中的 `Context` 是关于 Container 和 logger 配置的详细信息：

```
type Context struct {
	Config              map[string]string
	ContainerID         string
	ContainerName       string
	ContainerEntrypoint string
	ContainerArgs       []string
	ContainerImageID    string
	ContainerImageName  string
	ContainerCreated    time.Time
	ContainerEnv        []string
	ContainerLabels     map[string]string
	LogPath             string
}
```

由日志用户在创建 Logger 的时候填入，其中 `Config` 字段是日志专用的日志配置信息，`LogPath` 是日志存储路径，但并非所有的日志系统都需要一个路径，其他字段都是 Container 的信息，以备日志驱动可能会用到。

实现日志驱动的时候需要注意的一点是，日志 `Creator` 的返回类型是 `Logger` 接口，而实际上，返回的这个 `Logger` 也要实现 `LogReader`，在使用中，会把 Logger cast 成 *logger.LogReader。

此外，日志驱动还可以提供一个方法来验证配置参数

```
type LogOptValidator func(cfg map[string]string) error
```

日志驱动可以用这个方法来检验用户给的日志配置的合法性，如果有错误，返回一个 `error` 就使得 Container 无法使用这个日志驱动。

## 注册日志驱动

日志驱动实现了上述的类型和工厂方法后，要注册自己来成为系统中可用的日志驱动

```
func RegisterLogDriver(name string, c Creator) error

func RegisterLogOptValidator(name string, l LogOptValidator) error
```

比如，`json-file` 就在自己的 `init()` 里注册了自己的 Creator 和 Validator:

```
func init() {
	if err := logger.RegisterLogDriver(Name, New); err != nil {
		logrus.Fatal(err)
	}
	if err := logger.RegisterLogOptValidator(Name, ValidateLogOpt); err != nil {
		logrus.Fatal(err)
	}
}
```

## 小结

没啥可小结的了，就这些。
