# Autotools

1. **autoscan** 扫描源代码目录生成autoscan.log和configure.scan文件。

```shell
# 假设include目录包含所有头文件 source目录包含所有源文件
autoscan [OPTION]... [SRCDIR] # 例如：autoscan source
```

2. 生成的configure.scan是一个模板文件，需要稍加修改一下。

```shell
AC_INIT([FULL-PACKAGE-NAME], [VERSION], [BUG-REPORT-ADDRESS]) # 修改1
# FULL-PACKAGE-NAME 为程序名称，VERSION为当前版本， BUG-REPORT-ADDRESS为bug汇报地址
# 在AC_INIT 宏下一行添加AM_INIT_AUTOMAKE
AM_INIT_AUTOMAKE
# 在AC_OUTPUT上一行添加AC_CONFIG_FILES宏，指定输出的文件。
AC_CONFIG_FILES([Makefile])
```

完成后将configure.scan重命名为configure.ac。

3. 生成**aclocal.m4**文件：

```shell
aclocal # Generate 'aclocal.m4' by scanning 'configure.ac' or 'configure.in'
```

4. 使用**autoconf**，生成configure文件：

```shell
autoconf 
```

5. 生成**config.h.in**文件

```shell
autoheader
```

5. 编写**Makefile.am**文件：

```shell
bin_PROGRAMS = sum
sum_SOURCES  = sum.c
sum_CPPFLAGS = -I ../include/
```

**reference list**

[在Linux操作系统下自动生成Makefile的方法](https://zhuanlan.zhihu.com/p/466365720)

[Autotools工具介绍](https://www.zhihu.com/question/22644913/answer/141475420)

