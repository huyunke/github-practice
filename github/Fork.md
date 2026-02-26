#### 什么是Fork
**Fork** 是 GitHub 提供的功能，用于 **在自己的账户下复制一个别人仓库的完整副本**。

特点：

- Fork 后，你拥有 **独立于原仓库的仓库**，可以自由修改，不会影响原仓库。
    
- 适用于：
    
    - 贡献开源项目（先 Fork，再修改，再提交 Pull Request）。
        
    - 保留原仓库的版本备份

- **先 Fork** → 在自己账户有副本

- **再 Clone** → 下载到本地开发

#### Fork使用流程
- **Fork 仓库**
    
    - 在 GitHub 上打开目标仓库 → 点击右上角的 **Fork**。
        
    - 这时你账号下就有了该仓库的完整副本。
        
- **Clone 到本地**
```bash
git clone https://github.com/你的用户名/仓库名.git  
cd 仓库名    
```

    
- **添加上游仓库（upstream）**保证你的 Fork 可以同步原仓库更新：
```bash
git remote add upstream https://github.com/原作者用户名/仓库名.git  
git fetch upstream  
git merge upstream/main  # 或 master        
```
    
- **创建新分支修改**(可有可无)
```bash
git checkout -b feature/修改功能 
```
- **Push 到自己的 Fork**
    
```bash
git push origin feature/修改功能
```
    
- **提交 Pull Request**
    
    - 在 GitHub 上点击 “Compare & Pull Request”，将你的修改请求合并到原仓库。