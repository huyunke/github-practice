CMake 的典型使用是围绕一个或多个名为 `CMakeLists.txt`. 该文件有时被称为“列表文件”或“CML”。在给定的软件项目中，任何我们希望向 CMake 提供如何处理该目录或子目录本地文件和作的指令，都会存在一个 `CMakeLists.txt`。每个命令由一组命令组成，描述与构建软件项目相关的信息

并不是软件项目中的每个目录都需要 CML，但==强烈建议项目根节点包含 CML==。这将作为 CMake 在配置时初始设置的切入点。

```cmake
#调用时，它确保 CMake 会采用所列版本的行为。如果在包含上述代码的 CML 上调用后续版本的 CMake，它将完全像 CMake 3.23 一样运作。
cmake_minimum_required(VERSION 3.23)

#常见写法：参数为项目名称
project(MyProjectName)
#另一种写法：指定项目语言（推荐）C：C 语言,CXX：C++ 语言
projext(MyProject LANGUAGES C CXX)
```

CMake 中的路径通常是绝对路径，或相对于 [`CMAKE_CURRENT_SOURCE_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_CURRENT_SOURCE_DIR.html#variable:CMAKE_CURRENT_SOURCE_DIR "CMAKE_CURRENT_SOURCE_DIR") 路径。我们还没谈过类似的变量，可以把这理解为“相对于当前 CML 的位置”。

### add_executable()创建目标
`add_executable()` 用来告诉 CMake：把哪些源文件编译成一个可执行程序
例如：
```cmake
add_executable(BookSystem
    main.cpp
    user.cpp
    userdao.cpp
)

#另一种写法：改项目名只要在project()里重写就行
add_executable(${PROJECT_NAME}
    main.cpp
    user.cpp
    utils.cpp
)
```
- 生成一个叫 **BookSystem** 的可执行程序
- 由这些 `.cpp` 文件编译而成

==`add_executable` 只负责编译，不负责链接库（`target_link_libraries`负责)==
使用过程中的常见错误
- 把头文件当必须项
- 忘了 `main.cpp`，可执行程序必须有`main()`

### add_library()在 CMake 中定义一个“库目标（target）“
基本用法
```cmake
add_library(MyLib)
```
创建一个名为 `MyLib` 的库 target(类型暂时不确定)

### target_sources()给已经存在的目标（target）追加源文件

基本语法（目标名可以用`${PROJECT_NAME}`代替）
```cmake
target_sources(
    目标名
    <作用域>
        源文件1
        源文件2
)
```
#### 三种作用域
1️⃣ PRIVATE（最常用）
`target_sources(app PRIVATE user.cpp)`

含义：
- 这些源文件 **只属于当前 target**
- 不会影响依赖它的其他 target
    
📌 **可执行程序几乎全用 PRIVATE**

---
2️⃣ PUBLIC（给别人用）
`target_sources(mylib PUBLIC lib.cpp)`

- 自己用
- **依赖 mylib 的 target 也能看到**

📌 常用于库

---
3️⃣ INTERFACE（自己不用，别人用）
`target_sources(mylib INTERFACE header_only.h)`

- 当前 target 不编译
- 依赖者可见

📌 常用于 header-only 库

✅ **适合使用的场景**
- 文件多、模块多
- 一个 target 分多块定义
- 想按目录/功能组织 CMake
```cmake
add_executable(app main.cpp)

# 用户模块
target_sources(app PRIVATE
    src/user/User.cpp
    src/user/UserDAO.cpp
)

# 工具模块
target_sources(app PRIVATE
    utils/PasswordUtils.cpp
    utils/Logger.cpp
)

# 数据库模块
target_sources(app PRIVATE
    db/Database.cpp
)

```

为了描述一组头文件，我们将使用所谓的 `FILE_SET`。
```cmake
target_sources(MyLibrary
  PRIVATE
    library_implementation.cxx

  PUBLIC
    FILE_SET myHeaders
    TYPE HEADERS
    BASE_DIRS
      include
    FILES
      include/library_header.h
)
```
范围关键字后面是一个 `FILE_SET`，即一组文件，需描述为一个单元。`FILE_SET` 由以下部分组成：
- `FILE_SET<name>` 是该 `FILE_SET` 的名称，==如果 `FILE_SET` 名与类型相同，则无需提供 `TYPE` 字段==
- `TYPE <type>` 就是我们描述的文件类型。最常见的是头部
- `BASE_DIRS` 表示头文件的“根目录”是，排除文件集的`BASE_DIRS`后，`BASE_DIRS` 默认使用当前的源目录。
- `FILES` 是文件列表，与之前的实现源代码列表相同
### target_link_libraries()用于指定一个 target 在链接阶段依赖哪些库，并同时传递这些依赖的使用要求
```cmake
target_link_libraries(
    目标名
    <作用域>
        源文件1
        源文件2
)
```

### add_subdirectory()用来把一个子目录中的 `CMakeLists.txt` 纳入当前构建系统
以下面为例
```text
MyProject/
├─ CMakeLists.txt        # 顶层
├─ src/
│  ├─ CMakeLists.txt
│  ├─ main.cpp
│  ├─ user/
│  │  ├─ CMakeLists.txt
│  │  ├─ User.cpp
│  │  └─ UserDAO.cpp
│  └─ utils/
│     ├─ CMakeLists.txt
│     ├─ PasswordUtils.cpp
│     └─ PasswordUtils.h
```
- 顶层 CMakeLists.txt 怎么写
```cmake
cmake_minimum_required(VERSION 3.23)
project(BookSystem)

add_subdirectory(src)

```
顶层只做三件事：
- 定最低版本
- 定项目名
- 拉子目录

- src/CMakeLists.txt
```cmake
add_subdirectory(utils)
add_subdirectory(user)

add_executable(BookSystem main.cpp)
target_link_libraries(BookSystem PRIVATE User)
```

- utils/CMakeLists.txt
```cmake
add_library(Utils STATIC)

target_sources(Utils
  PRIVATE
    PasswordUtils.cpp
  PUBLIC
    FILE_SET headers
    TYPE HEADERS
    BASE_DIRS .
    FILES
      PasswordUtils.h
)
```

- user/CMakeLists.txt
```cmake
add_library(User STATIC)

target_sources(User
  PRIVATE
    User.cpp
    UserDAO.cpp
)

target_link_libraries(User
  PRIVATE
    Utils
    sqlite3
)
```
==注意事项==
1️⃣：**子目录不能“反向依赖”父目录**
2️⃣：顺序很重要，如果：`user` 依赖 `utils`，那 **utils 一定要在 user 前面**
3️⃣：子目录里不要写 project()

### CMake语言基础
CMakeLang 中唯一的基础类型是字符串和列表。CMake 中的每个对象都是字符串，列表本身是包含分号作为分隔符的字符串

- `set()`用来定义或修改一个 CMake 变量
  基本用法：
```cmake
set(VAR value)  
```
  变量的值可以通过大括号展开访问，例如我们想用[[message()]] 命令打印由 `var` 命名的字符串。
```cmake
set(var "World!")
message("Hello ${var}")
```

- `foreach()`遍历列表
```cmake
set(stooges "Moe;Larry")
list(APPEND stooges "Curly")

message("Stooges contains: ${stooges}")

foreach(stooge IN LISTS stooges)
  message("Hello, ${stooge}")
endforeach()
```
  
