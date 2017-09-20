---
layout: post
title: 'Raspberry pi 設定 WIFI 固定 IP'
tags: [RaspberryPi]
---

###### tags: `rpi` `wifi` `static-ip`

* [參考來源](https://www.modmypi.com/blog/tutorial-how-to-give-your-raspberry-pi-a-static-ip-address)
* [中文部落格](http://onionys.blogspot.tw/2015/03/raspberry-pi-wifi.html)

---

1. 開啟修改網路設定: `$ sudo nano /etc/network/interfaces`
2. `wlan0` 底下找到 broadcast, netmask: `$ ifconfig` 
![](https://i.imgur.com/AzxbMuC.png)

3. 找到 gateway, network(destination): `$ netstat -nr`
![](https://i.imgur.com/sRSjekK.png)

4. 設定完應該像下面
> 保險起見也可以先註解掉原本的設定
```
auto wlan0
allow-hotplug wlan0

iface wlan0 inet static
address 192.168.1.81 # 你要指定他的 address
netmask 255.255.255.0 
network 192.168.1.0 
broadcast 192.168.1.255 
gateway 192.168.1.254 

wpa-ssid "你的無線AP的SSID"
wpa-psk "你的無線AP的password"
```
>若不知道 AP 名稱可嘗試: `sudo iwlist scan | grep
 "ESSID"`
 
5. 重新開機 `$ sudo reboot`

6. 測試是否成功 `$ ping {gateway} -c 10`

![](https://i.imgur.com/lP4sLt7.png)

這樣就代表成功了
