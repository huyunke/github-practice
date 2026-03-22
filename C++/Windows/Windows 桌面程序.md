#### WinMain
每个 Windows 桌面应用程序也必须具有 `WinMain` 函数
```cpp
int WINAPI WinMain(
   _In_     HINSTANCE hInstance,      // 当前实例句柄
   _In_opt_ HINSTANCE hPrevInstance,  // 以前的实例句柄（废弃），但必须带着
   _In_     LPSTR     lpCmdLine,      // 命令行参数
   _In_     int       nCmdShow        // 显示状态指令
);
```
通常在 `WinMain` 函数体内部，你会进行以下三个标准步骤：

1. **注册窗口类** (`RegisterClassEx`)
    
2. **创建窗口** (`CreateWindowEx`)
    
3. **启动消息循环** (一个 `while` 循环，不断检查并分发 Windows 消息)
除此之外还需要在文件顶部列出下列代码
```cpp
#include <windows.h>
#include <tchar.h>
```

#### 消息循环
- Windows 是一个**消息驱动**的系统。每当你移动鼠标、按下一个键，系统都会生成一个“消息”并丢进程序的**消息队列**中。

- 消息循环的作用就是：**不停地从队列里抓出消息，然后交给对应的窗口处理。**
```cpp
MSG msg;
// GetMessage 从队列中取出消息
while (GetMessage(&msg, NULL, 0, 0)) 
{
    TranslateMessage(&msg); // 1. 翻译消息（如：将按键码转为字符消息）
    DispatchMessage(&msg);  // 2. 分派消息给对应的 `WndProc` 函数去处理
}
```
- [MSG](MSG.md)
- [DispatchMessage](DispatchMessage.md)
- `case WM_DESTROY:` 中，通常需要调用`PostQuitMessage(0)`来让 `GetMessage` 返回 0 从而结束循环（[PostQuitMessage](PostQuitMessage.md)）

- 在一个标准的 Win32 程序中，通常所有窗口都共享**同一个消息循环**（即 `WinMain` 里的那个 `while` 循环），如果一个程序有多个窗口，关闭其中一个时调用 PostQuitMessage 会导致其他窗口也关闭，如果你希望用户关掉一个设置面板或子窗口时，主程序依然运行，你只需要在那个子窗口的 `WM_DESTROY` 里**不调用** `PostQuitMessage` 即可。这样窗口会被销毁，但消息循环依然在为其他窗口服务（[多窗口环境下的程序退出逻辑](多窗口环境下的程序退出逻辑.md)）
#### Windows 程序的“生命周期”

| **阶段** | **核心组件**          | **动作**                            |
| ------ | ----------------- | --------------------------------- |
| **1**  | `WinMain`         | 初始化，注册并创建窗口                       |
| **2**  | **消息循环**          | `GetMessage` 不断从队列里捞东西            |
| **3**  | `WndProc`         | 根据 `switch-case` 决定如何响应（点火、绘图、按键） |
| **4**  | `DefWindowProc`   | 你不想管的事情，通通丢给这个“管家”                |
| **5**  | `PostQuitMessage` | 优雅地告诉循环：“戏演完了，可以散场了”              |
