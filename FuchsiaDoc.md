
#Fuchsia文档阅读笔记#

Fuchsia源码库文档地址[https://fuchsia.googlesource.com/docs/+/master/README.md](https://fuchsia.googlesource.com/docs/+/master/README.md)

`README.md`文档是Fuchsia文档的顶层入口，其目录包括如下的内容：

* 行为规范
* 术语，通常使用的术语的定义
* 开发，编译，执行和测试Fuchsia以及运行在Fuchsia上软件的一些说明
* 系统，关于Fuchsia如何工作的文档
* 仓库说明
* 贡献变化

在当前仓库的其他的文件都是Fuchsia系统级别的文档文章，每一个子工程都有自己的文档在它们的工程仓库中。上面这些链接都在系统级别库中和单独工程库中链接到单独的文档。

###行为准则###

这一块为Fuchsia的行为准则，用于规定开发人员之间的行为，要尊重他人，且有建设性。对于不满可以讲出来，发送邮件给社区管理员。

###Fuchsia的术语###

这篇文档的目的是提供在Fuchsia源码树中使用的一系列技术术语的定义。要添加新的定义要遵守如下的一些规则：

* 术语定义应该是现在在两句或三句话，给出术语准确的描述
* 当一个重大的技术术语需要作为描述的一部分使用时，考虑为它增加一个定义，并且给出连接到原始定义的链接。
* 定义应该是使用一系列指向更加详细文档和相关主题的链接进行补充

**术语**

**Agent**: 一个组件，它的生命周期不和任何Story绑定，在用户范围内是个单例，为其他的组件提供服务。代理人（Agent）可以被其他的组件或系统调用，用于响应触发器，例如推送通知。代理人可以给组件提供服务，发送和接收消息，给用户建议。

**AppMgr**: 应用程序管理器（AppMgr）负责加载组件以及管理组件运行的命名空间。它是有DevMgr在fuchsia中启动的第一进程。

**Armadillo**: 一个用户Shell的实现。

**Base Shell**: 平台保证的软件功能集合，它提供了一个基础的用于引导，第一次使用，授权，选择用户shell以及设备恢复的用户接口，

**Component**: 一个组件是一个执行和记录的单元。它包含了一组说明文件和相关的代码，这些来源于一个Fuchsia包（package）。组件运行在沙箱中，通过它的命名空间访问对象，通过它的导出目录发布对象。Modules和Agents是组件的例子。

**Channel**: 信道是Zircon提供的一组基础IPC原语。它是一个双向的，类数据包传传输通道，用于传输小的消息，包括Handles。

**DevHost**: 一个设备主（DevHost）是一个进程，它用于枚举，加载和管理设备驱动的生命周期，以及低层次系统任务（为引导文件系统提供文件系统服务器进程，启动AppMgr等等）。

**DDK**: 驱动开发包是一个文档，APIs，ABI等构建Zircon设备驱动的必要的组件。设备驱动被实现为ELF共享库，它有Zircon的设备管理器加载。

**Escher**: 图形库用于组成用户接口内容。它的设计来源于现代的实时和基于物理的渲染技术，因此我们期望它所渲染的大部分内容具有非真实的或风格化的质量，以适合用于用户界面。

**FAR**: 它是Fuchsia Archive Format的简写，它是Zircon和Fuchsia使用的文件容器。它用于替代Zircon的老式BootFS容器，在Fuchsia软件包中使用。

**fdio**: fdio是Zircon的`I/O`库，它提供了POSIX风格的`open`，`close`，`write`，`select`，`poll`等实现，相对于RemoteIO RPC协议。这些API在libc中是返回未支持的stubs，在链接时用libfdio的函数实现替代这些stubs。

**FIDL**: `Fuchsia Interface Definition Language`（FIDL）是一个语言用于定义信道（Channels）中协议定义。FIDL不会感知编程语言，并且已经绑定用于许多流行的语言，包括`C`，`C++`，`Dart`，`Go`和`Rust`。这个方法使得用各种语言编写的系统组件无缝交互。

**Flutter**: Flutter是一个函数反应式的用户接口框架，优化后用于Fuchsia，被许多系统组件使用。Flutter也运行在各种平台上，包括Android和iOS。Fuchsia自身不要求你使用任何特定语言或用户接口框架。

**FVM**: `Fuchsia Volume Manager`是一个分区管理器，提供动态分配块组，即分隔成虚拟块地址空间。FVM分区提供了一组接口使得文件系统在管理大的普通块设备一致性中可以与它交互。

**Garnet**: Garnet是Fuchsia代码基础的四层之一。可以参考Fuchsia的[层模型](https://fuchsia.googlesource.com/docs/+/master/development/source_code/layers.md)

**GN**: GN是一个元编译系统，它可以生成编译文件，这样Fuchsia可以使用Ninja进行编译。GN很快，具有丰富的工具管理和浏览依赖文件。GN文件通常位于整个库的任何工程中，即`BUILD.gn`。

**Handle**: Zircon内核的文件描述符，句柄用于用户空间进程指向内核对象。它们可以通过Channel传递给其他的进程。

**Hub**: hub是一个任务查看自身的入口。它使得工具访问区域或组件实例在运行时的详细结构化信息。例如它们的名字，作业和进程ID，公开的服务等。

**Jiri**: Jiri是一个工具用于多库开发，它用于检出Fuchsia代码库。它支持多个子命令，这是得开发者可能很轻松管理本地的检出代码。

**Job**: 待添加定义

**Ledger**: Ledger是Fuchsia的一个分布式存储系统。应用程序可以直接使用Ledger，或者通过Modular框架导出的状态同步原语使用它，Modular框架在内部是基于Ledger编写。

**LK**: `Little Kernel`（LK）是嵌入式内核，是Zircon内核的组成部分。LK是以微控制器为中心，缺乏MMU，用户空间，系统调用的支持，Zircon添加了这些功能。

**MaxWell**: 它是一组服务，用于导出周围环境和任务相关上下文，建议和利用机器智能的基础设施。

**Module**: 带有module元数据文件的组件，元数据主要描述了模块的数据兼容性和语义角色。Modules在运行时显示UI和参与进Stories。

**Mozart**: 显示子系统，包括显示，输入，文字排版器和GPU服务。

**Musl**: Fuchsia的标准C库（libc）是基于Musl Libc实现。

**Namespace**: 命名空间是一个文件，目录，套接字和其他命名对象的复合层级。

**Netstack**: 待定义

**Ninja**: Ninja是一个编译系统，用于执行Fuchsia的编译，它是一个小型的编译系统，带有强劲的编译速度。与其他的系统不同，Ninja文件不支持手动编写，应该有其他更加特色的系统生成，比如Fuchsia中的GN。

**Paver**: Zircon中的一个工具，用于安装分区映像到设备的内部存储中。

**Peridot**: Peridot是Fuchsia代码基础的四层之一。

**Realm**: 待定义

**RemoteIO**: RemoteIO是Zircon RPC协议，用于fdio和文件系统，设备驱动等之间。作为FIDL v2的一部分，它会成为FIDL接口（文件，目录，设备等）的一个集合，实现客户端或服务器更容易的互操作性，更加便捷的一部I/O。

**Service**: 一个服务是一个FIDL接口的实现，组件可以为它们的创建这提供一组服务，创建者可以直接使用或提供给其他的组件使用。

服务也可以从命名空间通过接口名字获得，这使得组件可以创建实现了接口的命名空间。长期运行的服务，比如Mozart局势典型的通过Namespace获取的，这使得许多客户端连接到同一个实现上。

**Story**: 一个用户面对的逻辑容器，封装了人的活动，满足其中一个或多个相关模块。Story允许用户按照他们找到自然顺序组织活动，不需要开发者提前考虑这些方式。

**Story Shell**: 系统响应故事的可视化展现，包括呈现组件，加上从每一个故事中读取的结构和状态信息。

**Topaz**: Topaz是Fuchsia代码中的四层之一。

**User Shell**: 用户指定和可替代软件功能集合，这个功能与设备联合工作，创建一个人们可以与模块交互的环境。

**VDSO**: VDSO是`Virtual Shared Library`的缩写，它是由Zircon内核提供，不会出现在文件系统或程序包中。它将Zircon系统调用API/ABI以ELF二进制形式提供给用户空间进程。在FuchsiaSDK和Zircon DDK，它以libzircon.so存在，用于提供一些代表VDSO的信息给链接器。

**VMAR**: 虚拟内存地址区是Zircon内核对象，控制这那块和如何将VMO映射到进程地址空间。

**VMO**: 虚拟内存对象是Zircon内核对象，代表了一组页面（或潜在页面），这些页面会被读取，写入和映射到进程地址空间，或通过Channel传递句柄与其他进程共享。

**Zedboot**: 待添加

**Zircon**: Zircon在Fuchsia内核中是微内核和最底层的用户空间组件（驱动运行时环境，核心驱动，libc等）。在传统的宏内核中，许多Zircon的用户空间组件会是内核自己的一部分。Zircon也是Fuchsia代码基的四层之一。

**ZX**: ZX是在Zircon C API/ABI（`x_channel_create()`，`zx_handle_t`，`ZX_EVENT_SIGNALED`等）和库（特别是libzx）中Zircon的一个缩写。


###开发###

这部分是所有和开发Fuchsia和Fuchsia执行软件相关的Fuchsia文档的顶层入口。这一部分包含如下几个主题：

* 开发者流程
* 语言：使用的各种语言介绍
* API： 设计Zircon系统接口的红界
* SDK： 关于开发Fuchsia SDK的信息
* 硬件： Fuchsia开发硬件目标介绍
* 约定：这一节覆盖了Fuchsia范围内的约定与最好的实践
* 杂项

在另外一篇文章`Fuchsia开发`中详细记录阅读的笔记吧。

此外，**系统**也在另外一部分笔记中详细记录；**仓库说明**是说明文档目录中文件作用，省略。

By Andy @2018-06-24 14:42:23