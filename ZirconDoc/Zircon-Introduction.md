
#Zircon#

`Zircon`是支持`Fuchsia OS`的核心平台，`Zircon`是由微内核（源码位于`kernel`目录），一小组用户空间服务集，驱动和库（源码位于`system`目录）组成，这些服务与库是系统引导，与硬件交互，加载用户空间进程，并执行这些进程等所必须的内容。Fuchsia在此基础上构建了一个相对更大的OS。

Zircon的Git库位于[https://fuchsia.googlesource.com/zircon](https://fuchsia.googlesource.com/zircon)，在Github上有一个只读库[ https://github.com/fuchsia-mirror/zircon](https://github.com/fuchsia-mirror/zircon)。

`Zircon`内核提供了管理进程，线程，虚拟内存，进程间通信，等待对象变化和锁（通过futexes提供）等系统调用。当前有一些临时的系统调用用于早期的辅助工作，在未来一旦长期的系统调用`API/ABI`接口完成，这些调用会去掉。预计会有100多个系统调用。`Zircon`的系统调用通常都是非阻塞的，但是像`wait_one`，`wait_many`，`port_wait`和线程睡眠等这些系统调用都是例外。

这篇并不是Zircon文档的完整索引，但是可以按照如下的顺序了解一下Zircon。

* [开始](GettingStarted.md)
* [贡献补丁]()
* [概念概览]()
* [内核对象]()
* [进程对象]()
* [线程对象]()
* [句柄]()
* [系统调用]()
* [驱动开发包（DDK）]()
* [测试]()
* [Hacking笔记]()
* [内存使用分析工具]()
* [与LK的关系]()
* [基准测试]()


By Andy @2018-06-24 10:25:23


