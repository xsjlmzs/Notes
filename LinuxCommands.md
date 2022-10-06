# Linux Commands

``` shell
# 源码安装 以cmake为例
./autogen # 需要autoconf,automake,libtool;生成configure.sh
./bootstrap --prefix=/usr/local/cmake # 配置编译选项，生成Makefile文件,有时为configure.sh脚本
make # 执行编译
make check # 对Makefile所要构建的程序自检
make install # 执行安装
ldconfig # 创建出动态装入程序(ld.so)所需的连接和缓存文件,详见如下引用
```

**[通过ldconfig的工作看linux如何找到.so库文件](https://blog.csdn.net/winycg/article/details/80572735)**

```shell
$(command) # 命令替换，运行command，捕获其输出
${parameter} # 参数替换，替换为parameter的值
```

## [Where do executables look for shared objects at runtime?](https://unix.stackexchange.com/questions/22926/where-do-executables-look-for-shared-objects-at-runtime)

Linux 上常用的动态链接器使用缓存来查找其库。缓存存储在 中`/etc/ld.so.cache`，`ldconfig`命令通过`/etc/ld.so.conf`文件所包含的路径进行更新（通常是`/etc/ld.so.conf.d`中的文件）。它的内容可以通过运行列出`ldconfig -p`。

所以没有默认值`LD_LIBRARY_PATH`，默认库查找根本不需要它。如果`LD_LIBRARY_PATH`已定义，则首先使用它，但不会禁用其他查找（其中还包括一些默认目录）。详见[`ld.so(8)` manpage](http://man7.org/linux/man-pages/man8/ld.so.8.html)

> If a shared object dependency does not contain a slash, then it is searched for in the following order:
>
> - Using the directories specified in the `DT_RPATH` dynamic section attribute of the binary if present and `DT_RUNPATH` attribute does not exist. Use of `DT_RPATH` is deprecated.
> - Using the environment variable `LD_LIBRARY_PATH`, unless the executable is being run in secure-execution mode (see below), in which case it is ignored.
> - Using the directories specified in the `DT_RUNPATH` dynamic section attribute of the binary if present.
> - From the cache file `/etc/ld.so.cache`, which contains a compiled list of candidate shared objects previously found in the augmented library path. If, however, the binary was linked with the `-z nodeflib` linker option, shared objects in the default paths are skipped. Shared objects installed in hardware capability directories (see below) are preferred to other shared objects.
> - In the default path `/lib`, and then `/usr/lib`. (On some 64-bit architectures, the default paths for 64-bit shared objects are `/lib64`, and then `/usr/lib64`.) If the binary was linked with the `-z nodeflib` linker option, this step is skipped.

## [LD_LIBRARY_PATH vs LIBRARY_PATH](https://stackoverflow.com/questions/4250624/ld-library-path-vs-library-path)

gcc在编译时通过使用`LIBRARY_PATH`去搜索静态和共享库所在的目录。

程序在成功编译链接后使用`LD_LIBRARAY_PATH`目录去搜索包含所需的共享库。

> 如下所述，您的库可以是静态的或共享的。 如果它是静态的，那么代码将被复制到您的程序中，并且您无需在程序编译和链接后搜索库。 如果您的库是共享的，那么它需要动态链接到您的程序，这就是 `LD_LIBRARY_PATH` 发挥作用的时候。

> ld 本身不会在 `LIBRARY_PATH` 或 `LD_LIBRARY_PATH` 中查找库。 只有当 gcc 调用 ld 时才会使用 `LIBRARY_PATH`。

ld是静态链接器，ld.so是动态链接器。

动态链接器在程序运行时加载编译时被动态链接的库进入程序的地址空间。

`LIBRARY_PATH`是gcc定义和使用的环境变量，gcc链接时将`LIBRARY_PATH`作为-L参数调用ld

`LD_LIBRARY_PATH`是ld.so动态链接器定义和使用的环境变量，供程序运行时访问共享库提供目录信息。

### 静态库链接时搜索路径顺序：

1. ld会使用gcc命令中传递的参数-L；
2. gcc的环境变量LIBRARY_PATH；
3. 内定目录 /lib、/usr/lib和/usr/local/lib，这是当初compile gcc时写在程序内的。

### 动态链接时、执行时搜索路径顺序：

​	详见上述引用内容

## pkg-config

一个命令行工具，告诉 gcc 等编译器，程序依赖的库和头文件的位置0。

用法举例：``c++ my_program.cc my_proto.pb.cc `pkg-config --cflags --libs protobuf` ``

### 如何知道库的存放路径和头文件路径？

pkg-config 会从 /usr/lib/pkgconfig/ 下面找该库的 .pc 文件
或者从 PKG_CONFIG_PATH 环境变量中找该库的 .pc 文件路径

####       方式一：

某个库在安装完成后，会在其安装目录下的 lib/ 下生成 .pc 文件。把这个文件复制到 **/usr/lib/pkgconfig/** 下。然后用 pkg-config --libs libxxx , 查看是否生效。

#### 方式二：

把指定库的 .pc 文件路径，添加到 PKG_CONFIG_PATH 环境变量中

### CMake中使用pkg-config 

```cmake
find_package(PkgConfig REQUIRED) # 在Modules目录查找FindPkgConfig.cmake 
pkg_check_modules(PB REQUIRED IMPORTED_TARGET protobuf) # pkg-config查找protobuf模块
target_link_libraries(${PROJECT_NAME} PkgConfig::PB) # 添加pkg-config提供的信息
```

```shell
echo -e "\texecuting a PRE_BUILD command" # -e 激活转义字符
```

## 链接

```shell
sudo ln -s /usr/local/cmake/bin/cmake  cmake # 创建软链接，源文件到目标文件
```

### 头文件搜索路径

> gcc在编译时如何去寻找所需要的头文件：
>
> 1、先搜索-I指定的目录
>
> 2、然后找gcc的环境变量C_INCLUDE_PATH，CPLUS_INCLUDE_PATH，OBJC_INCLUDE_PATH可以通过设置这些环境变量来添加系统include的路径
>
> 3、最后搜索gcc的内定目录(编译时可以通过-nostdinc++选项屏蔽对内定目录搜索头文件)
>
> /usr/include
>
> /usr/local/include
>
> /usr/lib/gcc/x86_64-linux-gnu/4.8/include

```shell
groupadd dbgroup# 创建新的组 dbgroup
useradd -g dbgroup omm# 建立用户账号omm分配到组dbgroup
```

## Linux权限

Linux下权限的粒度有 拥有者u(user) 、群组ug(user group) 、其它组(o) 三种。每个文件都可以针对三个粒度，设置不同的rwx(读写执行)权限。一个文件只能归属于一个用户和组， 如果其它的用户想有这个文件的权限，则可以将该用户加入具备权限的群组，一个用户可以同时归属于多个组。对文件的权限类型包括读r、写w、执行x。 r=4，w=2，x=1 。

```shell
chmod a+r,ug+w,o-w a.conf b.xml # 设置文件 a.conf 与 b.xml 权限为拥有者与其所属同一个群组 可读写，其它组可读不可写
ls -ld # 查看文件夹权限
ls -l # 查看文件权限
# 权限前的d表示文件夹，-表示普通文件，l表示链接文件
```

## Linux命令记录

```shell
echo 'abc' > test.txt # >覆盖原有内容
echo '123' >> test.txt # >>追加原有内容
```



