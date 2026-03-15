### 🛠️ 核心语法示例

假设我们有一个名为 `MyLib` 的库，目录结构如下：

- `include/MyLib/engine.h`
    
- `src/engine.cpp`
    

在 `CMakeLists.txt` 中，我们可以这样写：

```cmake
add_library(MyLib src/engine.cpp)

target_sources(MyLib
    INTERFACE
        FILE_SET HEADERS
        BASE_DIRS include
        FILES include/MyLib/engine.h
)
```

##### 这里的关键点：

- **`FILE_SET HEADERS`**：声明这是一个头文件集合。
    
- **`BASE_DIRS`**：这是最重要的部分。它定义了头文件的“根目录”。CMake 会根据这个路径自动计算出 `include` 路径。一旦这样设置，CMake 会自动为 `MyLib` 添加 `PUBLIC` 的包含目录，外部目标只需 `target_link_libraries(App MyLib)` 就能直接 `#include <MyLib/engine.h>`。

#### 用target_sources()处理多目录结构
- `FILE_SET` 通过 `BASE_DIRS` 告诉 CMake：“这是我头文件的根目录，请根据它来计算包含路径”。
假设你的项目结构非常规范：

- `include/MyLib/core.h`
    
- `include/MyLib/utils.h`
    
- `extra_inc/MyLib/special.h`
    

我们可以通过 `target_sources` 这样管理：

```cmake
target_sources(MyLib
    PRIVATE src/engine.cpp
    PUBLIC
        FILE_SET HEADERS
        BASE_DIRS include extra_inc  # ⬅️ 关键：定义逻辑根目录
        FILES 
            include/MyLib/core.h
            include/MyLib/utils.h
            extra_inc/MyLib/special.h
)
```

因为 CMake 会自动把 `include` 和 `extra_inc` 加入到目标的包含路径中。对于用户来说，他们只需要： `#include <MyLib/core.h>` 或 `#include <MyLib/special.h>`

####  自动化的安装与导出（不过无法解决ABI兼容问题）
假设你正在开发一个名为`MyLib`的库
- 库作者的视角
- 以下代码包含了：定义目标 $\rightarrow$ 绑定头文件 $\rightarrow$ 配置安装 $\rightarrow$ 生成导出说明书
```cmake
cmake_minimum_required(VERSION 3.23) # FILE_SET 需要 3.23+
project(MyLib VERSION 1.0 LANGUAGES CXX)

# --- 1. 创建目标 (Target Definition) ---
# 定义库的源文件，这里使用 PRIVATE，因为 cpp 不需要暴露给用户
add_library(MyLib src/engine.cpp)

# 假设你链接了第三方库（这是自动收集 DLL 的前提，即使用$<TARGET_RUNTIME_DLLS:target>的前提） 
# 如果有.cmake
# find_package(assimp REQUIRED) 
# target_link_libraries(MyLib PRIVATE assimp::assimp)
# 如果没有.cmake（手动下载的 `.dll` 和 `.lib`）
# add_library(assimp SHARED IMPORTED) # 声明：这是一个外部现成的库 
# set_target_properties(assimp PROPERTIES 
#	  IMPORTED_LOCATION "..." # 告诉 CMake：DLL 在这（运行时用） 
#	  IMPORTED_IMPLIB "..." # 告诉 CMake：LIB 在这（链接时用） 
#	  )

# --- 2. 绑定头文件 (The FILE_SET Magic) ---
# 这里是“剪刀”逻辑发生的地方
target_sources(MyLib
    PUBLIC
        FILE_SET HEADERS
        BASE_DIRS include extra_inc  # 定义搜索根目录
        FILES 
            include/MyLib/core.h
            include/MyLib/utils.h
            extra_inc/MyLib/special.h
)
# 💡 此时，CMake 已经自动为你做了 target_include_directories
# 内部和外部现在都能用 #include <MyLib/core.h>

# --- 3. 配置安装产物 (Install Targets) ---
# 这一步决定了 .lib/.a 文件和头文件去哪
install(TARGETS MyLib
    EXPORT MyLibTargets              # 这里的名字对应下一步的 EXPORT
    FILE_SET HEADERS DESTINATION include # 自动剪掉 BASE_DIRS，拷贝到安装目录的 include/
    ARCHIVE DESTINATION lib          # 静态库安装路径
    LIBRARY DESTINATION lib          # 动态库安装路径
    RUNTIME DESTINATION bin          # 可执行文件/DLL 安装路径
)

# --- 搬运第三方现成文件 (例如 assimp，如果没有第三方库可以跳过这个) --- 
# 自动收集 MyLib 依赖的所有第三方 DLL 并安装到 bin 
install(FILES $<TARGET_RUNTIME_DLLS:MyLib> DESTINATION bin # 把它和你的库放在同一个 bin 目录下，确保程序能跑通 
)

# --- 4. 生成并导出“说明书” (Exporting) ---
# 这一步生成 MyLibConfig.cmake，并定义“姓氏”
install(EXPORT MyLibTargets
    FILE MyLibConfig.cmake           # 最终生成的文件名
    NAMESPACE MyLib::                # 定义双冒号左边的“姓”
    DESTINATION lib/cmake/MyLib      # 说明书存放的位置
)
```
然后执行安装命令
```bash
`cmake --install . --prefix ./MySDK`。
```

