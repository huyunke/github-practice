本指南介绍了如何利用 MSVC 提供的 C 运行时库（CRT）在开发阶段识别、定位并修复内存泄漏
#### 1.启用内存泄漏检测
可以通过启用调试堆的所有函数来检测内存泄漏，在源文件（通常是 `main.cpp`）的最顶部包含以下代码。**顺序很重要**：必须在包含其他头文件之前定义宏
```cpp
#define _CRTDBG_MAP_ALLOC //显示首次分配泄漏的内存的文件名和行号
#include <stdlib.h>
#include <crtdbg.h>

//进阶配置
#ifdef _DEBUG
    #define DBG_NEW new ( _NORMAL_BLOCK , __FILE__ , __LINE__ )
    // Replace _NORMAL_BLOCK with _CLIENT_BLOCK if you want the
    // allocations to be of _CLIENT_BLOCK type
#else
    #define DBG_NEW new
#endif
```
- 之后在**应用出口点**[^1] 之前添加`_CrtDumpMemoryLeaks()` 来在应用退出时显示内存泄漏报告
- 无论是否定义了 `_CRTDBG_MAP_ALLOC`，内存泄漏报告都会显示以下信息[^2]：
```bash
Detected memory leaks!
Dumping objects ->
c:\users\username\documents\projects\leaktest\leaktest.cpp(20) : {18}
normal block at 0x00780E80, 64 bytes long.
 Data: <                > CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD CD
Object dump complete.
```

#### 2. 消息路由：决定报告发往何处
- 使用 `_CrtSetReportMode` 配置不同严重程度的消息去向：例如`_CrtSetReportMode( _CRT_WARN, _CRTDBG_MODE_DEBUG );`

- **`_CRT_WARN`**: 警告（如：内存泄漏）
    
- **`_CRT_ERROR`**: 运行时错误（如非法参数）
    
- **`_CRT_ASSERT`**: 断言失败（代码逻辑假设不成立）
    
**常用目的地：**

- `_CRTDBG_MODE_DEBUG`: 发往 IDE（Visual Studio）的“输出”窗口（默认）

###### 可能出现的问题
- 由于调试报告的设置（`_CrtSetReportMode`）是**全局有效**的，这会导致以下冲突
- 如果你的项目引用了第三方库（如 `MagicImage`）时：

- **你的意图**：你想在调试窗口看报告
```cpp
_CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG);
```
- **库的行为**：库的开发者可能为了方便自己，在初始化时偷偷更改了全局设置
```cpp
_CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_FILE);
_CrtSetReportFile(_CRT_WARN, _CRTDBG_FILE_STDOUT);
```
- **后果**：由于这些设置是**全局生效**的，库的设置覆盖了你的设置。程序结束时，调试窗口空空如也，报告被“拐”到了控制台或日志文件中

###### 解决方法
- 在一个复杂的、多模块的项目中，最好的习惯是在**程序即将退出前**，再次强制声明输出位置
```cpp
int main() {
    _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF);

    // ... 调用各种第三方库的代码 ...

    // 重点：在程序即将结束前，再次确认报告的目的地
    // 这样即使中间被库改了，最后这一刻也能被你纠正回来
    _CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG);
    
    return 0;
}
```
- 如果要追求“绝对稳妥”，那就不能只靠在 `main` 函数最后一行写代码，因为现实中的程序往往有多个 **`return`** 或者可能因为 **异常 (Exception)** 提前崩溃，最推荐的方法是**RAII 机制**：利用 C++ 对象的**析构函数**。在 `main` 函数栈上创建一个对象，当 `main` 结束（无论是正常返回还是因为报错被销毁）时，析构函数会自动执行
```cpp
struct LeakReportGuard {
    // 析构函数：对象生命周期结束时自动调用
    ~LeakReportGuard() {
        _CrtSetReportMode(_CRT_WARN, _CRTDBG_MODE_DEBUG);
        // 在main函数退出时立刻强制性执行一次内存盘点
        _CrtDumpMemoryLeaks(); 
    }
    LeakReportGuard() { 
	    // 在构造时就开启“自动检测”开关 
	    // 这样即使不手动调 Dump 函数，系统也会在程序【彻底】退出后检查（包括全局变量） 
	    _CrtSetDbgFlag(_CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF); 
	}
};

int main() {
    LeakReportGuard guard; // 只要这一行，剩下的交给 C++ 编译器
    
    // ... 无论中间有多少库在乱改设置，
    // ... 无论有多少个 return，
    // ... 甚至发生异常导致程序崩溃，
    // guard 的析构函数都会在最后时刻“拨乱反正”。
    
    return 0;
}
```
当你执行到 `return 0;` 时，`main` 函数内的所有局部变量（包括 `guard` 对象）都会按顺序销毁
- **此时调用 `_CrtDumpMemoryLeaks()`**：它会检查此时堆内存的情况
    
