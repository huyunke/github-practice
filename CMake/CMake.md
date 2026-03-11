##### 注释
- 注释行：`CMake`使用`#`进行行注释，可以放在任何位置
- 注释块：：`CMake`使用`#[[]]`进行块注释

##### 基础项目构建流程
- `cmake_minimum_require()`:指定使用的 cmake 的最低版本,可选，非必须，如果不加可能会有警告
- `project(项目名)`:定义工程名([project()](project().md)详细食用方案)
- `add_executable(可执行程序名 源文件1 源文件2...)`,各个源文件之间可以采用空格或者分号进行分隔
- 构建隔离，为了避免生成的中间文件污染源码目录，建议创建`build`目录，并在其中执行`cmake...`和`make`(但是CLion 和大多数现代 IDE 默认采用脱源构建：即编译产物不会污染源码目录)
- `CMake`会自动找到依赖的头文件，因此不需要特别指定，当头文件修改的时候，会重新编译依赖它的目标文件
- 构建`cmake`项目常用的命令
```cmake
cmake -B build #生成构建目录
cmake --build build #执行构建
./build/可执行程序名 #运行可执行程序
```

##### 定义变量
```cmake
# 定义普通变量
set(MY_VARIABLE "hello world")

# 定义列表变量
set(MY_LIST item1 item2 item3)
```
- 定义列表变量的使用场景（不过要是文件多起来了，这也不优雅）
```CMake
set(SRC_LIST 源文件1 源文件2 源文件3)
add_executable(app ${SRC_LIST})#通过${}获取变量的值
```

除此之外还可以`定义缓存变量`和`内部缓存变量`，不过我不知道在什么情况下需要定义这两种变量，所以不讲了🥹

##### 指定使用的C++标准（我没有试过这个）
- 在编写C++程序的时候可能会用到C++11，C++14等新特性，所以需要在编译命令中指定出要使用哪个标准
- app为可执行文件名称
```bash
$ g++ *.cpp -std=c++11 -o app
```

- 除了命令行，也可以通过CMake来指定C++标准，C++标准对应有一宏叫做`CMAKE_CXX_STANDARD`
1. 在CMakeLists.txt中通过set命令指定
```cmake
#增加-std=c++11
set(CMAKE_CXX_STANDARD 11)
```
2. 在执行cmake命令的时候指定出这个宏的值
```cmake
cmake CMakeLists.txt文件路径 -DCMAKE_CXX_STANDARD=11
```

##### 文件搜索
- 如果一个项目里边的源文件很多，在编写`CMakeLists.txt`文件的时候不可能将项目目录中的各个文件罗列出来，那太麻烦了，而且也很容易出错，这里有两种办法进行文件搜索
- 方法一：`file`支持**递归搜索**
```cmake
file(GLOB 变量名 要搜索的文件路径和文件类型)
file(GLOB_RECURSE 变量名 要搜索的文件路径和文件类型)
```
- `GLOB`:将指定目录下搜索到的满足条件的所有文件名生成一个列表，并将其存储到变量中
- `GLOB_RECURSE`:递归搜索指定目录，将搜索到的满足条件的文件名生成一个列表，并将其存储到变量中
- 例子(**CMAKE_CURRENT_SOURCE_DIR宏表示当前访问的CMakeLists.txt文件所在的路径**)
```cmake
#关于要搜索的文件路径和类型可以加双引号，也可不加
file(GLOB MAIN_SRC ${CMAKE_CURRENT_SOURCE_DIR}/src/*.cpp)
file(GLOB MAIN_HEAD ${CMAKE_CURRENT_SOURCE_DIR}/include/*.h)
```
- 方法二：`aux_source_directory`
```bash
aux_source_directory(要搜索的目录 变量名)
```

##### 指定头文件路径
```cmake
include_directories(头文件路径)
```

##### 制作库
- 关于[静态库和动态库](静态库和动态库.md)的介绍
- 源文件列表可以通过`file`或者`aux_source_directory`来获得
1. 制作静态库
```cmake
add_library(库名称 STATIC 源文件1 [源文件2] ...) 
```

2. 制作动态库
```cmake
add_library(库名称 SHARED 源文件1 [源文件2] ...) 
```

- 指定存放路径
```cmake
#设置动态库的存放路径
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

#设置静态库的存放路径
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib
```

##### add_subdirectory（目录名）
- 核心作用是**进入子目录并执行那里的 `CMakeLists.txt`**
- **创建子工程**：它建立了一个父子关系。父目录定义的变量通常可以被子目录继承
- **独立构建目标**：通常每个子目录下都有自己的 `add_executable()` 或 `add_library()`。当你运行顶层编译时，CMake 会按顺序把它们全算进去

##### target_link_libraries()
1. 基本语法
```cmake
target_link_libraries(<target> [item1] [item2] [...]
                      [[PRIVATE|PUBLIC|INTERFACE] <item>] ...)
```
- **`<target>`**：你的目标文件（通常由 `add_executable()` 或 `add_library()` 定义）
    
- **`<item>`**：你想链接的库（可以是库的名字、完整路径，或者是另一个 CMake Target）
2. “传播范围”
当你链接一个库时，你需要决定这个“依赖关系”是否要传递给别人：

|**关键字**|**含义**|**场景举例**|
|---|---|---|
|**`PRIVATE`**|**只有我自己用**。链接的库仅用于构建当前目标，不会传递给依赖我的其他目标。|你的 App 内部使用了 `sqlite` 处理数据，但别人调用你的 App 接口时不需要知道 `sqlite`。|
|**`INTERFACE`**|**我自己不用，给别人用**。当前目标不需要这个库，但任何链接到我的目标都需要它。|你写了一个“纯头文件”库（Header-only），它依赖 `math` 库，你自己不编译，但别人用你时必须链接 `math`。|
|**`PUBLIC`**|**我和别人都要用**。既用于构建自己，也会自动传递给所有依赖我的目标。|你写了一个动态库 `libA`，其头文件里引用了 `libB` 的类型。那么任何用 `libA` 的人也必须链接 `libB`。|
3. 需要注意的地方
`target_link_libraries` 必须放在 `add_executable` 或 `add_library` **之后**，否则 CMake 找不到目标会报错。
