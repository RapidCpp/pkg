# CMake入门实战

# 什么是CMake？

也许你听说过GNU Make、QMake、MS-NMake、PMake、Makepp等Make工具，均遵循着不同的规范和标准，对应的Makefile格式也千差万别。如果软件想跨平台，必须保证能够在不同平台编译，就需要为每种标准写一次Makefile，这将无限抓狂！

CMake就是针对如上问题设计的工具：首先允许编写一种平台无关的CMakeList.txt文件来定制整个编译流程，然后根据目标用户平台进一步生成所需本地化Makefile和工程文件，如Unix的Makefile或者Windows的VisualStudio工程。一些使用CMake作为的项目架构系统的知名开源项目有：[VTK](http://www.vtk.org/)、[ITK](http://www.itk.org/)、[KDE](http://kde.org/)、[OpenCV](http://www.opencv.org.cn/)、[OSG](http://www.openscenegraph.org/)等。

在Linux平台下使用生成Makefile并编译的流程如下：
1. 编写CMake配置文件CMakeLists.txt
2. 执行命令 **cmake PATH** 或者 **ccmake PATH**生成Makefile，其中*PATH*是CMakeLists.txt所在的目录
3. 使用 **make** 命令进行编译

# 抛砖引玉：单个源文件

对于简单的项目，仅需要几行代码即可。例如，假设现在仅有一个main.cc，用于计算一个数的指数幂。

```
# /examples/camke/demo1/main.cc

#include <stdio.h>
#include <stdlib.h>

double power(double base, int exponent) {
    int result = base;
    if (exponent == 0) return 1;
    for (int i = 1; i < exponent; ++i>) {
        result = result * base;
    }
    return result;
}

int main(int argc, char* argv[]) {
    if (argc < 3>) {
        printf("Usage: %s base exponent \n", argv[0]);
        return 1;
    }
    double base = atof(argv[1]);
    int exponent = atoi(argv[2]);
    auto result = power(base, exponent);
    printf("%g ^ %d is %g\n", base, exponent, result);
    return 0;
}
```

## 编写CMakeLists.txt，并保证位于根目录下：

```
# CMake最低版本要求
cmake_minimum_required(VERSION 2.8)

# 项目信息
project(demo1)

# 指定生成目标
add_executable(demo main.cc)
```

CMakeLists.txt的语法比较简单，由命令、注释和空格组成。命令不区分大小写，符号*#*后面的内容被认为为注释。命令由命令名称、小括号和参数组成，参数之间使用空格进行间隔。

## 编译项目

在项目根目录执行**cmake .**,  得到Makefile后再使用*make*命令编译得到demo可执行文件。

# 多个源文件

## 同一目录多个源文件

```
# /examples/cmake/demo2/CMakeLists.txt

# CMake最低版本要求
cmake_minimum_required(VERSION 2.8)

# 项目信息
project(demo2)

# 指定生成目标
# add_executable(demo main.cc MathFunctions.cc)

# 查找当前目录下的所有源文件，并保存到DIR_SRCS变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(demo ${DIR_SRCS})
```

## 多目录多源文件

```
# /examples/cmake/demo3/CMakeLists.txt

# CMake最低版本要求
cmake_minimum_required(VERSION 2.8)

# 项目信息
project(demo3)

# 查找当前目录下的所有源文件，并保存到DIR_SRCS变量
aux_source_directory(. DIR_SRCS)

# 添加math子目录
add_subdirectory(math)

# 指定生成目标
add_executable(demo ${DIR_SRCS})

# 添加链接库
target_link_libraries(demo MathFunctions)


# /examples/cmake/demo3/math/CMakeLists.txt

# 查找当前目录下所有源文件并保存到变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library(MathFunctions ${DIR_LIB_SRCS})
```

# 自定义编译选项

```
# /examples/cmake/demo4/CMakeLists.txt

# CMake最低版本要求
cmake_minimum_required(VERSION 2.8)

# 项目信息
project(demo4)

# 加入配置头文件，处理CMake对源码的设置
configure_file("${PROJECT_SOURCE_DIR}/config.h.in" "${PROJECT_SOURCE_DIR}/config.h")

# 是否使用自己的MathFunctions库
option(USE_MYMATH "Use provided math implementation" ON)

# 是否加入MathFunctions库
if (USE_MYMATH)
    include_directories("${PROJECT_SOURCE_DIR}/math")
    add_subdirectory(math)
    set(EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
endif(USE_MYMATH)

# 查找当前目录下的所有源文件，并保存到DIR_SRCS变量
aux_source_directory(. DIR_SRCS)

# 指定生成目标
add_executable(demo ${DIR_SRCS})

# 添加链接库
target_link_libraries(demo ${EXTRA_LIBS})


# /examples/cmake/demo4/math/CMakeLists.txt

# 查找当前目录下所有源文件并保存到变量
aux_source_directory(. DIR_LIB_SRCS)

# 生成链接库
add_library(MathFunctions ${DIR_LIB_SRCS})
```

1. *configure_file* 命令用于加入一个配置头文件config.h, 由CMake从config.h.in生成，通过这样的机制，可以通过预定义一些参数和变量来控制代码生成
2. option命令添加一个*USE_MYMATH*选项，默认值设为*ON*
3. 根据*USE_MYMATH*变量值来决定是否使用自己编写的MathFunctions库

## 编写config.h.in文件

```
# /examples/cmake/demo4/config.h.in
#cmakedefine USE_MYMATH
```

## 编译项目

```
$ ccmake
```

# 安装和测试

CMake可以指定安装规则和添加测试，分别可以通过产生Makefile后使用*make install*和*make test*来执行。

## 定制安装规则

```
# /examples/cmake/demo5/math/CMakeLists.txt
aux_source_directory(. DIR_LIB_SRCS)
add_library (MathFunctions ${DIR_LIB_SRCS})

# 指定MathFunctions库的安装路径
install(TARGETS MathFunctions DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)

# /examples/cmake/demo5/CMakeLists.txt
...

# 指定安装路径
install(TARGETS demo DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/config.h" DESTINATION include)
```

通过定制，生成的demo文件和libMathFunctions.o文件会被复制到/usr/local/bin中，MathFunctions.h和config.h被复制到/usr/local/include中。
```
$ sudo make install
$ ls /usr/local/bin
$ ls /usr/local/include
```

## 为工程添加测试

CMake提供了一个CTest的测试工具，在根目录的CMakeLists.txt中调用一系列的add_test命令即可。

```
# 启动测试
enable_testing()

# 测试是否成功运行
add_test(test_run demo 5 2)

# 测试帮助信息是否可以正常提示
add_test(test_usage demo)
set_tests_properties(test_usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage: .* base exponent")

# 测试
add_test(test_5_2 demo 5 2)
set_tests_properties(test_5_2 PROPERTIES PASS_REGULAR_EXPRESSION "is 25")

# 测试一组数据
# 定义宏来简化
macro(do_test arg1 arg2 result)
    add_test(test_${arg1}_${arg2} demo ${arg1} ${arg2})
    set_tests_properties(test_${arg1}_${arg2} PROPERTIES PASS_REGULAR_EXPRESSION ${result})
endmacro(do_test)
do_test(5 2 "is 25")
do_test(10 5 "is 100000")
```

启动测试：
```
$ make test
```

# 支持GDB

仅需指定*Debug*模式下开启*-g*选项，之后可以直接对生成程序使用gdb来调试：
```
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g -ggdb")
set(CMAKE_CXX_FLAGS_RELEASE "$ENV{CXXFLAGS} -O3 -Wall")
```

# 添加环境检查

## 添加CheckFunctionExists宏

在顶层CMakeLists.txt文件添加CheckFunctionExists.cmake宏，并调用*check_function_exists*命令测试链接器是否能够在链接阶段找到pow函数，将其放在configure_file命令前。

```
# /examples/cmake/demo6/CMakeLists.txt

# 检查系统是否支持pow函数
include(${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
check_function_exists(pow HAVE_POW)
```

## 预定义相关宏变量

```
# /examples/cmake/demo6/config.h.in
// Does the platform provide pow function?
#cmakedefine HAVE_POW
```

## 在代码中使用宏和函数
```
# /examples/cmake/demo6/main.cc

#ifdef HAVE_POW
    printf("Now we use the standard library. \n");
    double result = pow(base, exponent);
#else
    printf("Now we use our own Math Library. \n");
    double result = power(base, exponent);
#endif // HAVE_POW
```

# 添加版本号
```
# /examples/cmake/demo7/CMakeLists.txt
project(demo7)
set(DEMO_VERSION_MAJOR 1)
set(DEMO_VERSION_MINOR 0)

# /examples/cmake/demo7/config.h.in
// Configured options and settings for demo7
#define DEMO_VERSION_MAJOR @DEMO_VERSION_MAJOR@
#define DEMO_VERSION_MINOR @DEMO_VERSION_MINOR@

# /examples/cmake/demo7/main.cc
printf("%s Version %d.%d\n", argv[0], DEMO_VERSION_MAJOR, DEMO_VERSION_MINOR);
```

# 生成安装包

生成各平台的二进制安装包和源码安装包，需要用到CPack，由CMake提供专门用于打包。

```
# /examples/cmake/demo8/CMakeLists.txt?尾部添加如下命令

# 构建一个CPack安装包
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CAMKE_CURRENT_SOURCE_DIR}/License.txt")
set(CPACK_PACKAGE_VERSION_MAJOR "${DEMO_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${DEMO_VERSION_MINOR}")
include(CPack)
```

1. 生成二进制安装包
```
$ cpack -C CPackConfig.cmake
```

2. 生成源码安装包
```
$ cpack -C CPackSourceConfig.cmake
```

3. 浏览生成文件
```
$ ls demo8-*
$ sh demo8-1.0.1-Linux.sh
$ ./demo8-1.0.1-Linux/bin/demo 5 2
```

# 将其他平台的项目迁移到CMake

CMake 可以很轻松地构建出在适合各个平台执行的工程环境。而如果当前的工程环境不是 CMake ，而是基于某个特定的平台，是否可以迁移到 CMake 呢？答案是可能的。下面针对几个常用的平台，列出了它们对应的迁移方案。

#### autotools
* [am2cmake](https://projects.kde.org/projects/kde/kdesdk/kde-dev-scripts/repository/revisions/master/changes/cmake-utils/scripts/am2cmake) 可以将 autotools 系的项目转换到 CMake，这个工具的一个成功案例是 KDE
* [Alternative Automake2CMake](http://emanuelgreisen.dk/stuff/kdevelop_am2cmake.php.tgz) 可以转换使用 automake 的 KDevelop 工程项目
* [Converting autoconf tests](http://www.cmake.org/Wiki/GccXmlAutoConfHints)

#### qmake
* [qmake converter](http://www.cmake.org/Wiki/CMake:ConvertFromQmake) 可以转换使用 QT 的 qmake 的工程

#### Visual Studio
* [vcproj2cmake.rb](http://vcproj2cmake.sf.net/) 可以根据 Visual Studio 的工程文件（后缀名是 .vcproj 或 .vcxproj）生成 CMakeLists.txt 文件
* [vcproj2cmake.ps1](http://nberserk.blogspot.com/2010/11/converting-vc-projectsvcproj-to.html) vcproj2cmake 的 PowerShell 版本
* [folders4cmake](http://sourceforge.net/projects/folders4cmake/) 根据 Visual Studio 项目文件生成相应的 “source_group” 信息，这些信息可以很方便的在 CMake 脚本中使用。支持 Visual Studio 9/10 工程文件

#### CMakeLists.txt 自动推导
* [gencmake](http://websvn.kde.org/trunk/KDE/kdesdk/cmake/scripts/) 根据现有文件推导CMakeLists.txt文件
* [CMakeListGenerator](http://www.vanvelzensoftware.com/postnuke/index.php?name=Downloads&req=viewdownload&cid=7) 应用一套文件和目录分析创建出完整的CMakeLists.txt文件。仅支持Win32平台

# 相关链接

1. 官方主页：http://www.cmake.org/
2. 官方文档：http://www.cmake.org/cmake/help
3. 官方教程：http://www.cmake.org/cmake/help/cmake_tutorial.html
4. [Wiki](http://www.cmake.org/Wiki/CMake#Basic_CMakeLists.txt_from-scratch-generator)
5. 视频教程：https://www.youtube.com/watch?v=CLvZTyji_Uw
6. 类似工具：[SCons](http://scons.org/) --- Eric S. Raymond、Timothee Besset、Zed A. Shaw 等大神力荐的项目架构工具。和 CMake 的最大区别是使用 Python 作为执行脚本