# jetson nano 配置安装步骤记录<!-- omit in toc -->

- [系统烧写与启动](#系统烧写与启动)
  - [从SD 卡启动](#从sd-卡启动)
  - [从USB 启动系统](#从usb-启动系统)
  - [Jetson nano 超频](#jetson-nano-超频)
- [软件安装与卸载](#软件安装与卸载)
  - [修改 apt 源文件 /etc/apt/sources.list](#修改-apt-源文件-etcaptsourceslist)
  - [安装软件](#安装软件)
    - [卸载自带的gdm3 , 安装lightdm 以及xfce4 桌面 (大约节省500MB 运行内存)](#卸载自带的gdm3--安装lightdm-以及xfce4-桌面-大约节省500mb-运行内存)
    - [安装jtop](#安装jtop)
    - [安装oh-my-zsh](#安装oh-my-zsh)
    - [编译安装CMake](#编译安装cmake)
    - [编译安装openCV](#编译安装opencv)

## 系统烧写与启动
### 从SD 卡启动
下载[镜像文件](https://developer.nvidia.com/jetson-nano-sd-card-image) 并解压出img 文件 

使用win32 diskImager 或 etcher 将镜像文件烧录至SD 卡中

烧写完毕后, 忽略各分区需要格式化的警告, 弹出SD 卡并插入jetson nano 卡槽中, 上电启动

** 注意 **

> jetpack 更新到4.5 后bootloader 将从SD 卡 移至模块内置的QSPI-NOR 存储芯片内, 届时已烧录的SD卡将无法在jetpack 版本在4.4 或以下的jetsonnano 底板上启动
> 
接入屏幕, 按照要求设置用户名密码后安装系统, 完成后重启进入桌面

### 从USB 启动系统

相比于SD 卡, USB 移动硬盘拥有更好的读写性能(这里指固态硬盘), 这里我们将配置jetson nano 从usb 启动系统.

首先烧写系统镜像至SD 卡内并启动系统

将已格式化为EXT4 格式的移动硬盘连接至jetson nano

使用 `lsblk | grep /dev/sda*` 命令查看移动硬盘是否已被识别出

如果移动硬盘分区为`/dev/sda1`, 则执行以下命令

`wget https://github.com/JetsonHacksNano/bootFromUSB/blob/main/copyRootToUSB.sh`

`chmod +x RootToUSB.sh`

`./RootToUSB.sh -p /dev/sda1`

等待命令执行完毕

更改启动顺序

挂载移动硬盘分区，打开 `boot/extlinux/extlinux.conf`, 找到下行所示的配置

```bash
APPEND ${cbootargs} quiet root=/dev/mmcblk0p1 rw rootwait rootfstype=ext4 console=ttyS0,115200n8  console=tty0 fbcon=map:0 net.ifnames=0
```
`root=/dev/mmcblk0p1` 表示系统默认从SD 卡分区启动, 这里修改成`/dev/sda1` 就可以从USB 移动硬盘启动, 但这样存在风险: 如果上电开机时jetson nano 上连接有多个usb 硬盘或者SD卡, 则会导致启动失败. 因此我们应该使用移动硬盘的UUID 来指定启动分区

读取移动硬盘的UUID

`echo $(sudo blkid -o value -s PARTUUID /dev/sda1)`

将启动文件指向改成如下所示

```bash
APPEND ${cbootargs} quiet root=PARTUUID=e7f125ea-7035-42c4-adda-a108ccb3331b rw rootwait rootfstype=ext4 console=ttyS0,115200n8 console=tty0 fbcon=map:0 net.ifnames=0
```

关机，拔出SD 卡再重新上电, 即可从USB移动硬盘进入系统.

### Jetson nano 超频

## 软件安装与卸载

### 修改 apt 源文件 /etc/apt/sources.list

注释掉原有的源地址, 加入清华源地址
```bash
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-security main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-updates main multiverse restricted universe
deb-src http://mirrors.tuna.tsinghua.edu.cn/ubuntu-ports/ bionic-backports main multiverse restricted universe
```
修改完成后执行
```bash
sudo apt update
sudo apt upgrade -y
```
此时可能会遇到`nvidia-l4t-bootloader` 更新报错的问题

`dpkg: error processing package nvidia-l4t-bootloader (*--configure)*`

解决办法
```bash
sudo mv /var/lib/dpkg/info/ /var/lib/dpkg/info_old/
sudo mkdir /var/lib/dpkg/info/
sudo apt-get update
sudo apt-get -f install
sudo mv /var/lib/dpkg/info/* /var/lib/dpkg/info_old/
sudo rm -rf /var/lib/dpkg/info
sudo mv /var/lib/dpkg/info_old/ /var/lib/dpkg/info/
```
然后再执行一次 `sudo apt update && sudo apt upgrade` 即可

为节省空间, 删除系统中不需要的软件
```bash
sudo apt purge libreoffice-* gnome-mahjongg gnome-sudoku gnome-mines gnome-todo gnome-calendar thunderbird transmission-*
```

如果开机系统语言选的中文, 为了方便起见, 建议把home 下目录名改为英文
```bash
sudo apt install -y xdg-user-dirs-gtk
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN.UTF-8
```
### 安装软件

#### 卸载自带的gdm3 , 安装lightdm 以及xfce4 桌面 (大约节省500MB 运行内存)
```bash
sudo apt install lightdm xfce4
sudo apt purge gdm3
sudo apt install openssh-server nano htop screen wget curl vim zsh neofetch
```
开机后在输入密码界面选择使用xfce4 桌面

#### 安装jtop
``` bash
sudo apt install python3-pip
sudo pip3 install jetson-stats
jtop
```
最新版本的jetson-stats 已经不需要root权限执行 jtop了
#### 安装oh-my-zsh
```bash
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ~/.oh-my-zsh/plugins/zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-autosuggestions.git $ZSH_CUSTOM/plugins/zsh-autosuggestions

vim ~/.zshrc
plugins=(git zsh-autosuggestions zsh-syntax-highlighting)
source ~/.zshrc
```
#### 编译安装CMake

以cmake-3.20.5 为例

首先移除自带的cmake

`sudo apt purge cmake`

下载源码
```bash
wget https://github.com/Kitware/CMake/archive/refs/tags/v3.20.5.tar.gz
tar -xvzf v3.20.5.tar.gz
cd CMake-3.20.5/
```
修改CMakeLists.txt, 加入`set(CMAKE_USE_OPENSSL OFF)` 
```bash
./configure
make -j4
sudo make install
```
查看cmake 版本
`cmake --version`

如果返回值是3.20.5 则安装成功

#### 编译安装openCV

首先增加swap 分区大小, 建议设为8G
```bash
sudo apt install dphys-swapfile
sudo vim /sbin/dphys-swapfile
```
将CONF_MAXSWAP 项改为需要的大小

`sudo reboot`

`free -lh` 查看swap 分区

下载opencv 所需安装版本的源码并解压
```
wget https://github.com/opencv/opencv/archive/3.3.1.zip
unzip 3.3.1.zip
```
执行 `./configure`, 无报错后 `make -j4 && sudo make install`

大约需要2 小时完成, 建议加上散热风扇

