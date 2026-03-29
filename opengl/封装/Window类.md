封装以下内容到Window类中
1. **加载库**：初始化 GLFW (`glfwInit`)
    
2. **设置上下文**：通过 `glfwWindowHint` 告知系统 OpenGL 版本 (如 3.3/4.6) 和模式 (Core Profile)
    
3. **创建窗口**：`glfwCreateWindow` 获取窗口句柄
    
4. **激活环境**：`glfwMakeContextCurrent` 让当前线程接管窗口
    
5. **加载指针**：调用 GLAD (`gladLoadGLLoader`)，映射显卡驱动函数
    
6. **注册回调**：绑定窗口缩放、鼠标键盘等事件函数

- `Window.h`
```cpp
#ifndef OPENGL_PROJECT_WINDOW_H  
#define OPENGL_PROJECT_WINDOW_H  
#include <glad/glad.h>  
#include <GLFW/glfw3.h>  
#include <string>  
#include <iostream>  
  
class Window {  
    GLFWwindow* window;  
    static void FramebufferSizeCallback(GLFWwindow* window, int width, int height);  
  
public:  
    Window(int width, int height, const std::string& title);  
    ~Window();  
    void processInput() const;  
    [[nodiscard]] bool shouldClose() const;  
    void swapBuffers() const;  
};  
  
  
#endif //OPENGL_PROJECT_WINDOW_H
```
- `Window.cpp`
```cpp
#include "Window.h"  
  
Window::Window(int width, int height, const std::string& title) {  
    if (!glfwInit()) {  
        window =nullptr;  
        throw std::runtime_error("Failed to initialize GLFW");  
    }  
  
    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);  
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);  
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);  
  
    window = glfwCreateWindow(width, height, title.c_str(), NULL, NULL);  
    if (!window) {  
        glfwTerminate();  
        throw std::runtime_error("Failed to create GLFW window");  
    }  
  
    glfwMakeContextCurrent(window);  
    glfwSetFramebufferSizeCallback(window, FramebufferSizeCallback);  
  
    if (!gladLoadGLLoader((GLADloadproc)glfwGetProcAddress)) {  
        throw std::runtime_error("Failed to initialize GLAD");  
    }  
  
    glViewport(0, 0, width, height);  
}  
  
Window::~Window() {  
    glfwTerminate();  
}  
  
void Window::processInput() const {  
    if(glfwGetKey(window, GLFW_KEY_ESCAPE) == GLFW_PRESS)  
        glfwSetWindowShouldClose(window, true);  
}  
  
bool Window::shouldClose() const {  
    return glfwWindowShouldClose(window);  
}  
  
void Window::swapBuffers() const {  
    glfwSwapBuffers(window);  
}  
  
void Window::FramebufferSizeCallback(GLFWwindow* window, int width, int height) {  
    glViewport(0, 0, width, height);  
}
```
