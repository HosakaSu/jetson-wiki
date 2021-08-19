# jetson nano 内核文件预编译<!-- omit in toc -->

- [使用Ubuntu18.04 发行版系统或虚拟机](#使用ubuntu1804-发行版系统或虚拟机)
  - [交叉编译L4T](#交叉编译l4t)
  - [构建Nvidia 内核](#构建nvidia-内核)
    - [设置环境变量](#设置环境变量)
    - [生成.config 文件](#生成config-文件)
    - [编译生成设备树文件和模块文件](#编译生成设备树文件和模块文件)
- [使用Docker 构建交叉编译环境](#使用docker-构建交叉编译环境)
## 使用Ubuntu18.04 发行版系统或虚拟机
>   需要下载的文件
>
>   gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar  交叉编译工具
>
>   public_sources.tbz2	L4T 内核源码
>
>   Jetson-210_Linux_R32.5.1_aarch64.tbz2  L4T 设备驱动包 用于烧录内核
>
>   Tegra_Linux_Sample-Root-Filesystem_R32.5.1_aarch64.tbz2 根文件系统

### 交叉编译L4T

解压内核源码压缩包

`tar -xjf public_source.tbz2`

`cd Linux_for_Tegra/source/public/`

`tar -xjf kernel_src.tbz2`

`cd kernel/kernel-4.9`

### 构建Nvidia 内核

安装预装软件

`sudo apt install build-essential bc`

安装交叉编译工具链

`sudo tar -xvf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar.xz -C /opt`

#### 设置环境变量

>   `TEGRA_KERNEL_OUT=<outdir>` 
>
>   <outdir> 为保存编译好的内核文件位置
>
>   `export CROSS_COMPILE=<cross_prefix>`
>
>   <cross_prefix> 为交叉编译工具链的绝对路径
>
>   `export LOCALVERSION=-tegra`

如果想使编译工具链永久生效，修改环境变量

`vim ~/.bashrc`

`export PATH=/opt/gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu/bin/:$PATH`

`source ~/.bashrc`

#### 生成.config 文件

>   `cd <kernel_source>`
>
>   <kernel_source> 为解压出来的内核源码路径，例如kernel-4.9文件夹
>
>   `mkdir -p $TEGRA_KERNEL_OUT`
>
>   `make ARCH=arm64 O=$TEGRA_KERNEL_OUT tegra_defconfig`

#### 编译生成设备树文件和模块文件

第一次编译前最好清理一下

`make mrproper`

然后开始编译，根据电脑配置增加线程 -n

`make ARCH=arm64 O=$TEGRA_KERNEL_OUT -j<n>`

得到的内核文件路径为arch/arm64/boot/Image

安装Linux_for_Tegra

烧录内核

解压设备驱动包

`tar -xjf jetson-210_linux_r32.5.1_aarch64.tbz2`

解压文件系统

## 使用Docker 构建交叉编译环境

新建文件夹，放入gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar

新建Dockerfile，内容

>   `FROM ubuntu:18.04`
>   `USER root`
>   `COPY gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar .`
>   `RUN apt-get -y update \`
>   `&& apt-get install -y build-essential bc wget tar vim \`
>   `&& tar -xvf gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar -C /opt \`
>   `&& echo "export PATH=/opt/gcc-linaro-7.5.0-2019-12-x86_64_aarch64-linux-gnu/bin/:$PATH" >> ~/.bashrc  \`
>   `&& /bin/bash -c "source ~/.bashrc" \`
>   `&& rm gcc-linaro-7.5.0-2019.12-x86_64_aarch64-linux-gnu.tar \`
>   `&& apt-get autoremove`

执行

`docker build -t crosscompile/nano:v2 -f ./Dockerfile .`

`docker run -d --name nano_compile -v /home/sirius/Documents/nano:/data -it crosscompile/nano:v2`



















