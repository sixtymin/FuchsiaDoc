#开发

Fuchsia源码库中文档关于开发的部分文档的学习笔记。官方文档地址: [https://fuchsia.googlesource.com/docs/+/master/development/README.md](https://fuchsia.googlesource.com/docs/+/master/development/README.md)

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

如何编译源码，参考上一节的开始中的内容。

**创建一个新的检出**

`bootstrap`要求已经安装了`Go 1.6`或更新版本，安装了Git，并添加到执行路径。

首先，选择你想要编译的系统的层。如果不确定则选择`topaz`，它包含了更低的层。如果你想要改主意，可以使用`fx set-layer <layer>`来换为其他的层。

执行下面的命令

```
curl -s "https://fuchsia.googlesource.com/scripts/+/master/bootstrap?format=TEXT" | base64 --decode | bash -s topaz
```

这段脚本会为给定的layer开启一个开发环境，首先会创建一个目录`fuchsia`，然后为layer下载对应代码库，以及在编译layer中所需的相关的代码库。

**设置环境变量**

如果成功后，bootstrap脚本会打印一条消息，提醒你将`.jiri_root/bin`目录添加到`PATH`中。这会将`jiri`添加到`PATH`中，这一条设置强烈推荐，其他的Fuchsia工具链也默认有这个设置。

另外一个在`.jiri_root/bin`目录的工具是`fx`，它帮助配置，编译，运行和调试Fuchsia。参考`fx help`帮助内容中可用的命令。

同时也推荐执行脚本`scripts/fx-env.sh`，它定义了几个在文档中常用的环境变量，例如`$FUCHSIA_DIR`，提供有用的shell功能，例如`fd`命令修改目录。参考`script\fx-env.sh`中的注释获得更多详细信息。

**不修改PATH执行**

如果不喜欢处理环境变量，并且想让`jiri`可以依赖于当前目录工作，那么将`jiri`拷贝到PATH目录中，必须有写权限到你拷贝jiri的目录。如果没有写权限，`jiri`不能保持它自己始终最新。

```
cp .jiri_root/bin/jiri ~/bin
```

使用`fx`工具，可以在`~/bin`目录创建符号链接。或者也可以使用`scripts/fx`来执行程序，但是要确保jiri在可执行目录。

```
ln -s `pwd`/scripts/fx ~/bin
```

在每一个代码库的根目录或其他的目录中都有MAINTAINERS文件，这些文件列出了熟悉那些仓库代码和可以提供代码review的人的邮件地址。
