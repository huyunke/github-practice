通过在项目里 **配置 Git 忽略文件**，这样一些文件夹或文件就不会出现在 **GitHub Desktop 的 Changes 列表**，也不会被提交到 Git 仓库
#### **1️⃣ 基本步骤**
1. 在项目根目录创建一个 `.gitignore` 文件（如果还没有的话）

2. 在 `.gitignore` 中写入要忽略的文件或文件夹

3. 提交 .gitignore 文件  
```bash  
git add .gitignore
git commit -m "Add .gitignore to exclude utils, sqlite3.h and cryptopp"
```  
  
3. 如果文件已经被跟踪 （以utils/和sqlite3.h 为例）**如果某个文件已经被 Git 跟踪（已经提交过），即使添加到 `.gitignore` 也不会自动忽略**
```bash  
# 1.从跟踪中移除  
# 移除单个文件  
git rm --cached 文件名  
  
# 移除整个文件夹  
git rm -r --cached 文件夹名/
  
# 2.提交更改，不提交.gitignore 也不会生效，GitHub Desktop 仍然会显示它们在 Changes 列表中
git commit --amend -m "Remove utils and sqlite3.h from Git tracking"  

# 如果已经推送，需要强制推送（谨慎使用）  
git push --force 
```  

4. 提交其他修改  

**注意**：  
- 可以直接在 CLion 终端里输入 Git 命令
---  
  
## 📊 .gitignore 规则说明  
  
| 规则      | 说明          | 示例                                                  |
| ------- | ----------- | --------------------------------------------------- |
| `文件名`   | 忽略特定文件      | `sqlite3.h`                                         |
| `文件夹/`  | 忽略整个文件夹     | `utils/`                                            |
| `*.扩展名` | 忽略所有该扩展名的文件 | `*.exe`                                             |
| `路径/*`  | 忽略路径下所有文件   | `cmake-build-*/*` # 忽略所有`cmake-build-` 开头的文件夹下的所有文件 |
| `!文件名`  | 不忽略（例外）     | `!important.exe`                                    |
| `#`     | 注释          | `# 这是注释`                                            |
  
---  
  
## ✨ 验证 .gitignore 是否生效  
  
### 方法1：使用 git status  
```bash  
git status
```  
如果文件没有出现在列表中，说明已被忽略。  
  
### 方法2：使用 git check-ignore  
```bash  
# 检查某个文件是否被忽略  
git check-ignore -v utils/PasswordUtils.h  
  
# 如果输出类似下面的内容，说明被忽略了：  
# .gitignore:1:utils/    utils/PasswordUtils.h  
```  
  
### 方法3：在 GitHub Desktop 中查看  
打开 GitHub Desktop，查看 "Changes" 标签页：  
- 被忽略的文件不会出现在列表中  
- 已修改的文件会显示为 "Modified"  
  
---  
### 其他
.idea/ 文件夹建议忽略。这是 CLion 的配置文件，每个人的配置可能不同
  