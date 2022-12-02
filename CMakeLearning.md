# CMake Learning

```cmake
cmake_minimum_required(VERSION 3.23.1) # 指定最小需要的cmake版本
project(codebase VERSION 1.0) # 指定项目名称codebase，版本1.0
set(CMAKE_CXX_STANDARD 11) # 设置CXX标准为11
set(CMAKE_CXX_STANDARD_REQUIRED True) # 设置CXX标准必须
add_executable(codebase ./src/main.cc) # 从./src/main.cc生成可执行文件codebase
```

**target_include_directories()**：指定**目标**包含的头文件路径。[官方文档](https://cmake.org/cmake/help/v3.15/command/target_include_directories.html?highlight=target_include_directories)

**target_link_libraries()**：指定**目标**链接的库。[官方文档](https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/v3.15/command/target_link_libraries.html%3Fhighlight%3Dtarget_link_libraries)

**target_compile_options()**：指定**目标**的编译选项。[官方文档](https://link.zhihu.com/?target=https%3A//cmake.org/cmake/help/v3.15/command/target_compile_options.html%23command%3Atarget_compile_options)

**目标** 由 *add_library()* 或 *add_executable()* 生成。

**PUBLIC**，**PRIVATE**，**INTERFACE** 的区别，[参考](https://zhuanlan.zhihu.com/p/82244559)。

> `PRIVATE` - 被添加到目标（库）的包含路径中。
> ` INTERFACE` - 没有被添加到目标（库）的包含路径中，而是添加到了链接这个库的其他目标（库或者可执行程序）包含路径中
>  `PUBLIC` - 既被添加到目标（库）的包含路径中，同时添加到了链接这个库的其他目标（库或者可执行程序）的包含路径中
>
> 如果一个library仅有header文件，只能`INTERFACE`来修饰

```cmake
install(TARGETS Proto DESTINATION lib) # 安装Proto库到lib目录
install(FILES ./include/txn.pb.h DESTINATION include) # 安装./include/txn.pb.h到include目录
install(TARGETS ${PROJECT_NAME} DESTINATION bin) # 安装可执行文件爱你到bin目录
```

```shell
cmake --build . # 编译构建，如果使用的构建器(builder generator)是Unix Makefiles 则等同于make
cmake --install . --prefix "/home/myuser/installdir" # 执行安装到指定目录下
```

`${VAL}`被解释为列表，包含**0或多个**参数; `"${VAL}"`被解释为**单个**字符串，一定是一个参数。**[example](https://stackoverflow.com/questions/13582282/cmake-difference-between-and?noredirect=1&lq=1)**

`message`命令可以接受可变数量的参数。它连接所接受的所有参数为**一个字符串**

## CMake内置变量

- CMAKE_SYSTEM  操作系统信息，如Linux-5.13.0-41-generic
- CMAKE_VERSION  CMake版本
- PROJECT_BINARY_DIR  运行 cmake 命令的目录，通常是 ${PROJECT_SOURCE_DIR}/build
- PROJECT_SOURCE_DIR  工程的根目录
- CMAKE_CXX_COMPILER  C++编译器，/usr/bin/g++
- CMAKE_CXX_FLAGS  编译 C++ 文件时的选项，默认为空
- CMAKE_CURRENT_SOURCE_DIR  当前处理的 CMakeLists.txt 所在的路径
- CMAKE_INCLUDE_PATH  添加头文件搜索路径，默认为空。配合 FIND_FILE() 以及 FIND_PATH 使用。
- CMAKE_LIBRARY_PATH  添加库文件搜索路径. 默认为空。配合 FIND_LIBRARY() 使用

## find_package()查找自定义安装目录的Protobuf

CMake需要查找FindProtobuf.cmake文件，其搜索路径为`CMAKE_MODULE_PATH`指示目录以及CMake的Modules目录。在Modules目录中找到FindProtobuf.cmake并执行。

```cmake
# Find the include directory
find_path(Protobuf_INCLUDE_DIR
    google/protobuf/service.h
    PATHS ${Protobuf_SRC_ROOT_FOLDER}/src
)
mark_as_advanced(Protobuf_INCLUDE_DIR)
```

在查找protobuf的头文件时，使用Protobuf_INCLUDE_DIR（默认为空），如果是protobuf目录是自定义安装的，`find_path()`方法未找到，则会报错`Could NOT find Protobuf (missing: Protobuf_LIBRARIES Protobuf_INCLUDE_DIR)`。`protoc`可执行文件因为设置在PATH中，所以可以找到。

`find_path()`查找规则：根据`CMAKE_SYSTEM_PREFIX_PATH`和`CMAKE_PREFIX_PATH`指定的prefix执行查找。

**总结**：通过设置`CMAKE_PREFIX_PATH`指定查找目录的prefix。

[reference](https://www.jianshu.com/p/5dc0b1bc5b62)

```cmake
list(APPEND CMAKE_PREFIX_PATH "/usr/local/protobuf")
find_package(Protobuf REQUIRED)
```

```shell
cmake -E # cmake支持的与平台无关的一些命令，如echo/copy等
```

## add_custom_command() & add_custom_target()

`add_custom_command()`添加自定义的命令，`OUTPUT`参数指定输出结果为一个/组文件，`COMMAND`参数指代要执行的自定义命令，如`OUTPUT`的文件存在，则不会再执行这些`COMMAND`。`DEPENDS`指代其运行的依赖。`VERBATIM`命令使得，所有参数都将为构建工具正确转义。

`add_custom_target()`添加自定义的目标，`NAME`指代target的名称，`ALL`参数指定该taget会在`cmake --build .`时一同被构建，否则需要使用`cmake --build . --target tar`的命令单独构建。`DEPENDS`参数指定其运行的依赖，一般为`add_custom_command()`的`OUTPUT`文件，这样，target在构建时如果未找到依赖的`OUTPUT`文件，则会去执行相关的`add_custom_command()`命令。

- [example](https://gist.github.com/baiwfg2/39881ba703e9c74e95366ed422641609) 
- [reference](https://www.bookstack.cn/read/CMake-Cookbook/content-chapter5-5.4-chinese.md)

## 使用FetchContent方法导入外部包

```cmake
include(FetchContent) # 引入FetchContent.camke包 搜索路径为CMAKE_MODULE_PATH
# 设置外部包的地址版本等信息
FetchContent_Declare(
  catch
  GIT_REPOSITORY https://github.com/catchorg/Catch2.git
  GIT_TAG        v2.13.6
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/ext/catch # 指定库下载地址
)
# 使用 FetchContent_Declare(MyName) 来从 URL、Git 仓库等地方获取数据或者是软件包。
# CMake 3.14+
FetchContent_MakeAvailable(catch) # 获取外部包内容
target_link_libraries(<project-name> catch)

# CMake 3.11+
FetchContent_GetProperties(catch)
# 使用 FetchContent_GetProperties(MyName) 来获取 MyName_* 等变量的值，这里的 MyName 是上一步获取的软件包的名字。
if(NOT catch_POPULATED)
  FetchContent_Populate(catch)
  add_subdirectory(${catch_SOURCE_DIR} ${catch_BINARY_DIR})
 endif()
# 检查 MyName_POPULATED 是否已经导出，否则使用 FetchContent_Populate(MyName) 来导出变量（如果这是一个软件包，则使用 add_subdirectory("${MyName_SOURCE_DIR}" "${MyName_BINARY_DIR}") ）

# 使用 FetchContent 模块下载的源码都在 build 下的 _deps 目录里
```

## 设置Debug模式

```cmake
SET(CMAKE_BUILD_TYPE "Debug")
SET(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
SET(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

## CMake 配置 Protobuf 

```cmake
protobuf_generate_cpp(PROTO_SRCS PROTO_HDRS foo.proto)
# 根据 foo.proto 生成头文件和源文件到相应的 build-tree 目录
# 要求 protobuf_generate_cpp 命令和生成 add_executable() 或 add_library() 的命令必须在同一个CMakeList中

```

