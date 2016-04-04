---
title: 在CMake中使用SWIG构建封装模块
date: 2016-04-02 15:25:05
tags:
  - CMake
  - swig
  - Cpp
  - python
categories:
  - 学习小结
feature: assets/images/features/cmake_logo.jpg
description: "cmake支持SWIG的使用，因此在这里总结下在CMAKE中使用swig的方法，以KMCLib为例子。"
toc: true
---
由于CMake与SWIG的相互支持，使得在cmake脚本中直接定义封装模块的构建成为可能，KMCLib中也是使用这种方式直接使用swig进行自动封装。也就是在生成的Makefile中包含了使用swig命令封装库文件。

在这里我主要参考了[swig文档中关于cmake的部分](http://www.swig.org/Doc1.3/Introduction.html#Introduction_build_system)和[cmake文档中使用swig的部分](https://cmake.org/cmake/help/v3.5/module/UseSWIG.html)。

<!-- more -->

还是上例子吧
### 简单的例子
``` CMAKE
# This is a CMake example for Python

FIND_PACKAGE(SWIG REQUIRED)  
INCLUDE(${SWIG_USE_FILE})  

FIND_PACKAGE(PythonLibs)  
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})

INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

SET(CMAKE_SWIG_FLAGS "")

SET_SOURCE_FILES_PROPERTIES(example.i PROPERTIES CPLUSPLUS ON)
SET_SOURCE_FILES_PROPERTIES(example.i PROPERTIES SWIG_FLAGS "-includeall")
SWIG_ADD_MODULE(example python example.i example.cxx)
SWIG_LINK_LIBRARIES(example ${PYTHON_LIBRARIES})
```

### 每条语句分析：
```
FIND_PACKAGE(SWIG REQUIRED)
```
寻找swig有没有安装在本机上，如果没有，由于指定了`REQUIRED`参数，cmake会产生一个fatal error然后停止构建。

---

```
INCLUDE(${SWIG_USE_FILE})  
```
载入预定义的cmake模块

---

```
FIND_PACKAGE(PythonLibs)  
```
查看python是否安装并生成几个变量确定python的头文件以及库文件在哪里。变量列表：
{% alert info %}
<br>
<code>PYTHONLIBS_FOUND</code>           - have the Python libs been found<br>
<code>PYTHON_LIBRARIES</code>           - path to the python library<br>
<code>PYTHON_INCLUDE_PATH</code>     - path to where Python.h is found (deprecated)<br>
<code>PYTHON_INCLUDE_DIRS</code>        - path to where Python.h is found<br>
<code>PYTHON_DEBUG_LIBRARIES</code>     - path to the debug library (deprecated)<br>
<code>PYTHONLIBS_VERSION_STRING</code>  - version of the Python libs found (since CMake 2.8.8)<br>
{% endalert %}

---

```
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})
```
将python的include路径加入编译的include路径中。

---

```
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})
```
将当前路径加入到编译的include路径中。

---

```
SET(CMAKE_SWIG_FLAGS "")
```
`CMAKE_SWIG_FLAGS`可以用来设置给所有要调用swig的地方的参数，例如`-python`, `-c++`等

---

```
SET_SOURCE_FILES_PROPERTIES(example.i PROPERTIES CPLUSPLUS ON)
SET_SOURCE_FILES_PROPERTIES(example.i PROPERTIES SWIG_FLAGS "-includeall")
```
如果某个swig生成的原代码需要某种参数就要通过这个macro进行设置，上面的第一个应该就相当于在swig的参数中添加`-c++`以生成`cxx`文件。

---

```
SWIG_ADD_MODULE(example python example.i example.cxx)
```
这个cmake函数是定义swig生成的最终模块的名称和封装的语言

---

```
SWIG_LINK_LIBRARIES(example ${PYTHON_LIBRARIES})
```
将库与swig模块一起链接。上文已说明`PYTHON_LIBRARY`变量存放的是python库的路径。

### KMCLib中CMakeLists.txt中的swig
```
# Make sure the swig package is loaded.
find_package(SWIG REQUIRED)
include(${SWIG_USE_FILE})

include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories( ${KMCLib_SOURCE_DIR}/src )

# Set the flags.
set(CMAKE_SWIG_FLAGS "")

# Set the properties for the interface file.
set_source_files_properties(backend.i PROPERTIES CPLUSPLUS ON)
set_source_files_properties(backend.i PROPERTIES SWIG_FLAGS "")

...

# Add the target.
swig_add_module( Backend python backend.i )

# -----------------------------------------------------------------------------
# LINK
# -----------------------------------------------------------------------------

message( STATUS "Creating makefiles for system: ${CMAKE_SYSTEM}")
# For Mac OS X
if(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")

  # To force a Mac OSX with macports Python, use with -DMACPORT=TRUE
  if (MACPORT)
    set(CMAKE_LIBRARY_PATH "/opt/local/Library/Frameworks")
    message( STATUS "Looking for libraries in ${CMAKE_LIBRARY_PATH}" )
  endif()

  find_library( PYTHON_LIB python )
  message( STATUS "Using python library ${PYTHON_LIB}")
  swig_link_libraries( Backend ${PYTHON_LIB} src )

# For Linux (Ubuntu 12.04 LTS)
else()#${CMAKE_SYSTEM_NAME} MATCHES "Linux")
  swig_link_libraries( Backend src )
endif()
```
按照上面的介绍，这里的cmake也不难理解了。
