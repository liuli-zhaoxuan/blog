# Cmake学习笔记
## C++编译过程
Cmake学习笔记：C++的工程有如下几步：1预处理；2编译；3汇编；4链接

编译最小单元是cpp/c文件，头文件没有必要

main.cpp 函数可以找到当前路径下的头文件，别的路径下的头文件
## 基本的语法
~~~C
cmake_minimum_required(VERSION Cmake最低版本号)     Cmake版本
add_subdirectory(添加子文件夹)                      添加子文件夹
project(项目名称)                                   创建项目名
include_directories(包含路径)                       添加头文件包含路径(这个命令在modern cmake中已经不怎么用了)
target_link_libraries(target1 PRIVATE(PUBLIC/INTERFACE) target2)        链接库，减轻依赖
target_include_directories(target1 PRIVATE/PUBLIC 路径)                 链接头文件路径
add_executable(可执行文件名 源文件1.cpp 源文件2.cpp) 根据源文件，生成可执行文件

find_package(xxx REQUIRED)                       找库,大部分第三方库为了适配cmake都会配置类似(build文件夹下)：xxxConfig.cmake    eg.OpenCVConfig.cmake  大小写不能错

message("============${OpenCV_INCLUDE_DIRS}")
message("============${OpenCV_LIBS}")
~~~

## 怎么用命令行实现cmake的配置和编译
~~~cmake
mkdir build
cd ..
cd build
cmake ..                根据CMakeLists文件，创建Makefile文件；配置信息
cmake ../文件夹 -DCMAKE_VERBOSE_MAKEFILE = ON       打开编译细节（中间文件）

cmake --build .         编译Makefile文件（跨平台编译）
make                    编译Makefile(Linux下才能用)                 二选一

ldd 可执行文件          查看可执行文件需要的动态库
~~~

## 静态库、动态库
前者在编译阶段/链接阶段，把库中的内容加入到可执行文件中，所以最后生成的可执行文件 xx.exe 是可以单独执行的；后者是必须在程序运行的时候，才会加载这个库，所以必须和xx.exe放在一起（能找到库）。

Windows下静态库是lib后缀，Linux下是.a；动态库分别是dll，.so

对于Windows下，使用MSVC编译器，生成动态库需要在 .cpp文件函数前加入关键字 __deslspec(dllexport) ，会同时生成.lib和.dll文件

## 总结
通过B站视频学习[cmake本质](https://www.bilibili.com/video/BV1Mw411M761/?spm_id_from=333.788&vd_source=ea671ced0f3711791acabfaa64e4c3bf)，视频内容非常的生动。学完基本就能够cmake了，遇到问题就不会那么慌张，
OpenMVG的项目我反正是在Windows上和Linux上编译下来了。

## Cmake OpenMVG时遇到的困难总结
### Q1 vcpkg是什么东西？
vcpkg是怎么接触的呢？就是网上配OpenMVG的文章有很多，很多照搬都不行，有一个提到了vcpkg，我当时都没有成功就按照别人的配置配了。这里补一下知识。

vcpkg 是一个跨平台的开源C/C++包管理工具，由微软开发和维护。它旨在简化C/C++库的获取和管理过程，特别是在Windows、Linux和macOS平台上。使用vcpkg，可以轻松地下载、构建和集成各种C/C++库到你的项目中。
vcpkg专注于为C/C++开发者提供跨平台的库管理解决方案，而apt则是一个通用的包管理工具，主要用于Debian及其衍生系统上管理各种类型的软件包。

环境变量 VCPKG_DEFAULT_TRIPLET 指定了 vcpkg 的默认目标平台（triplet），它定义了库的架构、操作系统和编译器配置。对于你的例子，VCPKG_DEFAULT_TRIPLET 设置为 x64-windows，表示你希望 vcpkg 安装和构建的库适用于 64 位 Windows 平台。

最后编译成功了，发现其实VCPKG没有用上（汗流浃背），我发现基本是网络问题导致很多库下载不到位，挂个好梯子真的省很多事。

### Q2 项目编译完成后，有很多源码，我咋知道运行哪个？
设置单启动项目和当前选定内容的地方在"解决方案中"->"属性"->"启动项目"，这样就可以单独跑一个历程了。

### Q3 Linux下，第三方库通过apt下载后，cmake为啥能找到？
在C++中，编译器寻找库的顺序通常是通过指定的路径和环境变量来控制的。找一下几个地方：

１. 本地项目目录
２. 指定的包含路径和库路径。通过编译器选项（例如-I、-L和-l选项）指定的包含路径和库路径。
３. 环境变量,eg:'LIBRARY_PATH' 静态库和动态库的搜索路径
４. 系统默认路径,eg:/usr/include 和 /usr/local/include：头文件的默认搜索路径

apt显然就是将软件下载到了默认的路径下，这样相当于所有的C++工程都可以使用这些库。所以也体现了分开配置环境，Annaconda的作用，尤其是python，有时候只能用指定版本的函数包，所以分开配置可以防止污染导致奇怪的报错。
