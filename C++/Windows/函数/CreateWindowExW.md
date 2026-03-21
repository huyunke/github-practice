```cpp
HWND CreateWindowExW(
  [in]           DWORD     dwExStyle,//创建的窗口的扩展窗口样式
  [in, optional] LPCWSTR   lpClassName,
  [in, optional] LPCWSTR   lpWindowName,
  [in]           DWORD     dwStyle,//窗口的样式
  [in]           int       X,
  [in]           int       Y,
  [in]           int       nWidth,
  [in]           int       nHeight,
  [in, optional] HWND      hWndParent,//创建的窗口的父窗口或所有者窗口的句柄
  [in, optional] HMENU     hMenu,
  [in, optional] HINSTANCE hInstance,
  [in, optional] LPVOID    lpParam
);
```

类型：**HWND**

正在创建的窗口的父窗口或所有者窗口的句柄。 若要创建子窗口或拥有的窗口，请提供有效的窗口句柄。 对于弹出窗口，此参数是可选的。

若要创建 [仅消息窗口](https://learn.microsoft.com/zh-cn/windows/desktop/winmsg/window-features)，请向现有仅消息窗口提供 **HWND_MESSAGE** 或句柄。