---
layout: post
title: 'MAVROS on RPi3'
tags: [Notes]
---

###### tags: `ros` `mavros` `rpi` `sop`

## 環境建置

#### step 1: 安裝ubuntu-mate 16.04
首先，準備一張容量大於8G的SD卡
至官網下載 Ubuntu mate 16.04 
```
https://ubuntu-mate.org/raspberry-pi/ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img.xz
```
接著，開啟Terminal輸入以下指令將Ubuntu mate 16.04燒錄至SD卡
```
 sudo apt-get install gddrescue xz-utils
 unxz ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img.xz
 sudo ddrescue -D --force ubuntu-mate-16.04-desktop-armhf-raspberry-pi.img /dev/sdx
```

將燒錄完成的SD卡放入Raspberry pi 3，並將Raspberry pi 3接上電源開機

#### step 2:安裝ROS
執行以下指令設定 sources.list ，使它可以接受packages.ros.org的軟體
```
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
```
接著，設定金鑰
```
sudo apt-key adv --keyserver hkp://ha.pool.sks-keyservers.net:80 --recv-key 0xB01FA116
```
更新套件清單
```
sudo apt-get update
```
>若出現以下錯誤:
>```
>E: Could not get lock /var/lib/apt/lists/lock - >open (11: Resource temporarily unavailable)
>E: Unable to lock directory /var/lib/apt/lists/
>```
>則執行以下步驟把錯誤的 lock 刪掉，之後再重新更新一次:
>```
>$ sudo rm -rf /var/lib/lists/lock
>$ sudo dpkg --configure -a
>```

安裝ros-base
```
sudo apt-get install ros-kinetic-ros-base
```
安裝完base後安裝desktop
```
sudo apt-get install ros-kinetic-desktop
```
初始化rosdep(rosdep是用來解決ROS套件相容性的問題)
```
sudo rosdep init
rosdep update
```
當這邊都裝完成後，還需設定環境變數
```
echo "source /opt/ros/kinetic/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
>＊NOTE : 如果不只安裝一個版本的ROS的話，可以透過指定特定的setup.bash，來選擇你想要的ROS版本。執行以下指令:
>```
>source /opt/ros/kinetic/setup.bash
>```
最後，執行rosinstall，將相關連ROS package一次下載安裝完畢
```
sudo apt-get install python-rosinstall
```
#### step 3: 安裝 MAVROS
```
sudo apt-get install python-catkin-tools python-rosinstall-generator
```

1. unneded if you already has workspace
```
mkdir -p ~/catkin_ws/src
cd ~/catkin_ws
catkin init
wstool init src
```
2. get source (upstream - released)
```
rosinstall_generator --upstream mavros | tee /tmp/mavros.rosinstall
```
>alternative: latest source
>```
>rosinstall_generator --upstream-development >mavros | tee /tmp/mavros.rosinstall
>```

3. latest released mavlink package
you may run from this line to update ros-*-mavlink package
```
rosinstall_generator mavlink | tee -a /tmp/mavros.rosinstall
```
>alternative: to build master on Indigo or Jade:
>```
>rosinstall_generator --rosdistro kinetic -->upstream mavlink | tee -a /tmp/mavros.rosinstall
>```

4. workspace & deps
```
wstool merge -t src /tmp/mavros.rosinstall
wstool update -t src
rosdep install --from-paths src --ignore-src -y
```
5.0. allocate swap space for mavros installation
檢查 swap memory
```
$sudo swapon -s
```
若回傳結果是空的，表示沒有將 swap enable。增加 2GB 的 swap memory 容量:
```
$sudo dd if=/dev/zero of=/swapfile bs=1024 count=2M
$sudo mkswap /swapfile
$sudo swapon /swapfile
```
> 這邊可能遇到以下:
> ```
> swapon: /swapfile: insecure permissions 0644, 0600 suggested.
> ```
> 改一下 swapfile 權限:
> ```
> $ sudo chmod 600 /swapfile
> ```

安裝 vim
```
$ sudo apt-get install vim
```


在fstab中增加這一行，使得swap為永久的
```
$sudo vim /etc/fstab
/swapfile       none    swap    sw      0       0 
```


5. finally - build
build 之前先檢查一下現在記憶體使用狀況(以 MB 為單位列出):
```
$ free -m
```
確定 swap 沒有被占用，再執行最後一步:
```
catkin build
```

---

* 目前 mavros/mavlink 版本

![](https://i.imgur.com/cffXq9D.png)

* 安裝成功訊息:


![](https://i.imgur.com/qtpzHBK.png)

## Reference

* [mavros on raspberry pi 3](https://docs.google.com/document/d/1ZZZeF4QtDbHJLRRviZ4XfYKzI3YTtZv2t_gpkLs5NFM/edit#heading=h.izqg5b6rtc0m)
* [mavros installation 官方](http://rosindex.github.io/p/mavros/)

## 驗證安裝是否成功
### 飛機部分
#### step 1: 設定與companion computer連線baud
在QGroundControl上面設定參數[SYS_COMPANION](https://pixhawk.org/firmware/parameters#system)為`Companion Link (921600 baud, 8N1)`
#### step 2: 設定Offboard switch channel(目前設定為Channel 9)
#### step 3: 飛機與rpi3連線(不接5V)
![](https://i.imgur.com/y1eIVf8.jpg)
> * telem2 r腳位
> ![](https://i.imgur.com/YMFJYRt.jpg)
> * rpi3 pin
> ![](https://i.imgur.com/XyhMIsm.jpg)



### rpi3部份
#### step 1: 開啟terminal，完成參數定義，並執行ros的master node
```
$ cd catkin_ws/
$ source devel/setup.bash
$ roscore
```

#### step 2: 開啟另一個terminal，完成mavros跟飛機的連線(飛機為上電狀態)
```
$ source devel/setup.bash
$ rosrun mavros mavros_node _fcu_url:=/dev/ttyS0:921600
```
> 如果有遇到`Permission denied`，執行以下[指令](http://dev.px4.io/starting-installing-linux.html)，再重新開機，並且從rpi3 step 1開始
> ```
> $ sudo usermod -a -G dialout $USER
> ```
#### step 3: 再開第三個terminal，從rpi3下指令給飛機
```
$ rosrun mavros mavsafty arm
```
若馬達有轉動即rpi3與飛機有成功連線











