1️⃣ message() 是干嘛的？
`message()` 就是 **CMake 的“打印日志 / 调试输出”工具**  
在你 `cmake ..` 配置阶段用来看：
- 变量值对不对
- 走到哪一步了
- 条件分支有没有进来
- 给使用你项目的人提示信息

⚠️ 注意：它发生在 **配置阶段(cmake决定怎么编译）**，不是编译阶段（编译器真正开始编译）

2️⃣ 一些用法
1. 基础用法（最常见的）
```cmake
message("Hello CMake")
```
输出效果：
```text
Hello CMake
```
==一般用来确认：==
- CMakeLists.txt 有没有被执行
- 某段逻辑有没有走到

2. 打印变量
```cmake
message("CMAKE_CXX_COMPILER = ${CMAKE_CXX_COMPILER}")
message("PROJECT_NAME = ${PROJECT_NAME}")
```
如果变量没定义，会输出空字符串，不会报错。

👉 调 bug 神器，尤其是：

find_package

option()

if() 判断

3️⃣不同“级别”的 message
- 常见格式
```cmake
message(<MODE> "内容")
```
-  常用 MODE 一览

|MODE|用途|会不会中断|
|---|---|---|
|`STATUS`|普通状态信息（最常用）|❌|
|`WARNING`|警告|❌|
|`AUTHOR_WARNING`|给 CMakeLists 作者的警告|❌|
|`SEND_ERROR`|错误（配置结束时报错）|⚠️|
|`FATAL_ERROR`|致命错误|✅ 立刻停止|
推荐的标准写法
```cmake
message(STATUS "Building project: ${PROJECT_NAME}")
```
输出会带 `--`：
`-- Building project: MyProject`