- **优点**：能立刻在控制台或输出窗口看到报告
    
- **缺点**：如果程序有**全局变量**或**静态变量**（它们在 `main` 之外），此时它们还没被销毁，会被误报为“泄漏”
    
在 `main` 退出后，C++ 运行时库还会做最后清理，释放全局变量

- 如果你在构造函数里设置了 `_CrtSetDbgFlag(... | _CRTDBG_LEAK_CHECK_DF)`，系统会在**这个时刻之后**自动调一次 `_CrtDumpMemoryLeaks()`
    
- **优点**：最准确，不会误报全局变量
    
- **缺点**：如果此时“报告路由”被第三方库改歪了，你可能根本看不到这份报告

#### 3. 监控策略：何时进行检查？
###### 自动监控`_CrtSetDbgFlag`
- 如果有多个出口点，可以通过调用
```cpp
_CrtSetDbgFlag ( _CRTDBG_ALLOC_MEM_DF | _CRTDBG_LEAK_CHECK_DF );
```
来实现，`_CrtSetDbgFlag` 决定要不要检查
- **`_CRTDBG_ALLOC_MEM_DF`**： 这是“**开启监控**”开关。它告诉调试堆（Debug Heap）：“从现在起，每一次 `new` 或 `malloc` 分配内存，你都要在小本本上记录下来，包括分配了多少、地址在哪。”
    
- **`_CRTDBG_LEAK_CHECK_DF`**： 这是“**自动报告**”开关。它告诉程序：“当你运行结束、准备退出的时候，自动调用一次 `_CrtDumpMemoryLeaks()`。如果发现小本本上还有没被释放的内存，就把清单打印在输出窗口里。”

###### 手动触发
在程序出口点（如 `return 0` 之前）手动调用：
```cpp
_CrtDumpMemoryLeaks();
```

#### 4. 定位
##### 1. 内存 Dump 比对 (Memory Snapshot/Diff)

**用途**：当你怀疑程序在**某一段特定的代码执行期间**（比如一个循环或一个按钮点击事件）产生了内存泄漏时，可以用它

**原理**：在代码的起始点拍一张内存“快照”，在结束点拍第二张，然后让编译器比对这两张快照之间的**差异**

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

**掌握点**：它能帮你精确定位泄漏发生的**时间段**

---

##### 2. 分配 Break (Setting a Breakpoint on an Allocation)

**用途**：当你从报告中得知是第 `{75}` 次分配漏了，但对应的行号在你的代码里被调用了几万次（比如在 `vector` 的构造函数里），你无法直接打断点时，就用这一招

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

#### 5.试验
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

[^1]: 是程序正式结束运行并退出到操作系统的最后时刻。在 C++ 程序中，最常见的出口点通常是 `main()` 函数的末尾，即 `return 0;` 发生的地方 

[^2]:      - 内存分配编号，在示例中为 `18`
    - 块类型，在示例中为 `normal` 。
    - 十六进制内存位置，在示例中为 `0x00780E80`。
    - 块的大小，在示例中为 `64 bytes`。
    - 块中前 16 个字节的数据（十六进制形式）。

[^3]: - **`_CRT_WARN`**: 这是**报告类型**。在内存泄漏检测中，所有的泄漏信息都被归类为“警告（Warning）”
	- **`_CRTDBG_MODE_DEBUG`**: 这是**显示目的地**。它指定将信息发送到 **“输出（Output）”窗口**

