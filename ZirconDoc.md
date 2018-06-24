
#Zircon#

Fuchsia系统的内核是`Zircon`，它的文档可以参考[Zircon Doc](https://fuchsia.googlesource.com/zircon/+/HEAD/README.md)。`Zircon`是Fuchsia OS的核心平台，Zircon是由微内核（源码位于`kernel`目录）组成，同时作为用户空间服务，驱动，系统引导所需库(源码位于`system`目录)的一个小的集合。它与硬件交互，加载用户空间进程，并执行这些进程。Fuchsia在此基础上构建了一个相对更大的OS。

Zircon的Git库位于[https://fuchsia.googlesource.com/zircon](https://fuchsia.googlesource.com/zircon)，在Github上有一个只读库[ https://github.com/fuchsia-mirror/zircon]( https://github.com/fuchsia-mirror/zircon)。

`Zircon`内核提供了管理进程，线程，虚拟内存，进程间通信，等待对象变化和锁（通过futexes提供）等系统调用。当前有一些临时的系统调用用于早期的培养工作，在未来一旦长期的系统调用API/ABI接口完成，这些调用会去掉。预计会有100多个系统调用。Zircon的系统调用通常都是非阻塞的，但是像`wait_one`，`wait_many`，`port_wait`和线程睡眠等这些系统调用都是例外。

Zircon是基于嵌入式内核LK开发，LK是一个SMP感知的内核，设计用于小系统中。LK的Github源码地址为[https://github.com/littlekernel/lk](https://github.com/littlekernel/lk)

这篇并不是Zircon文档的完整索引，但是可以按照如下的顺序了解一下Zircon。

* 开始
* 贡献补丁
* 概念概览
* 内核对象
* 进程对象
* 线程对象
* 句柄
* 系统调用
* 驱动开发包（DDK）
* 测试
* Hacking笔记
* 内存使用分析工具
* 与LK的关系
* 基准测试

###开始###

检出Zircon源码。

> 注意：Fuchsia源码包括了Zircon，参考Fuchsia的`Gettting Started`文档。本文档只针对Zircon。

Zircon的Git库位于[https://fuchsia.googlesource.com/zircon](https://fuchsia.googlesource.com/zircon)。克隆代码库可以使用如下的命令，这里假设已经设置了`$SRC`环境变量

```
git clone https://fuchsia.googlesource.com/zircon $SRC/zircon
```

本篇文档假设Zircon检出到`$SRC\zircon`目录，下面会在该目录下构建工具链。Qemu等都遵守这个约定。

**准备编译环境**

在Ubuntu上应该获取如下的一些工具。

```
sudo apt-get install texinfo libglib2.0-dev autoconf libtool bison libsdl-dev build-essential
```

在macOS上要安装Xcode命令行工具`xcode-select --install`，以及一些其他的编译工具。使用Homebrew可以使用如下方法下载。

```
brew install wget pkg-config glib autoconf automake libtool
```

使用MacPorts也可以，如下命令安装依赖工具。

```
port install autoconf automake libtool libpixman pkgconfig glib2
```

安装工具链，如果在Linux或MacOS上开发，有一些预编译工具链二进制可用。只需要在Zircon工作目录执行如下的脚本。

```
./scripts/download-prebuilt
```

如果想要自己编译工具链，可以按照文档后面的文档进行。

**编译Zircon**

编译结果会在`$SRC/zircon/build-{arm64,X64}`。如下例子中所用的变量`$BUILDDIR`指向编译输出目录。

```
$ cd $SRC/zircon

#for aarch64
make -j32 arm64

#for X64
make -j32 x64
```

使用`Clang`作为编译Zircon的目标工具链，在调用Make时设置`USE_CLANG=true`变量。

```
cd $SRC/zircon

# for aarch64
make -j32 USE_CLANG=true arm64

# for x64
make -j32 USE_CLANG=true x64
```

为所有的目标平台编译Zircon。

```
# The -r enables release builds as well
./scripts/buildall -r
```

请在提交前为所有目标平台进行编译，确保编译结果在所有平台上运行良好。

**QEMU**

如果在真实的硬件上测试则可以跳过这一部分，但是模拟器是手边非常方便的快速本地测试和通常非常值得使用的方式。参考[Qemu](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/qemu.md)文档中关于编译和在Qemu中运行Zircon。

**编译工具链（可选）**

如果预编译工具链二进制不适合你用，可以从这些工具链的上游代码中编译自己的工具。

在工程中使用GCC `binutils 2.30`版本，`GCC 8.2`版本，使用配置`--target=x86_64-elf`或`--target=aarch64-elf`和`--enable=initfini-array`。

余下一部分没有翻译，简单看了一下。如果之后要补充完整，可以再翻译一下。


###贡献补丁###

当前这个时间点，Zircon正处于大量，动态开发中，目前还没有从新的贡献者那里寻求重大的改变或新的功能的计划。当然了，如果你想要做一些贡献，小的bugfixed还是很欢迎的。

下面是一些为Zircon开发补丁的的指导。这个列表并不完整，在后面随着项目开发会不断完善。

内容包括如下：

* 过程
* 风格
* 文档

一部分没有翻译，需要补充在进行翻译。

###概念概览###

内核管理几个不同类型的对象。这些对象通过系统调用可以直接访问，它们是实现了分发接口的`C++`类。这些对象的实现位于`kernel\object`目录下。它们许多自包更高层次的对象；一些包装了低层次的lk原语。

**系统调用**

用户空间的代码与内核对象通过系统调用交互，几乎所有系统调用都是通过Handles进行。在用户空间，Handle是由32位的整数（类型是`zx_handler_t`）表示。当系统调用执行时，内核检查句柄参数，这个句柄参数指向真正的句柄，它们保存在调用进程的句柄表中。内核会进一步检查句柄是正确的类型（传递线程句柄到要求事件句柄的系统调用会发生错误），以及句柄是否具有要求操作的请求权限。

系统调用从标准视角看会进入三种宽泛的类别中：

1. 没有限制的调用，这种类型只有非常少的几个，例如`zx_clock_get()`和`zx_nanosleep()`可以由任何线程调用。
2. 使用Handles作为第一个参数的调用，句柄表示动作发生的对象，这种是系统调用中的大部分。例如`zx_channel_write()`和`zx_port_queue()`。
3. 创建新对象的调用，这种调用不需要Handle，例如`zx_event_create()`和`zx_chennel_create()`。这些系统调用的访问（限制于这些对象）是由包含在调用进程中的任务控制。

系统调用是由`libzircon.so`文件提供，它是一个Zircon内核提供给用户空间的虚拟共享库，更好的称呼是虚拟动态共享对象或`vDSO`。它们`zx_noun_verb()`或`zx_noun_verb_direct-object()`形式的`C ELF ABI`函数。

系统调用是在syscalls.abigen文件中定义，被abigen工具处理成包含文件，连接libzircon和内核的libsyscalls中的代码。

**句柄和权限**

对象可能有多个句柄（在一个或多个进程中）指向它们。

对于大部分的对象来说，如果最后一个指向它们的打开句柄关闭时，对象会或者被销毁，或者设置终止状态。句柄可以通过写入Channel(比如`zx_channel_write()`系统调用)从一个进程移到另外一个进程中，或者通过`zx_process_start()`系统调用将句柄作为参数传递给新进程的第一个线程。

在句柄或它指向的对象上可以执行的操作是由与句柄相关的权限控制，两个指向同一个对象的句柄可能有不同的权限。

`zx_handle_dumplicate()`和`zx_handle_replace()`两个系统调用可能用于获取指向相同对象的额外句柄，用于传参，在复制句柄时可以减少句柄权限。如果句柄是最后一个指向对象的，系统调用`zx_handle_close`可以关闭句柄，释放它指向的对象。

**内核对象ID**

内核中的每一个对象都有一个内核对象ID或简写的`koid`。它是一个64位的无符号证书，用于标识对象，在系统运行生命周期内它是一个唯一值。这意味着koid不会重用。

有两个特殊的koid值：

`ZX_KOID_INVALID`有零值，用做`null`标记。`ZX_KOID_KERNEL`，每个系统只有一个内核，这个值用于内核自己的koid。

**执行代码：任务，进程和线程**

`Threads`代表地址空间内的执行线程（CPU寄存器，栈等），它包含在进程内。进程包含在任务内，它定义了各种资源限制。任务可以有父任务，一直向上可以追溯到根任务，根任务是由内核在引导时创建，传递给[userboot](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/userboot.md)，开始执行的第一个用户空间进程。

没有任务句柄，在一个进程内的线程不可能创建另外一个进程或任务。

[进程加载](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/program_loading.md)是内核层之上的用户空间的工具和协议提供。可以参考`process_create`，`process_start`，`thread_create`和`thread_start`。

**消息传递：Socket和Channels**

套接字和信道都是IPC对象，它们都是双向和双端的。创建一个套接字或信道会返回两个句柄，每一个都指向对象的一端。

套接字是流式的，数据写入或读取都是以一个或多个字节形式。简短的写入（如果套接字的缓存满了）或简短的读取（如果读取大于缓存大小的数据时）都是可能完成的。

信道是数据报形式的，具有最多64K的消息大小（可能会修改，貌似会变得更小），以及最多1024个句柄附加到消息（也可能会改变，貌似会变小）。他们不支持短读或写，即消息或者满足或者不满足。

当句柄写入信道时，它们在发送进程中会被移除。当带有句柄的消息从信道读取后，句柄会被加入到接收进程中。在这两个事件之间，句柄持续存在（确保它们指向的对象持续存在），除非将要被写入到的进程中信道端被关闭，这样的话，这些将要写入到这个进程的消息会被丢弃，消息包含的句柄也会被关闭。

参考` channel_create`，`channel_read`，`channel_write`，`channel_call`，`socket_create`，`socket_read`，`and socket_write`。

**对象和信号**

对象可以有多达32个信号（由`zx_signals`类型和`ZX_SIGNAL`宏定义表示），信号代表了对象当前状态的一点信息。信道和套接字可能是`READABLE`或`WRITABLE`状态；线程或进程可能是`TERMINATED`状态等等。

线程可能等待一个或多个对象上的信号变为活动。

参考[信号](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/signals.md)一节获取更多信息。

**等待：等待一个，等多个和端口**

线程可以使用`zx_object_wait_one()`等待一个句柄上的信号变为可用或者`zx_object_wait_many()`等待多个句柄上的多个信号变为可用。两个调用都允许超时，如果超时则马上返回，即使信号处于挂起状态。

如果线程要等待大量的句柄，那么使用端口会更加有效率，端口也是一个对象，其他的对象可以绑定到该对象，当信号在它们上面变为可用，那么端口会接收一个数据包，包含了挂起信号的信息。

**事件和事件对**

事件是一个简单对象，除了它自己的活动信号集合没有其他的状态。

事件对是成对的事件，它们会相互通知。事件对一个比较有用的属性是，当一个事件对的一方关闭（所有指向其中一个事件的句柄）时，那么`PEER_CLOSED`信号会在另外一个事件上变活动状态。

参考`event_create`和`eventpaire_ceate`。

**共享内存：虚拟内存对象（VMOs）**

虚拟内存对象表示一组内存物理页面，或者潜在页面（即马上会被按需创建或延迟填充的页面）。

它们可以映射进进程的地址空间，使用`zx_vmar_map()`进行映射，使用`zx_vmar_unmap()`进行反映射。映射页面的权限可以使用`zx_vmar_protect()`进行调整（读写执行等）。

VMOs也可以使用`zx_vmo_read()`和`zx_vmo_write()`系统调用进行读出和写入。因此映射进地址空间的代价可以通过一键操作如"创建一个VMO，吸入数据集，将它交由另外一个进程使用"来消除。

**地址空间管理**

虚拟内存地址区（VMARs）提供了管理进程地址空间的抽象。在进程创建时，指向根VMAR的句柄就会传递给进程创建器。指向VMAR的句柄跨过整个地址空间。地址空间可以通过`zx_vmar_map()`和`zx_vmar_allocate()`接口进行分割。`zx_vmar_allocate()`可以用于生成新的VMARs（叫做子区或孩子），它们用于组织起一部分地址空间。

参考`vmar_map`, `vmar_allocate`, `vmar_protect`, `vmar_unmap`, `vmar_destroy`等系统调用。

**Futexes**

Futexes是内核原语，用于用户空间原子操作，实现有效的同步原语。例如Mutexes，它使得一个系统调用可以用于竞争环境。通常它们仅用于实现标准库，Zircon的`libc`和`libc++`为mutexes，条件变量等提供了`C11`，`C++`和pthread API，它们都是用Futexes实现。

###内核对象###

Zircon是一个基于对象的内核。用户模式代码几乎唯一可以通过对象句柄与OS资源交互。句柄可以认为是作为特定OS子系统范围内特定资源的一个活动会话。

Zircon动态管理如下的资源：

* 处理器时间
* 内存和地址空间
* 设备I/O内存
* 中断
* 信号和等待

**应用程序使用内核对象**

IPC包含了[信道](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/channel.md)，[套接字](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/socket.md)，[FIFO](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/fifo.md)。

任务包含了[进程](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/process.md)，[线程](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/thread.md)，[作业](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/job.md)，[任务](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/task.md)。

信号包括[事件](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/event.md)，[事件对](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/eventpair.md)，[Futex](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/futex.md)。

内存和地址空间包括[虚拟内存对象](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/vm_object.md)，[虚拟内存地址区](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/vm_address_region.md)，[总线转换初始器](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/bus_transaction_initiator.md)。

等待对象只有[Port](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/port.md)。

**驱动使用的内核对象**

驱动使用的内核对象包括[中断](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/interrupts.md)，[资源](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/resource.md)，[日志](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/objects/log.md)，

**内核对象和LK**

一些内核对象封装了一个或多个LK级别的概念，例如线程对象封装了一个`thread_t`，但是Channel对象则没有封装任何LK层的对象。

**内核对象生命周期**

内核对象是引用计数的，大部分内核对象都是在系统调用中创建，在句柄持有它且`refcount=1`时保持存活，句柄绑定了对象，会作为系统调用的返回值。句柄对象只要是连接到句柄表就一直保持存活。从句柄表分离的句柄会关闭句柄对象（通过`sys_close()`），减少内核对象的引用计数。通常，当最后一个句柄关闭时，内核对象的引用计数会到0，这会造成析构函数运行。

引用计数在新的句柄（指向对象）创建和直接指针引用（通过一些内核代码）时就会增加；因此内核对象的生命周期或许比床架它的进程还要长。

**Dispatchers（分发器）**

内核对象是作为一个`C++`类来实现，它们都是从`Dispatcher`类派生，并且会覆盖它所实现的方法。因此，例如线程对象的代码可以在`ThreadDispatcher`中找到。有大量的代码仅仅以共同的方式关注内核对象，这种情况下可能会看到名字是`fbl::RefPtr<Dispatcher>`。

**内核对象安全**

通常来讲，内核对象没有固有的安全概念，通常也并不做授权检查；安全权限都是由每一个句柄保存。同一个进程中可以有两个不同权限的句柄指向同一个对象。

参考句柄一节内容。

###进程对象###

待翻译

###线程对象###

待翻译

###句柄###

待翻译

###系统调用###

待翻译

###驱动开发包（DDK）###

待翻译

###测试###

待翻译

###Hacking笔记###

待翻译

###内存使用分析工具###

待翻译

###与LK的关系###

Zircon是作为LK的一个分支而诞生，甚至到今天许多内部的构建还是基于LK实现，但是其上的一层是全新的。例如Zircon有进程的概念，但是LK没有。然后Zircon进程是由LK层的组件构成，例如LK的`thread_t`。

LK是一个用于小系统的内核，比如用于嵌入式应用。它是一些商业提供的FreeRTOS或ThreadX内核的良好的替代。这些系统通常都有非常有限量的RAM，固定的一组外围设备和一组绑定的任务。

但是Zircon目标在于现代手机和现代的个人电脑，又很快的处理器，大量的RAM以及任意的外围设备完成开放中断计算。

更详细的一些比较，如下几个比较明显：

* LK可以运行在32位系统上，而Zircon仅用于64位
* Zircon有第一类的用户模式支持，LK没有用户模式
* Zircon有基于兼容性设计的安全模型，而LK所有的代码都是受信的。

随着时间变化，底层构建已经发生变化以满足新的要求，更好适应系统的其他部分。

###基准测试###

待翻译

By Andy @2018-06-24 10:25:23


