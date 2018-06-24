
#Zircon调试技巧#

###Debug技巧###

**调试信息生成**

有几个make变量用于控制调试信息生成。

`GLOBAL_DEBUGFLAGS`指定了调试信息生成的级别，默认是`-g`。要获得骄傲少的调试信息，仅用于回溯，可以使用`-g1`选项。

`BOOTFS_DEBUG_MODULES`允许指定哪一个模块（apps，libs，tests）在引导镜像中包含它自己相关的调试信息。这个变量值是逗号分隔的模块短名字列表，这个列表的某一项通常格式为`parent_directory/module_directory`，即`ulib/launchpad`，`utest/debugger`。Make样式的模式字符（%）也可以使用，比如`ulib/%`，`utest/debugger`。

默认这个值是空的，意味着啥也没有。

**使用GDB调试Zircon应用程序**

要用GDB调试Zircon的应用程序要使用`gdbserver`。

**boot镜像中添加调试信息**

默认情况下引导镜像不包含调试信息，因为它添加调试信息就需要额外占用空间。当使用本地的调试器时，增加调试信息非常有用。注意这并不是用于跨平台调试，即调试器运行于一个单独的机器上。当你在Zircon上运行调试工具时，额外增加的调试信息才有用。

例如：

```
$ make -j10 x86 BOOTFS_DEBUG_MODULES=ulib/%,utest/debugger GLOBAL_DEBUGFLAGS=-g1
```

这个例子将共享库和调试器测试程序的调试信息放到引导镜像调试信息文件中。要减少调试信息的量，只是用于栈回溯的信息可以设置`GLOBAL_DEBUGFLAGS=-g1`变量。

**使用QEMU+GDB调试内核**

参考Qemu文档中`使用GDB调试内核`的中关于Qemu+GDB调试Zircon的内容。

###Qemu+GDB调试内核###



By Andy @2018-06-24 10:25:23


