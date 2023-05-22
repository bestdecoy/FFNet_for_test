ARG PYTORCH="1.9.0"
ARG CUDA="11.1"
ARG CUDNN="8"
# Check the tag version at https://hub.docker.com/r/pytorch/pytorch/tags
FROM pytorch/pytorch:${PYTORCH}-cuda${CUDA}-cudnn${CUDNN}-devel

ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 7.0+PTX"
ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"
ENV CMAKE_PREFIX_PATH="$(dirname $(which conda))/../"

# Enroll new gpg key
RUN rm /etc/apt/sources.list.d/cuda.list
RUN rm /etc/apt/sources.list.d/nvidia-ml.list
RUN apt-key del 7fa2af80
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/3bf863cc.pub
RUN apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1804/x86_64/7fa2af80.pub

# apt aliyun
RUN sed -i s@/archive.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN sed -i s@/security.ubuntu.com/@/mirrors.aliyun.com/@g /etc/apt/sources.list
RUN apt clean
RUN apt-get update -y

# pip config tuna
RUN pip config set global.index-url  https://pypi.tuna.tsinghua.edu.cn/simple
RUN apt-get update
RUN apt-get install -y ffmpeg libsm6 libxext6 git ninja-build libglib2.0-0 libsm6 libxrender-dev libxext6 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Install MMCV, MMDetection and MMSegmentation
RUN pip install mmcv-full==1.3.14 -f https://download.openmmlab.com/mmcv/dist/cu101/torch1.6.0/index.html --trusted-host mirrors.aliyun.com
RUN pip install mmdet==2.14.0 
RUN pip install mmsegmentation==0.14.1 

# Install MMDetection3D
RUN conda clean --all
RUN git clone https://github.com/open-mmlab/mmdetection3d.git /mmdetection3d
WORKDIR /mmdetection3d
RUN git checkout v0.17.1
ENV FORCE_CUDA="1"
RUN pip install -r requirements/build.txt 
RUN pip install --no-cache-dir -e .

# Fix something wrong
RUN pip install psutil==5.9.1 
RUN pip install --upgrade git+https://github.com/klintan/pypcd.git

# Install FFNet
# RUN git clone https://github.com/haibao-yu/FFNet-VIC3D.git
# RUN mv FFNet-VIC3D ffnet
# RUN cd ffnet
# RUN pip install -e . --user

