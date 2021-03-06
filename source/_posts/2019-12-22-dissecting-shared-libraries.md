---
title: 【转】剖析共享程序库
toc: true
date: 2019-12-22 16:48:24
description: 原文地址：https://www.ibm.com/developerworks/cn/linux/l-shlibs.html，若有版权问题，请联系我删除。
tags:
- Linux
---

共享程序库是现代 UNIX® 系统中有效利用空间和资源的基础。SUSE 系统中的 C 程序库大约有 1.3 MB。为 /usr/bin 中每一个程序（我有 2,569 个）制作副本将占去几个 G 的空间。

当然这个数字有一些夸张 —— 静态链接程序只合并它们使用的那部分程序库。 尽管如此， `printf()` 的所有副本所占用的空间数量也会让系统 显得非常臃肿。

共享程序库不仅可以节省磁盘空间，而且还可以节省内存。内核可以在内存中保持某个共享程序库的一个惟一副本，并在多个应用程序间共享这个副本。所以，我们不但可以在磁盘上只有 `printf()` 的 一个副本，而且在内存中也只需要一个副本。这对性能有很大的影响。

在本文中，我们将讨论共享程序库所使用的底层技术，以及在共享程序库版本号帮助下预防兼容性难题的方法，过去，本机共享程序库实现也曾遇到过这些难题。首先来看一下共享程序库的工作原理。

## 共享程序库的工作原理

这个概念理解起来非常简单。拥有一个程序库；然后共享这个程序库。但是，当您的程序尝试调用 `printf()` 时，也就是说实际操作的时候，具体发生的事情却稍微有点复杂。

这个过程在静态链接系统中比在动态链接系统中更简单。在静态链接系统中，生成的代码会持有对某个 函数的引用。链接器使用加载该函数的真实地址去替换这个引用，以便生成的二进制代码 在适当的位置会有正确的地址。然后，在运行代码时，只需要跳转到相应的地址即可。对管理员来说， 这是一项简单的任务，因为它允许您对只在程序中的某个位置上实际引用的那些对象进行链接。

但是大部分共享程序库都是动态链接的。这具有一些更深层次的意义。其中一方面是，您不能事先预计 某个函数在调用时的确切地址！（以及静态链接的共享程序库模式，比如 BSD/OS 中的，但是它们 不在本文讨论范围之内。）

动态链接器可以为每个被链接的函数做相当多的工作，所以大部分链接器都是不积极的。只有在函数被调用时，它们才实际做一些工作。C 程序库中有一千多个外部可见的符号，有大约三千多个本地符号，因此这种 方法可以节省非常多的时间。

实现此奇妙功能的是一个称为 *过程链接表（Procedure Linkage Table）*（PLT）的数据块，它是程序中 的一个表，列出了程序所调用的每一个函数。当程序开始运行时，PLT 包含每个函数的代码，以便查询运行期链接器，从而获得已加载某个函数的地址。然后它会在表中填入这个条目并跳转到那个已加载函数。当每个函数被 调用时，它的 PLT 中的条目就会被简化为一个到那个已加载函数的直接跳转。

不过，重要的是，要注意到还有一个间接的额外层次 —— 可以通过跳转到某个表来解析每个函数调用。

## 兼容性不仅是为了关联

这意味着您最终要链接的程序库最好与调用它的代码相兼容。使用静态链接的可执行文件，可以在某种程度上 保证不会发生任何改变。如果使用动态链接，就得不到这样的保证。

当出现新版本的程序库时会怎样？特别是新版本改变了某个给定函数的调用次序时，又会怎样？

版本号可以解决这个问题 —— 共享的程序库将拥有一个版本号。当一个程序链接到某个程序库时，程序中 会存储一个它计划支持的版本号。如果更改程序库，那么版本号就会不匹配，程序也就不会被链接到较 新版本的程序库。

不过，动态链接的可能优势之一在于修正缺陷。如果可以修正程序库中的缺陷，而且不必重新编译上千个程序，就 可以利用这一修正功能，这将是非常令人愉快的。有时，需要链接到某个较新的版本。

不幸的是，这会导致在某些情况下，您希望链接到较新的版本，而在另外一些情况下，您宁愿坚持使用较老的版本。 不过，有一个解决方案 —— 使用两类版本号：

