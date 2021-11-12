# jetson nano 系统备份以及刷机 <!-- omit in toc -->
<!-- vscode-markdown-toc -->
* 1. [sd卡版本刷写系统及安装驱动](#sd)
* 2. [emmc 版本刷写系统及驱动安装](#emmc)
	* 2.1. [使用SDK Manager 刷写系统](#SDKManager)
	* 2.2. [使用flash.sh 刷写系统](#flash.sh)

<!-- vscode-markdown-toc-config
	numbering=false
	autoSave=true
	/vscode-markdown-toc-config -->
<!-- /vscode-markdown-toc -->



通过nvidia 提供的SDK-Manager 软件可以对jetson 模块进行系统刷写和jetpack 安装.
# 硬件版本说明
目前使用的模块有B01 sd卡以及emmc 版本.
sd卡版本对应型号是
##  1. <a name='sd'></a>sd卡版本刷写系统及安装驱动
关于sd卡版本的系统及驱动安装不再赘述, 主要问题在于emmc 版本.
##  2. <a name='emmc'></a>emmc 版本刷写系统及驱动安装
###  2.1. <a name='SDKManager'></a>使用SDK Manager 刷写系统
从[这个链接](https://developer.nvidia.com/nvidia-sdk-manager)下载对应的deb 包, 安装运行即可. 注意: sdk manager 每次启动均需联网验证开发者账号, 所以还需要互联网连接.
关于使用sdk manager 进行系统刷写的步骤无需多讲, 只有一点需要注意: 如果是在虚拟机内进行操作, 最好能分出100 G的空间. 
###  2.2. <a name='flash.sh'></a>使用flash.sh 刷写系统
解压 Jetson-210_Linux_R32.5.2_aarch64.tbz2
```bash
tar -xjf Jetson-210_Linux_R32.5.2_aarch64.tbz2
cd Linux_for_Tegra
```
模块断电, 跳线连接"FC REC" 和GND, 上电, 如果使用虚拟机, 则要将弹出的USB 设备挂载至虚拟机内. 在虚拟机内通过 lsusb 确认设备已挂载后断开跳线.

默认刷写模块
```bash
sudo ./flash.sh jetson-nano-emmc internal
```
只刷写设备树
将需要更新的dtb文件放入kernel/dtb/下
```bash
sudo ./flash.sh -r -k DTB jetson-nano-emmc internal
```
克隆模块内文件系统, <clone>自定义名称, 将会产生两个文件: <clone>.img 和 <clone>.img.raw
```bash
sudo ./flash.sh -r -k APP -G <clone> jetson-nano-emmc internal
```
将生成的<clone>.img 传入bootloader内并修改为system.img
如果模块已经刷入系统, 则执行
```bash
sudo ./flash.sh -r -k APP jetson-nano-emmc internal
```
如果模块还未刷入系统, 则
```bash
sudo ./flash.sh -r jetson-nano-emmc internal
```
