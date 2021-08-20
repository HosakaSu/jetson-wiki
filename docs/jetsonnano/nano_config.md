# 相关配置操作<!-- omit in toc -->
- [配置硬件时钟](#配置硬件时钟)
- [查看硬件温度](#查看硬件温度)
- [设置代理](#设置代理)
  - [apt 代理](#apt-代理)
  - [pip 代理](#pip-代理)
  - [终端代理](#终端代理)
  - [git 代理](#git-代理)
  
## 配置硬件时钟
``` bash
sudo timedatectl set-ntp no
sudo timedatectl set-time ‘2021-08-19 16:24:00’
sudo hwclock
```

## 查看硬件温度
除了使用jtop 直观查看硬件温度外, 还可以

查看硬件
``` bash
cat /sys/devices/virtual/thermal/thermal_zone*/type
```
查看硬件对应温度, 需要除以1000 得到实际温度
```bash
cat /sys/devices/virtual/thermal/thermal_zone*/temp
```

## 设置代理
首先在局域网内可联网的电脑内搭建socks5 代理服务

这里建议使用Clash for Windows 并开启"Allow LAN" 选项

### apt 代理
``` bash
sudo vim /etc/apt/apt.conf.d/proxy.conf
Acquire::http::Proxy "http://proxyAddress:port";
Acquire::https::Proxy "http://proxyAddress:port";
Acquire::socks5::proxy "socks://proxyAddress:port";
```
端口和IP 根据实际情况修改

### pip 代理
使用参数 --proxy=proxy
```bash
sudo pip3 install jetson-stats --proxy=http://proxyAddress:port
```
pip 不支持socks5 代理

### 终端代理
开启
```bash
export http_proxy=http://proxyAddress:port
export https_proxy=http://proxyAddress:port
```
关闭
```bash
unset http_proxy
unset https_proxy
```

### git 代理
开启
```bash
git config --global http.proxy 'socks5://proxyAddress:port' 
git config --global https.proxy 'socks5://proxyAddress:port'
```
关闭
```bash
git config --global --unset http.proxy
git config --global --unset https.proxy
```