- 主版本号表明程序库版本之间的潜在不兼容性。
- 次要版本号表明只是修正了缺陷。

这样，在大部分情形下，加载具有相同主版本号和更高次要版本号的程序库是安全的；而加载主版本号更高的程序是不安全的行为。

为了让用户（和程序员）不必追踪程序库版本号和更新，系统提供了大量的符号链接。 通常，其模式是：

```
libexample.so
```

将是一个指向

```
libexample.so.N
```

的链接，其中 *N* 是在系统中可以找到的最高的 *主* 版本号。

对受支持的每一个主版本号而言，

```
libexample.so.N
```

将是一个指向

```
libexample.so.N.M
```

的链接，其中 *M* 是最高的 *次要* 版本号。

这样，如果为链接器指定了 `-lexample`，那么它会去寻找 `libexample.so`，这是一个符号链接，指向某个指向最新版本的符号链接。 另一方面，当加载某个现有程序时，它将尝试去加载 `libexample.so.N`， 其中 *N* 是它先前链接的版本。各得其所！

## 为了进行调试，首先必须知道如何编译

为了调试使用共享程序库的问题，对它们如何编译有更多一些了解会对您有所帮助。

在传统的静态程序库中，生成的代码通常封装在一个程序库文件中（其名称以 `.a` 结尾），然后传递给链接器。在动态程序库中，程序库文件的名称通常以 `.so` 结尾。 文件结构稍有不同。

常规的静态程序库的格式是 `ar` 工具（一个非常简单的存档程序，类似于 `tar`，但是更简单）所创建的那种格式。 相反，共享程序库通常以更复杂的文件格式存储。

在现代 Linux 系统中，这一格式通常是 ELF 二进制格式（可执行与可链接格式（Executable and Linkable Format））。 在 ELF 中，每个文件的组成包括：一个 ELF 头，随后是零或者一些段（segments），以及零或者一些区段（sections）。 *段* 中包含文件的运行时执行所需要的信息，而 *区段* 中包含用于链接和重定位的重要数据。 整个文件中的每个字节每次只能由一个区段使用，不过可以存在不被任何区段所包含的孤立字节。 通常，在 UNIX 可执行文件中，一个或多个区段会封装在一个段内。

ELF 格式中包含用于应用程序和程序库的规范。但程序库格式要复杂得多，不仅仅是对象模块的简单存档。

链接器将所有对符号的引用进行分类，标识出它们是在哪个程序库中找到的。将静态程序库的符号添加到最终的 可执行文件中；然后将共享程序库的符号放入 PLT 中，最后创建对 FLT 的引用。在完成这些任务之后，生成的可执行文件 会拥有一个列表，该列表列出了计划从运行期将加载的程序库中找出的那些符号。

在运行期间，应用程序将加载动态链接器。实际上，动态链接器本身使用与共享程序库相同种类的版本号。 例如，在 SUSE Linux 9.1 中， `/lib/ld-linux.so.2` 文件是一个指向 `/lib/ld-linux.so.2.3.3` 的符号链接。另一方面，寻找 `/lib/ld-linux.so.1` 的程序不会尝试使用新的版本。

然后动态链接器开始进行所有有趣的工作。它会查明某个程序先前链接到了哪些程序库（以及哪个版本）， 然后加载它们。加载程序库的步骤包括：

- 找到程序库（它可能在系统中若干个目录中的任意一个目录中）。
- 将程序库映射到程序的地址空间。
- 分配程序库可能需要的由零填充的内存块。
- 添加程序库的符号表。

调试这一过程可能会比较困难。您可能会遇到多种问题。例如，如果动态链接器不能找到某个给定的 程序库，那么它将停止加载程序。如果它找到了所有需要的程序库，但却无法找到某个符号，那么它也可能会因此而停止加载操作 （但是可能直到真正尝试去引用那个符号时才会发生这种情形） —— 这是一种很少见的情况，因为通常如果 不存在某个符号，那么在初始化链接的时候就会被警告。

## 修改动态链接器的搜索路径

