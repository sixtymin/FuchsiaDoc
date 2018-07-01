
#快速开始方法#

###检出Zircon源码###

> Fuchsia源码包含了Zircon，参考Fuchsia的[入门](https://fuchsia.googlesource.com/docs/+/master/getting_started.md)文档，如果只是对Zircon感兴趣可以按照如下文档入门。

Zircon的Git库位于[https://fuchsia.googlesource.com/zircon](https://fuchsia.googlesource.com/zircon)。克隆代码库可以使用如下的命令，这里假设已经设置了`$SRC`环境变量

```
git clone https://fuchsia.googlesource.com/zircon $SRC/zircon
```

本篇文档假设Zircon检出到`$SRC\zircon`目录，下面会在该目录下构建工具链。Qemu等工具都遵守这个约定。各种各样的`make`调用参数中，`-j32`选项用于并行`make`。如果这个并行设置超过你本地机器性能，可以尝试使用`-j16`或`-j8`。

###准备编译环境###

**Ubuntu**

在Ubuntu上应该获取如下的一些必要的工具。

```
sudo apt-get install texinfo libglib2.0-dev autoconf libtool bison libsdl-dev build-essential
```

**macOS**

在macOS上要安装Xcode命令行工具:

```
xcode-select --install
```

一些其他的必备编译工具。使用Homebrew可以使用如下方法下载。

```
brew install wget pkg-config glib autoconf automake libtool
```

使用MacPorts也可以，如下命令安装依赖工具。

```
port install autoconf automake libtool libpixman pkgconfig glib2
```

**安装工具链**

如果在Linux或MacOS上开发，有一些预编译工具链二进制可用。只需要在Zircon工作目录执行如下的脚本。

```
./scripts/download-prebuilt
```

如果想要自己编译工具链，可以按照文档后面的编译工具链一节说明进行。

###编译Zircon###

编译结果会在`$SRC/zircon/build-{arm64,X64}`目录。如下例子中所用的变量`$BUILDDIR`指向编译输出目录。

```
$ cd $SRC/zircon

#for aarch64
make -j32 arm64

#for X64
make -j32 x64
```

**使用Clang**

使用`Clang`作为编译Zircon的目标工具链，在调用Make时设置`USE_CLANG=true`变量。

```
cd $SRC/zircon

# for aarch64
make -j32 USE_CLANG=true arm64

# for x64
make -j32 USE_CLANG=true x64
```

**为所有目标编译Zircon**

```
# The -r enables release builds as well
./scripts/buildall -r
```

请在提交前为所有目标平台进行编译，确保编译结果在所有平台上运行良好。

###QEMU###

如果在真实的硬件上测试则可以跳过这一部分，但是模拟器是手边非常方便的快速本地测试和通常非常值得使用的方式。参考[Qemu](https://fuchsia.googlesource.com/zircon/+/HEAD/docs/qemu.md)文档中关于编译和在Qemu中运行Zircon。

###编译工具链（可选）###

如果预编译工具链二进制你不能使用，可以从这些工具链的代码中编译自己的工具。

**GCC工具链**

在工程中使用GCC `binutils 2.30`版本，`GCC 8.2`版本，使用配置`--target=x86_64-elf`或`--target=aarch64-elf`和`--enable=initfini-array`。



By Andy @2018-06-24 10:25:23





