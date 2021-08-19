# 相关配置操作<!-- omit in toc -->
- [配置硬件时钟](#配置硬件时钟)
- [查看硬件温度](#查看硬件温度)
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