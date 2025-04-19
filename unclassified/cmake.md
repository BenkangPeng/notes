参考链接 ：[CMake Tutorial — CMake 3.30.3 Documentation](https://cmake.org/cmake/help/latest/guide/tutorial/)

配套代码在`/path/to/cmake/Help/guide/tutorial/`

### Step 1

```shell
mkdir Step1_build
cd Step1_build
cmake ../Step1
cmake --build .
```



```cmake
# TODO 1: Set the minimum required version of CMake to be 3.10
cmake_minimum_required(VERSION 3.10) # VERSION区分大小写

# TODO 2: Create a project named Tutorial
project(Tutorial VERSION 1.0) # Tutorial是项目名称
# TODO 7: Set the project version number as 1.0 in the above project command

# TODO 6: Set the variable CMAKE_CXX_STANDARD to 11
#         and the variable CMAKE_CXX_STANDARD_REQUIRED to True
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)#要在add_executable之前添加cxx_standard

# TODO 8: Use configure_file to configure and copy TutorialConfig.h.in to
#         TutorialConfig.h
configure_file(TutorialConfig.h.in TutorialConfig.h)

# TODO 3: Add an executable called Tutorial to the project
# Hint: Be sure to specify the source file as tutorial.cxx
add_executable(Tutorial tutorial.cxx)# 添加可执行文件

# TODO 9: Use target_include_directories to include ${PROJECT_BINARY_DIR}
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
# PROJECT_BINARY_DIR 是一个自动定义的变量，它指向项目的构建目录
```

* `configure_file`

`configure_file(TutorialConfig.h.in TutorialConfig.h)`命令用于根据一个文件模板(`TutorialConfig.h.in`)，生成另一个配置文件(`TutorialConfig.h`)。

例如模板文件TutorialConfig.h.in如下：

```cmake
#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
```

它定义了两个常量`Tutorial_VERSION_MAJOR,Tutorial_VERSION_MINOR`,但未指定特定的值，而是用占位符`@...@`代替。而这两个常量的值**隐式**地定义在`CMakeLists.txt`的`project(Tutorial VERSION 1.0)`中(`Tutorial_VERSION_MAJOR`为1，表示大版本号，`Tutorial_VERSION_MINOR`为0，表示小版本号)，换言之，`Tutorial_VERSION_MAJOR,Tutorial_VERSION_MINOR`是内置的常量。

结合`TutorialConfig.h.in`和`CMakeLists.txt`，可以得到这两个常量的值，从而生成`TutorialConfig.h`：

```cmake
#define Tutorial_VERSION_MAJOR 1
#define Tutorial_VERSION_MINOR 0
```

在主程序`Tutorial.cxx`中`#include TutorialConfig.h`即可加入这两个常量的定义。

同样地，我们可以在`CMakeLists.txt`中自定义变量：

```cmake
set(Tutorial_Time 20240926)
```

在`TutorialConfig.h.in`中添加模板：

```cmake
#define Tutorial_Time @Tutorial_Time@
```

从而生成了配置文件`TutorialConfig.h`:

```cmake
#define Tutorial_Time 20240926
```

### Step 2

`Step2/CMakeLists.txt`

```cmake
cmake_minimum_required(VERSION 3.10)

# set the project name and version
project(Tutorial VERSION 1.0)

# specify the C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# configure a header file to pass some of the CMake settings
# to the source code
configure_file(TutorialConfig.h.in TutorialConfig.h)

# TODO 2: Use add_subdirectory() to add MathFunctions to this project
add_subdirectory(MathFunctions)
#表示add_subdirectory(<directory>),<directory>表示子目录路径
#将子目录添加到构建系统中，
# 并在该子目录中查找 CMakeLists.txt 文件。此命令使得项目可以支持多层结构，可以在每个子目录中定义目标和构建规则。
# add the executable
add_executable(Tutorial tutorial.cxx)

# TODO 3: Use target_link_libraries to link the library to our executable
target_link_libraries(Tutorial PUBLIC MathFunctions)#链接库，MathFunctions是library名称，而不是路径名

# TODO 4: Add MathFunctions to Tutorial's target_include_directories()
# Hint: ${PROJECT_SOURCE_DIR} is a path to the project source. AKA This folder!
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}" "${PROJECT_SOURCE_DIR}/MathFunctions")
# 设置目标头文件的include目录,${PROJECT_BINARY_DIR}表示build目录，PROJECT_SOURCE_DIR表示源代码目录
# add the binary tree to the search path for include files
# so that we will find TutorialConfig.h

```



`MathFunctions/CMakeLists.txt`

```cmake
# TODO 1: Add a library called MathFunctions with sources MathFunctions.cxx
# and mysqrt.cxx
# Hint: You will need the add_library command
add_library(MathFunctions MathFunctions.cxx )#新建MathFunction library

# TODO 7: Create a variable USE_MYMATH using option and set default to ON
option(USE_MYMATH "Use tutorial provided math implementation" ON)#创建一个选项,默认开启
# TODO 8: If USE_MYMATH is ON, use target_compile_definitions to pass
if(USE_MYMATH)#如果cmake命令中添加选项-D USE_MYMATH
    target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")
    #给MathFunctions创建一个预处理宏：USE_MYMATH
    #PRIVATE表示这个宏定义是隐私的，仅在 MathFunctions目标的编译中使用，而不会被链接到该目标的其他目标所看到。

    # USE_MYMATH as a precompiled definition to our source files

    # TODO 12: When USE_MYMATH is ON, add a library for SqrtLibrary with
    # source mysqrt.cxx
    add_library(SqrtLibrary STATIC mysqrt.cxx)#创立SqrtLibrary库
    # TODO 13: When USE_MYMATH is ON, link SqrtLibrary to the MathFunctions Library
    target_link_libraries(MathFunctions PRIVATE SqrtLibrary)#链接到MathFunctions库中
endif()

```

`MathFunctions/MathFunctions.cxx`

```cpp
#include "MathFunctions.h"

// TODO 11: include cmath
#include<cmath>
// TODO 10: Wrap the mysqrt include in a precompiled ifdef based on USE_MYMATH
#ifdef USE_MYMATH
  #include "mysqrt.h"
#endif


namespace mathfunctions {
double sqrt(double x)
{
  // TODO 9: If USE_MYMATH is defined, use detail::mysqrt.
  // Otherwise, use std::sqrt.
  #ifdef USE_MYMATH
    return detail::mysqrt(x);
  #else
    return std::sqrt(x);
  #endif()
}
}
```



### Step 3





























