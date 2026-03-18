##### 启用内存泄漏检测
可以通过启用调试堆的所有函数来检测内存泄漏，按以下顺序包含以下语句：
```cpp
#define _CRTDBG_MAP_ALLOC //显示首次分配泄漏的内存的文件名和行号
#include <stdlib.h>
#include <crtdbg.h>
```
- 之后在应用出口点之前放置`_CrtDumpMemoryLeaks()`来在应用退出时显示内存泄漏报告
- 无论是否定义了 `_CRTDBG_MAP_ALLOC`，内存泄漏报告都会显示以下信息：
```bash
Detected memory leaks!
Dumping objects ->
c:\users\username\documents\projects\leaktest\leaktest.cpp(20) : {18}
normal block at 0x00780E80, 64 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD
Object dump complete.
```
    - 内存分配编号，在示例中为 `18`
    - 块类型，在示例中为 `normal` 。
    - 十六进制内存位置，在示例中为 `0x00780E80`。
    - 块的大小，在示例中为 `64 bytes`。
    - 块中前 16 个字节的数据（十六进制形式）。

##### `_CrtDumpMemoryLeaks()`
- 默认情况下，`_CrtDumpMemoryLeaks` 将内存泄漏报告输出到输出窗口的调试窗格中
- 如果使用库，该库可能会将输出重置到另一位置，具体的例子：

假设你在做一个图形处理软件，你引用了一个叫 `MagicImage` 的第三方库：

1. **你的代码**（在 `main` 函数开头）： 你很守规矩地设置了：“把内存泄漏报告发给 **调试窗口**”
```cpp
_CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG); 
```
2. **库的代码**（在你调用 `MagicImage::Init()` 时）： 这个库的开发者为了方便他自己排查问题，在库的初始化函数里偷偷写了这么一行，要把报告发给 **控制台（黑窗口）** 或者一个 **日志文件**：
```cpp
// 这个库的开发者想在黑窗口看报告
 _CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_FILE); 
_CrtSetReportFile(_CRT_WARN, _CRTDBG_FILE_STDOUT); 
```
3. **结果**： 虽然你的 `main` 函数里开启了内存检测，但当程序结束时，你盯着 Visual Studio 的“输出”窗口看半天也**什么都没有**。报告其实被那个库给“劫持”了，发到了控制台（或者文件）里。

如果你发现报告没出现在该出现的地方，你可以尝试在 `main` 函数的**最后一行**（或者在程序结束前）再次强制调用一次 `_CrtSetReportMode`，把所有权“抢”回来：
```cpp
int main() {
    _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);

    // ... 调用各种库的代码 ...

    // 重点：在程序即将结束前，再次确认报告的目的地
    // 这样即使中间被库改了，最后这一刻也能被你纠正回来
    _CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG);
    
    return 0;
}
```
在一个复杂的项目中，多个模块可能都在争夺报告的输出权。如果你看不到报告，多半是输出位置被别人中途改掉了


##### `_CrtSetDbgFlag`
- 如果有多个出口点，可以通过调用
```cpp
_CrtSetDbgFlag ( _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF );
```
来实现，`_CrtSetDbgFlag` 决定要不要检查
- **`_CRTDBG_ALLOC_MEM_DF`**： 这是“**开启监控**”开关。它告诉调试堆（Debug Heap）：“从现在起，每一次 `new` 或 `malloc` 分配内存，你都要在小本本上记录下来，包括分配了多少、地址在哪。”
    
- **`_CRTDBG_LEAK_CHECK_DF`**： 这是“**自动报告**”开关。它告诉程序：“当你运行结束、准备退出的时候，自动调用一次 `_CrtDumpMemoryLeaks()`。如果发现小本本上还有没被释放的内存，就把清单打印在输出窗口里。”

##### `_CrtSetReportMode`
- **`_CrtSetReportMode` 决定**检查结果往哪儿发
- `_CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_DEBUG );`
    - - **`_CRT_WARN`**：报告的类型。内存泄漏在 CRT 中被归类为“警告”。
    
    - **`_CRTDBG_MODE_DEBUG`**：报告的去向。它指向 Visual Studio 底部的 **“输出 (Output)”** 窗口。


##### 生成更有用的内存泄漏报告
```cpp
#ifdef _DEBUG
    #define DBG_NEW new ( _NORMAL_BLOCK , __FILE__ , __LINE__ )
    // Replace _NORMAL_BLOCK with _CLIENT_BLOCK if you want the
    // allocations to be of _CLIENT_BLOCK type
#else
    #define DBG_NEW new
#endif
```


