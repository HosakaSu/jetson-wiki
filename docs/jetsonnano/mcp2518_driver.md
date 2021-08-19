# mcp2518 驱动安装<!-- omit in toc -->
- [下载源码](#下载源码)
- [修改设备树文件](#修改设备树文件)
- [编译安装dtb](#编译安装dtb)

## 下载源码
``` bash
git clone https://github.com/Seeed-Studio/seeed-linux-dtoverlays
cd seeed-linux-dtoverlays
```
## 修改设备树文件
需要根据实际情况修改设备树, 例如

seeed 提供的设备树文件 2xMCP2518FD-spi0.dts 在SPI0 下挂在了两片mcp2518fd, 中断管脚为GPIO_B5

这里我们要根据tegra-gpio.h 中的管脚标号计算自己使用的中断脚编号

例如 如果使用nano 的GPIO168, 则对应的是GPIO_V0

如果使用的是SPI1 接口，则
``` dts
	fragment@0 {
		target-path = "/spi@7000d400/spi@0";
		__overlay__ {
			status = "disabled";
		};
	};

	fragment@1{
		target-path = "/spi@7000d400/spi@1";
		__overlay__ {
			status = "disabled";
		};
	};
```
中的@7000d400 要改为7000d600
如果仅使用一片2518, 则删掉fragment@1
最后, 还需要修改pinmux 中的管脚定义, 将原有的SPI0 管脚改为SPI1
## 编译安装dtb
```bash
make all_jetsonnano
sudo -E make install_jetsonnano
sudo cp overlays/jetsonnano/2xMCP2518FD-spi1.dtbo /boot
sudo /opt/nvidia/jetson-io/config-by-hardware.py -n "Seeed 2xMCP2518FD"
```
完成后重启nano, 启用CAN 接口
```bash
sudo ip link set can0 type can bitrate 500000
sudo ip link set up can0
```


