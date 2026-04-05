## **指南说明** 
- 背景：针对本地算力受限（运行慢、内存占用高）的痛点，通过 PyCharm 远程连接云 GPU 能够显著提升模型开发与调试效率
- 目标：本文详述了 PyCharm 远程开发环境配置的全流程，基于AutoDL平台
- **注：** 本文档基于笔者初次实践经验编写，旨在分享交流，欢迎纠错指正
- 如果对内容有疑惑的可以看这个[视频](https://www.bilibili.com/video/BV1s2YdzNESZ/?spm_id_from=333.337.search-card.all.click&vd_source=26b9e89650adcd86e8b59e72b59454b8)

## AutoDL
1. 在[AutoDL](https://www.autodl.com/)上注册账号，然后进行充值，之后在算力市场中根据自己模型的需求挑选
![](../图片/Pasted%20image%2020260405223516.png)
2. 选择镜像后，点击`创建并开机`
![](../图片/Pasted%20image%2020260405223832.png)
3. 然后关机
![](../图片/Pasted%20image%2020260405224311.png)
4. 选择`无卡模式开机`
![](../图片/Pasted%20image%2020260405224123.png)
5. 选择`JupyterLab`，然后进入`autodl-tmp`文件夹，通过[FileZilla](FileZilla.md)（选择client）来将项目代码上传到`autodl-tmp`文件夹
6. 配置环境，进入`autodl-tmp`文件夹，使用`conda create -n your_env_name python=3.10 -y`创建环境，如果你需要其他python版本，只需把3.10替换成你需要的python版本，然后通过`conda activate your_env_name`激活环境，然后关掉终端，打开一个新终端输入`conda init`，然后`conda activate your_env_name`激活环境

## Pycharm
现在打开pycharm的设置，找到解释器
1. 选择添加解释器，选择`基于SSH`
![](../图片/Pasted%20image%2020260405230853.png)
2. 根据`SSH登录`填写相关信息，点击`下一步`然后填写密码
![](../图片/Pasted%20image%2020260405230943.png)
3. 选择`conda`环境，现有环境选择你刚刚创建的虚拟环境的名称。然后同步文件夹选择你上传到服务器的文件夹，最后点击创建
![](../图片/Pasted%20image%2020260405231223.png)
4. 在终端找到远程服务器的终端，选择它，激活虚拟环境。之后你就可以通过这个终端安装你所需要的包了。**注意，远程服务器可能系统镜像过于精简，有很多东西都没有，需要手动安装**