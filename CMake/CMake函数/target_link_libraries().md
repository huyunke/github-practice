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