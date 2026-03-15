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

### C++ 标准演进概览表 📊

|**标准版本**|**核心定位**|**标志性特性**|**对开发者的影响**|
|---|---|---|---|
|**C++98/03**|**奠基时代** 🏛️|类 (Class)、模板 (Template)、STL 容器、异常处理|确立了面向对象和泛型编程的基础，但需手动管理内存。|
|**C++11**|**现代革命** 🚀|**移动语义 (Move)**、`auto`、智能指针、Lambda、`nullptr`|**里程碑式更新**。极大提升了性能和代码简洁度，减少了内存泄漏。|
|**C++14**|**小步快跑** 🛠️|泛型 Lambda、二进制字面量、`decltype(auto)`|对 C++11 的微调和完善，让语法更顺手。|
|**C++17**|**易用性提升** 💎|**结构化绑定**、`std::filesystem`、`if constexpr`、`std::optional`|强化了标准库，处理文件和复杂逻辑变得更加标准化。|
|**C++20**|**大刀阔斧** 🌊|**Concepts (概念)**、**Modules (模块)**、**Coroutines (协程)**、Ranges|**第二次重大变革**。解决了编译慢、模板报错难懂等陈年旧疾。|
|**C++23**|**精雕细琢** ✨|`std::expected`、多维数组访问 `[]`、`std::print`|进一步增强生产力，让 C++ 写起来更接近 Python 等现代语言。|
