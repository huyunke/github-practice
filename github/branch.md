#### branch命名规范
- **前缀说明类型**：
    
    - `feature/` → 新功能
        
    - `fix/` → 修复 bug
        
    - `hotfix/` → 紧急修复
        
    - `refactor/` → 重构
        
- **用短横线 `-` 连接单词**：
    
    - 可读性好，例如：`feature/add-login`
        
- **简洁但明确**：
    
    - 不要太长，但要表达功能意图

#### 把 `main` 的最新内容同步到功能分支
##### 方法 1：在功能分支上合并 main（Merge）
1. 切换到你的功能分支
```bash
git checkout feature/book-issue-report
```
2. 从 main 拉取最新更新
```bash
git fetch origin
```
`origin` 是你远程仓库的默认名字，如果你的 main 在别的远程名下，请替换，可以通过`git remote -v`命令来查看你的远程仓库名字
输出示例
```text
origin  https://github.com/你的用户名/仓库名.git (fetch)
origin  https://github.com/你的用户名/仓库名.git (push)
upstream  https://github.com/原作者/仓库名.git (fetch)
upstream  https://github.com/原作者/仓库名.git (push)
```
- 左边是远程仓库名字：`origin`、`upstream` 等
    
- 右边是远程仓库的 URL
    
- `(fetch)` 表示拉取，`(push)` 表示推送
通常情况：
- **`origin`** → 你自己 GitHub 账号里的仓库
    
- **`upstream`** → 你 Fork 的原始仓库
3. 合并 main 到你的分支
```bash
git merge origin/main
```

- 这个操作会把 main 的更新合并到你的功能分支
    
- 如果出现冲突，Git 会提示你解决冲突
    
- 合并完成后，你的分支就包含了 main 的最新内容

##### 方法 2：在功能分支上使用 Rebase（变基）
变基会把你的分支“移动到最新 main 的基础上”，提交历史更整洁
1. 切换到你的功能分支
```bash
git checkout feature/book-issue-report 
```
2. 拉取最新 main(前提是该分支无changes，如果有可以先暂存
3. ，等拉取最新的代码之后再恢复)==这一步只是把远程最新内容拉到本地，不会自动合并==
```bash
git fetch origin
```
3. 在功能分支上变基 main
```bash
git rebase origin/main
```
如果有冲突，解决冲突后使用
```bash
git rebase --continue
```
变基完成后，你的功能分支就是最新的 main 基础加上你自己的提交

💡 小技巧：

- 在个人功能分支上，**rebase 更常用**，提交记录干净
    
- 如果你的功能分支已经推送到远程，而且别人可能基于你的分支做了修改，最好用 **merge**，避免破坏远程历史
- 未提交的修改： 可以用 `git stash` 暂存：
```bash
git stash  
git rebase origin/main  
git stash pop
```

#### 其他
![[8bf8044bddd053c138b08040ed0d2f67.png]]
当前位置为main分支，点击想要切换的分支然后点击签出即可跳转至该分支