##### 1. 内存 Dump 比对 (Memory Snapshot/Diff)

**用途**：当你怀疑程序在**某一段特定的代码执行期间**（比如一个循环或一个按钮点击事件）产生了内存泄漏时，可以用它。

**原理**：在代码的起始点拍一张内存“快照”，在结束点拍第二张，然后让编译器比对这两张快照之间的**差异**。

**代码实现：**

```cpp
_CrtMemState s1, s2, s3;

// 1. 记录起始点的内存状态
_CrtMemCheckpoint(&s1);

// --- 执行你怀疑有问题的代码 ---
...

// 2. 记录结束点的内存状态
_CrtMemCheckpoint(&s2);

// 3. 比对差异。如果发现 s2 比 s1 多了内存没释放，s3 会记录下差异
if (_CrtMemDifference(&s3, &s1, &s2)) {
    _CrtMemDumpStatistics(&s3); // 把对比结果（泄漏了多少字节）打印出来
}
```

**掌握点**：它能帮你精确定位泄漏发生的**时间段**。

---

##### 2. 分配 Break (Setting a Breakpoint on an Allocation)

**用途**：当你从报告中得知是第 `{75}` 次分配漏了，但对应的行号在你的代码里被调用了几万次（比如在 `vector` 的构造函数里），你无法直接打断点时，就用这一招。

**原理**：CRT 内部有一个全局计数器，每分配一次内存就加 1。这行代码告诉程序：“当计数器数到 **75** 的时候，无论你在哪，立刻触发断点停下来！”

**代码实现：**

```cpp
// 必须在分配动作发生之前调用
// 通常写在 main 函数的最开头
_CrtSetBreakAlloc(75); 
```

**掌握步骤：**

1. 运行程序，获得内存泄漏报告。
    
2. 在输出窗口找到大括号里的数字（例如 `{75}`）。
    
3. 将该数字填入 `_CrtSetBreakAlloc(75)`。
    
4. 重新开始调试，程序会自动在那个分配动作发生的一瞬间**断住**。
    
5. 查看 **“调用堆栈 (Call Stack)”**，你就能看清楚是谁、在哪层逻辑里发起了这次分配。
    

---

|**技术**|**解决的问题**|**核心工具**|
|---|---|---|
|**内存 Dump 比对**|确定**哪一段逻辑**导致了内存增长。|`_CrtMemCheckpoint`|
|**分配 Break**|精确定位**哪一次特定的分配**导致了泄漏。|`_CrtSetBreakAlloc`|

#### 试验
```cpp
#define _CRTDBG_MAP_ALLOC  
#include <stdlib.h>  
#include <crtdbg.h>  
#include <iostream>  
  
// Simulating a function with a memory leak  
void ProcessData(int index) {  
    int* data = new int[5];
  
    // Logic error: Forget to delete when index is 10    
    if (index != 10) {  
        delete[] data;  
    }  
}  
  
int main() {  
    // PART 1: Environment Setup
    // Direct reports to the Visual Studio Output window    
    _CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG);  
  
    // Enable automatic leak check at program exit  
    _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);  
  
    // PART 2: Precise Targeting (Optional)
    // Uncomment and replace with the allocation number from your previous run    
    // _CrtSetBreakAlloc(156);  //填写花括号里面的数字，不一定是156
    std::cout << "Starting Memory Snapshot Comparison" << std::endl;  
  
    // PART 3: Memory Dump Comparison (The "Clamp" Technique)
    _CrtMemState s1, s2, s3;  
  
    // Take Snapshot A (Before the suspected logic)  
    _CrtMemCheckpoint(&s1);  
  
    for (int i = 0; i < 20; ++i) {  
        ProcessData(i);  
    }  
  
    // Take Snapshot B (After the suspected logic)  
    _CrtMemCheckpoint(&s2);  
  
    // Compare the two snapshots  
    if (_CrtMemDifference(&s3, &s1, &s2)) {  
        std::cout << "Memory change detected! Dumping statistics..." << std::endl; 

        _CrtMemDumpStatistics(&s3);  
    } else {  
        std::cout << "No memory leak detected in this block." << std::endl;  
    }  
  
    std::cout << "Program finished. Check the 'Output' window for details" << std::endl;  
  
    return 0;  
}
```
![](97160c6b5bde0dafb4d267df93e0a938.png)
填写157到`_CrtSetBreakAlloc()`中，重新调试
![](99ef3b4abf170e0ff1882f5d55253bdc.png)