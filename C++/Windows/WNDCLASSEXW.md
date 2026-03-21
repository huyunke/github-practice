```cpp
WNDCLASSEX wcex;

wcex.cbSize         = sizeof(WNDCLASSEX); //结构大小
wcex.style          = CS_HREDRAW | CS_VREDRAW; //类样式的任意组合
wcex.lpfnWndProc    = WndProc;
wcex.cbClsExtra     = 0;
wcex.cbWndExtra     = 0;
wcex.hInstance      = hInstance; //句柄
wcex.hIcon          = LoadIcon(wcex.hInstance, IDI_APPLICATION); //类图标的句柄
wcex.hCursor        = LoadCursor(NULL, IDC_ARROW); //类游标的句柄
wcex.hbrBackground  = (HBRUSH)(COLOR_WINDOW+1); //类背景画笔的句柄
wcex.lpszMenuName   = NULL;
wcex.lpszClassName  = szWindowClass;
wcex.hIconSm        = LoadIcon(wcex.hInstance, IDI_APPLICATION);
```
[类样式](https://learn.microsoft.com/zh-cn/windows/win32/winmsg/window-class-styles)
