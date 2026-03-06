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

##### 指定使用的C++标准
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
- 方法一：`file`
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