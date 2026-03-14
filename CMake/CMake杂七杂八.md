这里是CMake杂七杂八，放着我从不同地方了解到的关于CMake零碎的知识（过于零碎，以至于不知道归为哪一类）

- 在CMake中，**变量名是区分大小写的**，但**命令（函数名）不区分大小写**
- **注释**
    - 注释行：`CMake`使用`#`进行行注释，可以放在任何位置
    - 注释块：：`CMake`使用`#[[]#]`进行块注释
- `add_library`等那些添加编译目标的，目标名称不可以重名**但生成的文件名可以一样**，有一个技巧是：**target 不同，但输出名字一样。**
例如：
```cmake
add_library(MathStatic STATIC src/Mathlib.cpp)  
#修改目标 `MathStatic` 的属性，让它生成的文件名变成 `MathLib`
set_target_properties(MathStatic PROPERTIES OUTPUT_NAME MathLib)  
  
add_library(MathDynamic SHARED src/Mathlib.cpp)  
#修改目标 `MathDynamic` 的属性，让它生成的文件名变成 `MathLib`
set_target_properties(MathDynamic PROPERTIES OUTPUT_NAME MathLib)
```

这样生成的文件是：
```text
MathLib.lib      (静态库)  
  
MathLib.dll  
MathLib.lib      (动态库 import lib)
```

但 CMake 内部 target 仍然是：
```cmake
MathStatic  
MathDynamic
```
- `add_library()`可以不写头文件，因为头文件不会被编译，IDE写 header 只是为了组织文件不是为了编译，不过既然添加新类的时候`Clion`会自动在`CMakelists.txt`中添加头文件和`cpp`文件，关于这点只需要知道有这么个事就行