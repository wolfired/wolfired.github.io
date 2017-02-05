[TOC]

## 目的
Ubuntu Desktop自带了很多附加软件包，所以想先安装Ubuntu Server再安装桌面环境以减少不必要的软件包

## 版本
Ubuntu Server 16.04.1 LTS

## 安装
* 安装Ubuntu Server
* 安装桌面环境
```bash
sudo apt-get install --no-install-recommends ubuntu-desktop gnome-terminal
```

## 连网(WiFi)

* 首先关闭NetworkManager服务
```bash
sudo systemctl stop NetworkManager
sudo systemctl disable NetworkManager
```

* 查看无线网卡
```bash
ifconfig -a
```
假设无线网卡接口为：`wlan0`

如有必要，先停止无线网卡
```bash
sudo ifconfig wlan0 down
```

* 手动连网
```bash
wpa_passphrase ssid passwd > wpa_supplicant-wlan0.conf
sudo wpa_supplicant -B -i wlan0 -c wpa_supplicant-wlan0.conf
sudo dhclient wlan0
```

* 自动连网
```bash
sudo vi /etc/network/interfaces
```
增加以下内容
```text
#Dynamic IP
auto wlan0
iface wlan0 inet dhcp
wpa-ssid your_ssid
wpa-psk your_passwd
```
或
```text
#Static IP
auto wlan0
iface wlan0 inet static
wpa-ssid your_ssid
wpa-psk your_passwd
address 192.168.1.2
netmask 255.255.255.0
gateway 192.168.1.1
dns-nameservers 192.168.1.1
```

## Intel Graphics Driver(可选)
[Intel Graphics Update Tool](https://01.org/zh/linuxgraphics/downloads/intel-graphics-update-tool-linux-os-v2.0.2)

[Running the Update Tool using gdebi](https://01.org/linuxgraphics/documentation/running-update-tool-using-gdebi)

自动更新显卡驱动相关软件包
```bash
sudo intel-graphics-update-tool
```

如果运行时报错，可能是因为缺少packagekit软件包
```bash
sudo apt-get install packagekit
```

## 思源字体(可选)
[Source Han Sans](https://github.com/adobe-fonts/source-han-sans/tree/release)

解压字体文件到`~/.fonts/`

执行`sudo fc-cache -f -v`