当链接某个程序时，在运行期您可以指定另外的搜索路径。在 `gcc` 中，其 语法是 `-Wl,-R/path`。如果程序已经被链接，那么您也可以设置环境变量 `LD_LIBRARY_PATH` 来改变这一行为。通常只是在应用程序需要搜索的路径 不是系统级默认路径的一部分时才需要这样做，对大部分 Linux 系统来说，这种情况很少见。 理论上，Mozilla 用户可以发布某个使用这个路径设置所编译的二进制程序，但是他们 更倾向于发布包装器（wrapper）脚本，在启动可执行程序之前正确地设置程序库路径。

设置程序库路径可以为两个应用程序需要同一程序库的不兼容版本的这种罕见情况提供一个迂回解决方案。可以使用包装器脚本使某一应用程序在使用特殊版本程序库的目录中进行搜索。这称不上是一个 完美的解决方案，但是在某些情况下，这是您能采用的最佳方法。

如果出于不得已的原因需要为很多程序添加某个路径，那么也可以修改系统的默认搜索路径。通过 `/etc/ld.so.conf` 控制动态链接器，该文件包含默认搜索路径的列表。 对 `LD_LIBRARY_PATH` 中指定的任何路径的搜索都要先于 `ld.so.conf` 中列出的路径，所以用户可以覆盖这些设置。

大部分用户没有理由修改系统默认程序库搜索路径；通常环境变量更适用于修改搜索路径，比如 连接某个工具包中的程序库，或者使用某个程序库的较新版本的测试程序。

### 使用 `ldd`

`ldd` 是调试共享程序库问题的一个实用工具。其名称来自 *list dynamic dependencies*。这个程序会查看某个给定的可执行程序或者共享程序库，并 指出它需要加载哪些共享程序库以及要使用哪些版本。输出类似如下：

##### 清单 1. /bin/sh 的依赖

```
$ ldd /bin/sh    
		linux-gate.so.1 => (0xffffe000)    
		libreadline.so.4 => /lib/libreadline.so.4 (0x40036000)  		   
		libhistory.so.4 => /lib/libhistory.so.4 (0x40062000)
		libncurses.so.5 => /lib/libncurses.so.5 (0x40069000)
		libdl.so.2 => /lib/libdl.so.2 (0x400af000)
		libc.so.6 => /lib/tls/libc.so.6 (0x400b2000)
		/lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x40000000)
```

看到一个“简单的”的程序使用了这么多个程序库，可能会有些令人惊讶。或许是 `libhistory` 需要 `libncurses`。 为了查明真相，我们只需要运行另一个 `ldd` 命令：

##### 清单 2. libhistory 的依赖

```
$ ldd /lib/libhistory.so.4
		linux-gate.so.1 => (0xffffe000)
		libncurses.so.5 => /lib/libncurses.so.5 (0x40026000)
		libc.so.6 => /lib/tls/libc.so.6 (0x4006b000)
		/lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x80000000)
```

在某些情况下，可能需要为应用程序指定另外的程序库路径。例如，对 Mozilla 二进制程序尝试 运行 `ldd` 所得到输出的前几行如下所示：

##### 清单 3. 运行 dll 查找不在搜索路径中的 程序库的结果

```
$ ldd /opt/mozilla/lib/mozilla-bin
		linux-gate.so.1 => (0xffffe000)
		libmozjs.so => not found
		libplds4.so => not found
		libplc4.so => not found
		libnspr4.so => not found
		libpthread.so.0 => /lib/tls/libpthread.so.0 (0x40037000)
```

为什么找不到这些程序库？因为它们不在常见的程序库搜索路径中。实际上，它们在 `/opt/mozilla/lib` 中，所以，解决方案之一是将这个目录添加到 `LD_LIBRARY_PATH` 中。

另一个选项是将路径设置为 `.`，并在这个目录下运行 `ldd`， 尽管这样做更危险 —— 将当前目录添加到程序库路径中与将它添加到可执行程序路径中一样有着潜在的危险。

在这种情况下，将这些程序库所在的目录添加到系统级搜索路径中显然不是一个好办法。只有 Mozilla 需要这些程序库。

### 链接 Mozilla

说起 Mozilla，如果您觉得自己从未见过超过几行的程序库，那么在某种程度上，Mozilla 是一个更为典型的 大型应用程序。现在您可以明白为什么 Mozilla 的启动需要那么长时间了吧！

##### 清单 4. mozilla-bin 的依赖性

##### 清单 4. mozilla-bin 的依赖性