- 如果没写 `EXPORT`：`MySDK` 文件夹里有 `include/`, `lib/`, `bin/`
- 如果写了 `EXPORT`：`MySDK` 文件夹里有 `include/`, `lib/`, `bin/` **外加** `lib/cmake/MyLib/`
- 最后只要把这个变成压缩包发送给对方就行了

1. **`target_sources` 里的 `BASE_DIRS` (剪刀)**： 你告诉 CMake 根目录是 `include`。
    
2. **`install` 里的 `DESTINATION include` (粘贴)**： CMake 剪掉 `include` 之后，剩下的路径是 `MyLib/core.h`。它把它贴到安装目标的 `include` 后面，最终物理路径是 `安装路径/include/MyLib/core.h`。
    
3. **`install(EXPORT ... NAMESPACE MyLib::)` (取姓)**： 它把你在第一步 `add_library` 定义的名字 `MyLib` 拿过来，拼成 `MyLib::MyLib`。
    
4. **`find_package(MyLib REQUIRED)` (找书)**： 它在磁盘上搜寻 `MyLibConfig.cmake`。一旦找到，它就知道 `MyLib::MyLib` 这个目标包含了哪些 `.h` 路径和哪个 `.lib` 文件。

5. **`install(TARGETS ...)`**：针对的是你在 CMake 里通过 `add_library` 或 `add_executable` 定义出来的目标。CMake 知道这些目标对应的文件在哪里（编译出来的 `.exe` 或 `.lib`）。

6. **`install(FILES ...)`**：针对的是**普通磁盘文件**，比如这个文件是你从网上下载的，或者是别人编译好的。CMake 并不“拥有”它，所以你必须给出**具体的路径**，强行告诉 CMake：“把这个文件拷贝到安装目录的 `bin` 文件夹里。”简单来说`FILES` 是用来搬运那些“不属于你项目管理，但运行又必须依赖”的文件。这里注意一下，此处的`FILES`和`target_sources` 中的 `FILES`不一样

- 使用者
- 需要获取这个库对应的.h，.dll，.lib，.cmake文件（如果有`install(EXPORT...)`)
```cmake
cmake_minimum_required(VERSION 3.23)
project(MyApp)

# 1. 自动读取你生成的 MyLibConfig.cmake
find_package(MyLib REQUIRED)

add_executable(MyApp main.cpp)

# 2. 这里的 MyLib::MyLib 会自动带来：
#    - include 目录
#    - extra_inc 目录
#    - MyLib 的二进制库文件
target_link_libraries(MyApp PRIVATE MyLib::MyLib)
```

如果我不生成`.cmake`，使用者想要用我这个库就需要获取这个库的`.h,.lib,.dll`，并且需要使用者需要手动配置路径，但是如果我生成`.cmake`，使用者想要使用我这个库就还需要获取`.cmake`，然后通过`find_package()`来自动配置路径（上述例子是有生成`.cmake`这个情况）
