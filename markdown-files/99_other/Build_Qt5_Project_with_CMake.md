# Build Qt5 Project with CMake
[TOC]

cmake可以自动查找、使用Qt库，Qt4和Qt5都可以使用相同的cmake构建系统。对于Qt5工程的构建，cmake版本最低要求是3.1.0，Qt4对cmake的版本要求我没找到相关的文档说明，直接用高版本的cmake应该都ok。
## Qt构建工具
Qt集成了一些工具来自动生成代码，`moc`用于生成原对象代码，`uic`用于生成布局相关代码，`rcc`生成资源相关的内容。cmake构建Qt工程的原理就是它可以在合适的时机调用这些工具自动生成代码，先看一个Qt官方文档中的例子：
```
cmake_minimum_required(VERSION 3.1.0)

project(testproject)

# Find includes in corresponding build directories
set(CMAKE_INCLUDE_CURRENT_DIR ON)
# Instruct CMake to run moc automatically when needed
set(CMAKE_AUTOMOC ON)
# Create code from a list of Qt designer ui files
set(CMAKE_AUTOUIC ON)

# Find the QtWidgets library
find_package(Qt5Widgets CONFIG REQUIRED)

# Populate a CMake variable with the sources
set(helloworld_SRCS
    mainwindow.ui
    mainwindow.cpp
    main.cpp
)
# Tell CMake to create the helloworld executable
add_executable(helloworld WIN32 ${helloworld_SRCS})
# Use the Widgets module from Qt 5
target_link_libraries(helloworld Qt5::Widgets)
```
## find_package
`find_package`用于查找库的位置，cmake本身不提供任何搜索库的便捷方法，所有搜索库并给变量赋值的操作必须由cmake代码完成，它有两种搜索模式：
- `MODULE`模式：搜索`CMAKE_MODULE_PATH`指定路径下的FindXXX.cmake文件，执行该文件从而找到XXX库。其中，具体查找库并给XXX_INCLUDE_DIRS和XXX_LIBRARIES两个变量赋值的操作由FindXXX.cmake模块完成。
- `CONFIG`模式：搜索XXX_DIR指定路径下的XXXConfig.cmake文件，执行该文件从而找到XXX库。其中具体查找库并给XXX_INCLUDE_DIRS和XXX_LIBRARIES两个变量赋值的操作由XXXConfig.cmake模块完成。

以上XXX均代表搜索目标名，在以上例子中等同于`Qt5Widgets`。`find_package`默认使用`MODULE`模式，没找到再使用`CONFIG`模式，在上面的例子中指定使用CONFIG模式，可以忽略。Qt提供了所有模块的XXXConfig.cmake文件，位置在`PATH_TO_QT/QT_VERSION/QT_ARCH/lib/cmake`目录下。
如果`find_package`报错，那么只需要在`find_package`前添加一行：`set(CMAKE_PREFIX_PATH "PATH_TO_QT/QT_VERSION/QT_ARCH/lib/cmake")`，真的方便。
注：`PATH_TO_QT`代表Qt的安装目录；`QT_VERSION`代表Qt库的版本；`QT_ARCH`代表Qt库的架构。
## AUTOMOC
`set(CMAKE_AUTOMOC ON)`用于设置AUTOMOC属性，检查c++文件以确定它们需要`moc`来自动生成代码。如果`头文件`中存在`Q_OBJECT`或者`Q_GADGET`宏，那么cmake会调用`moc`生成名为`moc_<basename>.cpp`的元对象代码；如果在`cpp文件`中发现同样的宏，`moc`生成的文件则是`<basename>.moc`。
生成的`moc_*.cpp`和`*.moc`文件都在构建目录中，所以需要`set(CMAKE_INCLUDE_CURRENT_DIR ON)`设置构建目录为头文件目录。
cmake还可以通过`CMAKE_AUTOMOC_MOC_OPTIONS`变量设置`moc`命令在运行时的命令行参数，默认为空。
## AUTOUIC
`set(CMAKE_AUTOUIC ON)`用于设置AUTOUIC属性，检查是否需要运行`uic`工具。如果预处理器发现某个源文件`#include`了一个`ui_<basename>.h`头文件，并且存在`<basename>.ui`的UI文件，那么`uic`工具就会被调用了。
和`moc`类似，生成的`ui_*.h`文件也在构建目录中。

## AUTORCC
`set(CMAKE_AUTORCC ON)`用于设置AUTORCC属性，检查是否需要调用`rcc`工具，如果源文件中有`.qrc`为后缀的资源文件，cmake就会调用`rcc`工具。cmake通过`AUTORCC_OPTIONS`给`rcc`工具传递命令行参数。
一个qml工程的构建样例：
```
cmake_minimum_required(VERSION 3.12)
project(cmake_qt_qml)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set (CMAKE_PREFIX_PATH "C:/Qt/5.12.0/mingw73_64")

find_package(Qt5Core REQUIRED)
find_package(Qt5Widgets REQUIRED)
find_package(Qt5Quick REQUIRED)

add_executable(cmake_qt_qml main.cpp qml.qrc)

target_link_libraries(cmake_qt_qml Qt5::Core)
target_link_libraries(cmake_qt_qml Qt5::Widgets)
target_link_libraries(cmake_qt_qml Qt5::Quick)
```
## 参考文档
- [stackoverflow: How to configure CLion IDE for Qt Framework?](https://stackoverflow.com/questions/30235175/how-to-configure-clion-ide-for-qt-framework)
- [cmake: cmake-qt](https://cmake.org/cmake/help/v3.1/manual/cmake-qt.7.html#manual:cmake-qt%287%29)
- [Qt: CMake Manual](http://doc.qt.io/qt-5/cmake-manual.html)
