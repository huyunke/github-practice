#### 一、GitHub 语言统计机制说明

GitHub 的语言统计基于其使用的 GitHub Linguist 工具。

统计规则是：

> 根据仓库中各类文件的字节数进行统计，而不是根据项目架构或核心逻辑判断。

因此：

- 不关心哪种语言是“系统核心”
    
- 不关心后端是否主导
    
- 只看代码文件大小占比
    

如果某种语言文件较多（例如前端 HTML/CSS/JS），它就可能成为仓库主页显示的主语言。

--- 
如果你的项目：

- 使用多种语言
    
- 代码量接近
    
- 但希望体现某种语言为主体（例如 C++ / Java / Go）
    

可以通过修改 `.gitattributes` 来控制语言统计。

---
#### 二、什么是.gitattributes
`.gitattributes` 是：

> Git 属性配置文件（Git attributes file）

它用于定义 Git 如何处理特定文件。控制 Git 如何处理已追踪文件

它可以控制：

- 换行符行为
    
- diff / merge 策略
    
- 语言统计参与情况
    
- 语法高亮分类
    
- 是否当作二进制文件

#### 三、.gitattributes食用方法
##### 控制Github语言统计
- 忽略某些文件参与统计
1. 在仓库根目录创建`.gitattributes`写入：
```bash
# 忽略某类文件 *.文件后缀 linguist-detectable=false
*.html linguist-detectable=false
*.css linguist-detectable=false
*.js linguist-detectable=false
```
忽略某个目录
```bash
目录名称/** linguist-detectable=false
```
提交后，过一会 GitHub 会重新计算语言比例。

- 将某些文件标记为文档（不参与统计）
```bash
文件名 linguist-documentation=true
目录名称/** linguist-documentation=true
*.文件后缀 linguist-documentation=true
```
常用于：`Markdown,示例代码,文档文件`

- 强制指定某种语言类型
`.h` 可能被识别成 C 或 C++
通过在.gitattributes添加以下这句话可以将`.h`强制识别为C++
```bash
*.h linguist-language=C++
```

##### 统一换行符
```bash
* text=auto
*.cpp text
*.h text
*.sh text eol=lf
*.bat text eol=crlf
```
意思：
- 文本文件要统一换行
    
- C++ 是文本
    
- shell 必须 LF
    
- bat 必须 CRLF
作用：

- Windows / Linux 换行符统一，`.bat` 是 Windows 批处理文件必须用 CRLF 换行。`.sh` 是 Linux shell 脚本，必须使用 LF 换行符
    
- 避免无意义的 diff
