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
