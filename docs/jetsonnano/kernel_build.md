# jetson nano 内核编译<!-- omit in toc -->
<!-- vscode-markdown-toc -->
* [使用Ubuntu18.04 发行版系统或虚拟机](#Ubuntu18.04)
	* [配置编译环境](#)
		* [编译](#-1)
* [使用Docker 构建交叉编译环境](#Docker)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->


## <a name='Ubuntu18.04'></a>使用Ubuntu18.04 发行版系统或虚拟机

需要下载的文件
>   gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar  交叉编译工具
>
>   public_sources.tbz2	L4T 内核源码
>
>   Jetson-210_Linux_R32.5.1_aarch64.tbz2  L4T 设备驱动包 用于烧录内核
>
>   Tegra_Linux_Sample-Root-Filesystem_R32.5.1_aarch64.tbz2 根文件系统

### <a name=''></a>配置编译环境

```bash
sudo apt install build-essential bc
sudo tar -xvf gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu.tar.xz -C /opt
tar -xjf public_source.tbz2
cd Linux_for_Tegra/source/public/
tar -xjf kernel_src.tbz2
JETSON_NANO_KERNEL_SOURCE=$(pwd)
```

#### <a name='-1'></a>编译

```bash
cd $JETSON_NANO_KERNEL_SOURCE
make mrproper -C kernel/kernel-4.9/
TOOLCHAIN_PREFIX=/opt/gcc-linaro-7.3.1-2018.05-x86_64_aarch64-linux-gnu/bin/aarch64-gnu-
TEGRA_KERNEL_OUT=$JETSON_NANO_KERNEL_SOURCE/build
KERNEL_MODULES_OUT=$JETSON_NANO_KERNEL_SOURCE/modules
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} tegra_defconfig
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} -j8 --output-sync=target zImage
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} -j8 --output-sync=target modules
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra CROSS_COMPILE=${TOOLCHAIN_PREFIX} -j8 --output-sync=target dtbs
make -C kernel/kernel-4.9/ ARCH=arm64 O=$TEGRA_KERNEL_OUT LOCALVERSION=-tegra INSTALL_MOD_PATH=$KERNEL_MODULES_OUT modules_install
```
得到的内核文件路径为build/arch/arm64/boot/Image

## <a name='Docker'></a>使用Docker 构建交叉编译环境

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



















