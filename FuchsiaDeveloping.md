#开发

Fuchsia源码库中文档关于开发的部分文档的学习笔记。

###开始###

这一部分主要记录一下获取源码，编译和运行Fuchsia系统。`Pink + Purple == Fuchsia`，一个新的操作系统。这节内容包含了开始Fuchsia的所有东西。

> 注意: Fuchsia源码包含了Zircon（支持Fuchsia的核心平台）。Fuchsia的编译过程也会编译Zircon。如果指向在Zircon上工作，则可以阅读Zircon的开始文档。




**先决条件**

准备编译环境。

`Ubuntu`上执行如下命令下载编译使用的工具。

```
sudo apt-get install texinfo libglib2.0-dev liblz4-tool autoconf libtool libsdl-dev build-essential golang git curl unzip
```

在MacOS上则需要安装Xcode命令行工具`xcode-select --install`，除了Xcode命令行工具，还需要安装最新版本的完整的Xcode，从`https://developer.apple.com/xcode/`上下载完整版本的Xcode。此外要安装编译用的工具：

使用Homebrew可以按照如下的过程：

```
# Install Homebrew
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# Install packages
brew install wget pkg-config glib autoconf automake libtool golang
```

使用MacPorts可以按照如下过程安装：

```
# Install MacPorts
# See https://guide.macports.org/chunked/installing.macports.html
port install autoconf automake libtool libpixman pkgconfig glib2
```

**编译Fuchsia**

获取源码可以参考下一节获取源码笔记。


###下载源码###

Fuchsia使用`jiri`工具管理git库，这个工具通过一个配置文件manifest来管理一组库。

**创建一个新的检出**

开始之前要求已经安装了`Go 1.6`或更新版本，安装了Git，并添加到执行路径。



