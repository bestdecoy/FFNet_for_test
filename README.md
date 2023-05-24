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

# 环境配置：
完成了前期准备，就可以开始进行实战了，下面是简要流程。
## 1. 克隆项目

在克隆项目以前，我们先创建一个工作区，用于挂载宿主机的地址到容器上，以便后续的操作：   

```
cd ${your_workspace_dir}
mkdir ${repos_root}
````

cd到项目地址`${repos_root}`后，直接克隆：  
`git clone https://github.com/haibao-yu/FFNet-VIC3D.git`

## 2. 制作image / Pull image from dockerhub

用户可选自己通过本项目提供的Dockerfile自行制作image，或者从dockerhub上直接拉取笔者做好的镜像。

在Dockerfile中显示，本镜像创建的环境为：

```
ARG PYTORCH="1.6.0"
ARG CUDA="10.1"
ARG CUDNN="7"
```

## 对于制作image：

在项目地址的：/docker下有作者写好的Dockerfile，但在实际build中存在大量问题，因此对Dockerfile进行一定的修改。  
笔者将修改后的Dockerfile上传到笔者的仓库，可以在[这里](https://github.com/bestdecoy/FFNet_for_test/blob/main/docker/Dockerfile)获取。

1. 首先cd到项目地址中
2. 备份一份原Dockerfile：  
    `mv Dockerfile Dockerfile.bak`  
3. 下载该文件：  
    `wget https://raw.githubusercontent.com/bestdecoy/FFNet_for_test/main/docker/Dockerfile`
4. 直接构建：  
`docker build -t ffnet_img .`  
需要注意的是，使用30系以上的显卡配合较低版本的cuda并不兼容，欲下载不同版本的pytorch + cuda，请参考：https://hub.docker.com/r/pytorch/pytorch/tags
    > The -t flag tags your image with a name. (welcome-to-docker in this case). And the . lets Docker know where it can find the Dockerfile.  
 
## 对于从dockerhub pull 镜像：
无需自行构建镜像，直接从笔者构建的docker镜像下载：  
`docker pull bestdecoy/ffnet_160_101_7`

## 3. 容器操作  
构建镜像以后，我们就可以对容器进行操作。

- 查看image id：`docker images`
- 启动容器，允许GPU穿透、挂载工作区：`docker run --name ffnet -it --gpus all -v ${repos_root}:${corresponding_address_on_the_container} -d ${image-id} `(将{image-id}修改为刚刚查询到的image，或者直接运行`docker run --name ffnet -it --gpus all -d ffnet_img`)
- 进入容器：`docker exec -it ffnet bash`

进入容器的bash后，就可以对刚刚1.克隆FFNet的项目进行操作了。
- 使用cd进入项目地址：   

```
cd ${repos_root}
cd FFNet-VIC3D
```
- 安装FFNet：`pip install -e . --user`

安装FFNet后，我们查看一下GPU的相关信息：  
进入python交互模式，导入pytorch：`import torch`
- 检查cuda是否可用：torch.cuda.is_available()
- 检查cuda版本是否匹配：torch.cuda.get_device_name(0)，如果版本不匹配会报错；如果GPU兼容cuda版本，那么返回GPU的名称。
- 返回gpu索引值：torch.cuda.current_device()

最后退出py交互环境：`quit()`

注：CUDA和GPU在Pytorch上不兼容的报错提示为：

```
>>> torch.cuda.get_device_name(0)
/opt/conda/lib/python3.7/site-packages/torch/cuda/__init__.py:125: UserWarning:
NVIDIA GeForce RTX 3060 Ti with CUDA capability sm_86 is not compatible with the current PyTorch installation.
The current PyTorch install supports CUDA capabilities sm_37 sm_50 sm_60 sm_61 sm_70 sm_75 compute_37.
If you want to use the NVIDIA GeForce RTX 3060 Ti GPU with PyTorch, please check the instructions at https://pytorch.org/get-started/locally/

  warnings.warn(incompatible_device_warn.format(device_name, capability, " ".join(arch_list), device_name))
'NVIDIA GeForce RTX 3060 Ti'
```


# 测试
在下面的环境将会介绍如何测试示例数据集。

## 1. 数据准备
有关DAIR-V2X数据集的情况参考[这里](https://github.com/haibao-yu/FFNet-VIC3D/blob/main/data/dair-v2x/README.md)。

### 1.1 数据预处理
在官方库中提供了两个示例数据集：
- 未预处理的 DAIR-V2X-Example 数据集 --> [下载](https://drive.google.com/file/d/1bFwWGXa6rMDimKeu7yJazAkGYO8s4RSI/view?usp=sharing)
- 已预处理的 DAIR-V2X-C-Example 数据集 --> [下载](https://drive.google.com/file/d/1y8bGwI63TEBkDEh2JU_gdV7uidthSnoe/view?usp=sharing)

对于原示例数据集DAIR-V2X-Example的具体数据形式：具体可以参考[这里](https://github.com/bestdecoy/FFNet_for_test/blob/main/docs/dair-v2x/DAIR-V2X-Examples.md)。

进行数据预处理之前，将数据集放置到`./data/dair-v2x`下。  
之后进行数据预处理，仅需：
```
python ./data/dair-v2x/preprocess.py --source-root ./data/dair-v2x/DAIR-V2X-Examples/cooperative-vehicle-infrastructure
```
> 其中，`--source-root`指的是：Raw data root of DAIR-V2X-C.

由代码可知，数据预处理的对象主要是车路协同端的数据集，我们只需要观察`./data/dair-v2x/DAIR-V2X-Examples/cooperative-vehicle-infrastructure`内的变化即可。

预处理前的文件树节选：

```
./data/dair-v2x/DAIR-V2X-Examples/cooperative-vehicle-infrastructure
├─cooperative
│  └─label_world
...
```
运行脚本转换：
![](https://pic1.imgdb.cn/item/646ce4810d2dde5777270450.jpg)


预处理后的文件树节选：

```
./data/dair-v2x/DAIR-V2X-Examples/cooperative-vehicle-infrastructure
├─cooperative
│  ├─calib
│  │  └─lidar_i2v
│  ├─label
│  │  └─lidar
│  └─label_world
...
```
预处理用于将位于世界坐标系的车端与路端联合视角下的标注转化为点云图和生成变换参数及新的数据索引`data_info_new.json`。


### 1.2 生成数据对
同样是有脚本用于生成：

`python ./data/dair-v2x/frame_pair_generation.py --source-root ./data/dair-v2x/DAIR-V2X-Examples/cooperative-vehicle-infrastructure
`


输出信息如下：
![](https://pic1.imgdb.cn/item/646ce7810d2dde57772c2204.jpg)

该脚本在文件系统中的输出为：   
观察路径：`${FFNet_root}/data/dair-v2x/flow_data_jsons`，新生成了：
```
-rw-r--r-- 1 root root  42990 May 23 16:18 flow_data_info_train.json
-rw-r--r-- 1 root root 143514 May 23 16:18 flow_data_info_train_2.json
-rw-r--r-- 1 root root      2 May 23 16:18 flow_data_info_val_0.json
-rw-r--r-- 1 root root      2 May 23 16:18 flow_data_info_val_1.json
-rw-r--r-- 1 root root      2 May 23 16:18 flow_data_info_val_2.json
-rw-r--r-- 1 root root      2 May 23 16:18 flow_data_info_val_3.json
-rw-r--r-- 1 root root      2 May 23 16:18 flow_data_info_val_4.json
-rw-r--r-- 1 root root      2 May 23 16:18 flow_data_info_val_5.json
```
参考[这里](https://github.com/haibao-yu/FFNet-VIC3D/blob/main/data/dair-v2x/README.md)，用于生成：
> We construct the frame pairs to generate the json files for FFNet training and evaluation.

回到项目脚本`./data/dair-v2x/frame_pair_generation.py`，发现切分依据在：
```
...
132     split_json_path = os.path.join('./data/dair-v2x/split_datas', 'cooperative-split-data.json')
133     split_jsons = read_json(split_json_path)
...
```
位于`./data/dair-v2x/split_datas/cooperative-split-data.json`的json是其切分依据，是一个经过标注的数据索引，切分的关键函数如下：

```
def split_datas(data_infos, split_datas, split='val'):
    data_infos_split = []

    inf_split_datas = split_datas['infrastructure_split'][split]
    for data_info in data_infos:
        infrastructure_frame = data_info['infrastructure_image_path'].split('/')[-1].replace('.jpg', '')
        if infrastructure_frame in inf_split_datas:
            data_infos_split.append(data_info)

    return data_infos_split
```
可见切分的依据便是该切分的json文件。

## 2. 评估
切分好准备的数据集为train，val以后，就可以进行评估了。