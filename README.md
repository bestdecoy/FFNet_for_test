此项目依托原项目[FFNet](https://github.com/haibao-yu/FFNet-VIC3D)构建，详细README请参考原Repos。

本项目对原项目的Dockerfile进行了一定的修改，以便可以成功构建。

Check the modified Dockerfile [here](https://github.com/bestdecoy/FFNet_for_test/tree/main/docker).

---

FFNet相关资源：
- Paper：https://arxiv.org/abs/2303.10552
- Repos: https://github.com/haibao-yu/FFNet-VIC3D

本文介绍的方法主要是通过docker配置image便于在容器中启动FFNet的环境配置，免收环境配置之苦。


本文实验环境简介：
- 操作系统：Win11 x86
- 虚拟化工具：[Docker Desktop for Windows](https://docs.docker.com/desktop/install/windows-install/)
    - 在使用Docker以前请提前打开WSL2和主板BIOS中CPU设置中的虚拟化。
    - CPU虚拟化设置：  
    在BIOS中开启，例如以AMD平台上的SVM。以华硕BIOS为例，找到“高级”->“CPU设置”->“SVM”选择开启，F10保存重启；
    - WSL2:   
    在MS Store中搜索“Windows Subsystem for Linux”并安装。如果有旧版WSL，需要在PowerShell中设置默认WSL2:  
    `wsl --set-default-version 2`

    > Docker使用的Linux发行版内核来自于WSL2，WSL需要虚拟化支持。

# 实战：
完成了前期准备，就可以开始进行实战了，下面是简要流程。
## 1. 克隆项目
cd到项目地址后，直接克隆：  
`git clone https://github.com/haibao-yu/FFNet-VIC3D.git`

## 2. 制作image
在项目地址的：/docker下有作者写好的Dockerfile，但在实际build中存在大量问题，因此对Dockerfile进行一定的修改。  
笔者将修改后的Dockerfile修改到笔者的仓库，可以在[这里](https://github.com/bestdecoy/FFNet_for_test/blob/main/docker/Dockerfile)获取。

1. 首先cd到项目地址中
2. 备份一份原Dockerfile：  
    `mv Dockerfile Dockerfile.bak`  
3. 下载该文件：  
    `wget https://raw.githubusercontent.com/bestdecoy/FFNet_for_test/main/docker/Dockerfile`
4. 直接构建：  
`docker build -t ffnet .`  
    > The -t flag tags your image with a name. (welcome-to-docker in this case). And the . lets Docker know where it can find the Dockerfile.  


5. 
