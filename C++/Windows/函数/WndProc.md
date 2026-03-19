回调函数，由 Windows 操作系统在发生某些事情（比如点击鼠标、调整窗口大小）时调用的
```cpp
LRESULT CALLBACK WndProc(
   _In_ HWND   hWnd,      // 窗口句柄
   _In_ UINT   message,   // 消息 ID
   _In_ WPARAM wParam,    // 附加参数 1
   _In_ LPARAM lParam     // 附加参数 2
);
```
- 在此函数中编写代码，以处理发生事件时应用程序从 Windows 收到的消息，此过程称为处理事件。 你只需处理与应用程序相关的事件

- 参数解析

|**参数名**|**意义**|**类比**|
|---|---|---|
|**`hWnd`**|**窗口句柄**。指向接收消息的那个具体窗口。|就像收件人的地址，告诉函数是哪个窗口在“说话”。|
|**`message`**|**消息类型**。是一个整数宏，代表发生了什么事。|比如 `WM_PAINT`（需要重画）、`WM_LBUTTONDOWN`（左键按下）、`WM_DESTROY`（窗口关闭）。|
|**`wParam`**|**字参数**。提供消息的额外信息。|比如按键消息中，它代表哪个键被按下了（如 `VK_RETURN`）。|
|**`lParam`**|**长参数**。提供更详细的附加信息。|比如鼠标点击消息中，它包含了鼠标点击的 **X, Y 坐标**。|
一个标准的`WndProc`
```cpp
LRESULT CALLBACK WndProc(HWND hWnd, UINT message, WPARAM wParam, LPARAM lParam)
{
    switch (message)
    {
    case WM_PAINT:
        // 在这里画画，显示文字
        break;
    case WM_LBUTTONDOWN:
        // 弹出个对话框说“你点了我！”
        break;
    case WM_DESTROY:
        PostQuitMessage(0); // 告诉系统：我要彻底退出了
        break;
    default:
        // 重要！你不关心的消息，必须交给系统默认处理
        return DefWindowProc(hWnd, message, wParam, lParam);
    }
    return 0;
}
```
