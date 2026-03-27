### 代码
在调用任何OpenGL的函数之前我们需要初始化GLAD
```cpp
if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) { 
	std::cout << "Failed to initialize GLAD" << std::endl; 
	return -1; 
}
```
我们给GLAD传入了用来加载系统相关的OpenGL函数指针地址的函数。GLFW给我们的是`glfwGetProcAddress`，它根据我们编译的系统定义了正确的函数
- 作用：**找到显卡驱动中函数的具体地址，并把它们赋值给 GLAD 的函数指针，让你在代码里能正常调用 `glDrawArrays` 之类的 OpenGL 命令。**

### 放置位置
1. **初始化 GLFW** (`glfwInit`)

2. **创建窗口** (`glfwCreateWindow`)
    
3. **将窗口设为当前上下文** (`glfwMakeContextCurrent`)
    
4. **初始化 GLAD**
    
> **注意：** 如果你在创建窗口并设置上下文之前运行这段代码，GLAD 会因为找不到“环境”而初始化失败。