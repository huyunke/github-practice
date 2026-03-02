只列出了我学习到的一些git命令

#### 初始化和基础操作
```bash
git init #新建本地仓库
```
```bash
git clone <url> #克隆远端仓库到本地
```
```bash
git status #查看当前状态
```
```bash
git commit -m "描述"  #描述要认真写
```
```bash
git push origin main #推到远端main分支
```
```bash
git pull origin main #拉取远端最新代码
```
#### 分支操作
```bash
git branch #查看所有分支
```
```bash
git checkout -b <name> #新建并切换到新分支
```
```bash
git merge <name> #把某分支合并到当前分支
```
```bash
git log --oneline #查看简洁版提交历史
```
#### 出了问题也许能用上的
```bash
git diff #查看改了什么
```
```bash
git restore <filename> #撤销未暂存的改动
```
```bash
git reset HEAD~1 #撤销最近一次的commit（保留改动）
```