```
linux-gate.so.1 => (0xffffe000)
libmozjs.so => ./libmozjs.so (0x40018000)
libplds4.so => ./libplds4.so (0x40099000)
libplc4.so => ./libplc4.so (0x4009d000)
libnspr4.so => ./libnspr4.so (0x400a2000)
libpthread.so.0 => /lib/tls/libpthread.so.0 (0x400f5000)
libdl.so.2 => /lib/libdl.so.2 (0x40105000)
libgtk-x11-2.0.so.0 => /opt/gnome/lib/libgtk-x11-2.0.so.0 (0x40108000)
libgdk-x11-2.0.so.0 => /opt/gnome/lib/libgdk-x11-2.0.so.0 (0x40358000)
libatk-1.0.so.0 => /opt/gnome/lib/libatk-1.0.so.0 (0x403c5000)
libgdk_pixbuf-2.0.so.0 => /opt/gnome/lib/libgdk_pixbuf-2.0.so.0 (0x403df000)
libpangoxft-1.0.so.0 => /opt/gnome/lib/libpangoxft-1.0.so.0 (0x403f1000)
libpangox-1.0.so.0 => /opt/gnome/lib/libpangox-1.0.so.0 (0x40412000)
libpango-1.0.so.0 => /opt/gnome/lib/libpango-1.0.so.0 (0x4041f000)
libgobject-2.0.so.0 => /opt/gnome/lib/libgobject-2.0.so.0 (0x40451000)
libgmodule-2.0.so.0 => /opt/gnome/lib/libgmodule-2.0.so.0 (0x40487000)
libglib-2.0.so.0 => /opt/gnome/lib/libglib-2.0.so.0 (0x4048b000)
libm.so.6 => /lib/tls/libm.so.6 (0x404f7000)
libstdc++.so.5 => /usr/lib/libstdc++.so.5 (0x40519000)
libgcc_s.so.1 => /lib/libgcc_s.so.1 (0x405d5000)
libc.so.6 => /lib/tls/libc.so.6 (0x405dd000)
/lib/ld-linux.so.2 => /lib/ld-linux.so.2 (0x40000000)
libX11.so.6 => /usr/X11R6/lib/libX11.so.6 (0x406f3000)
libXrandr.so.2 => /usr/X11R6/lib/libXrandr.so.2 (0x407ef000)
libXi.so.6 => /usr/X11R6/lib/libXi.so.6 (0x407f3000)
libXext.so.6 => /usr/X11R6/lib/libXext.so.6 (0x407fb000)
libXft.so.2 => /usr/X11R6/lib/libXft.so.2 (0x4080a000)
libXrender.so.1 => /usr/X11R6/lib/libXrender.so.1 (0x4081e000)
libfontconfig.so.1 => /usr/lib/libfontconfig.so.1 (0x40826000)
libfreetype.so.6 => /usr/lib/libfreetype.so.6 (0x40850000)
libexpat.so.0 => /usr/lib/libexpat.so.0 (0x408b9000)
```

## 深入了解共享程序库

有兴趣深入了解 Linux 中的动态链接的用户有很多选择。GNU 编译器和链接器工具链（linker tool chain）文档都非常好， 虽然其内容是以 `info` 格式存储的，而且也没有在标准手册页中提及。

`ld.so` 的手册页包含有一个非常详尽的列表，列出了改变动态链接器行为的变量， 以及对过去曾经使用的不同版本的动态链接器的说明。

大部分 Linux 文档都假定所有共享程序库都是动态链接的，因为在 Linux 系统上，它们通常是这样的。 实现静态链接的共享程序库需要做的工作非常多，而且大部分用户不会因此获得任何好处，尽管支持这个特性的系统的性能会有显著改变。

如果您正在使用现成的预先包装好的系统，那么您可能不会遇到太多的共享程序库版本 —— 系统可能只附带它要链接的那些共享程序库版本。另一方面，如果您做过很多次更新和源代码构建，那么您可能最终得到 多个版本的共享程序库，因为老版本依然会被保留，“以防万一”。

像平时一样，如果想了解更多，那么就去亲自实践吧。记住，在某个系统上，几乎所有程序都会引用一些相同的共享程序库，所以，如果破坏了系统的某个核心共享程序库，那么您就得去求助系统恢复工具了。