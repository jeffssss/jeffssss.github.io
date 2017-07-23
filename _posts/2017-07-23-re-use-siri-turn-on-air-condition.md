---
published: true
title: Re.从0开始用Siri控制空调开机
layout: post
author: Jeffssss 
comments: true
category: 树莓派
tags:
- 树莓派
- 智能家居
---


花了三天时间折腾，终于能够用Siri打开空调，在此记录一下过程。内容涉及到树莓派、红外、Homebridge等相关技术。

## 背景
炎炎夏夜，当我风尘仆仆回到家时，只觉得我是从一个大蒸笼回到了一个小蒸笼里，我打开空调，等待房间温度慢慢下降，我这才觉得我是活着的。

几天前，当我依旧满身汗水回到家，打开空调时，我有了一个想法：我希望在我下班回家时，卧室的空调已经在运行中。

我是租房住，所以不可以对房间有破坏性的改造，不可能换一个空调。因为下班时间不固定，所以无法设置定时开机。所以我定下的目标是：用手机远程控制空调开机。

## 调研

回忆起大学的暑期实习，有小伙伴利用MSP430配合红外发射模块控制实验室的空调，于是我也想通过单片机等设备，控制红外模块，达到控制空调的目的。MSP430板子对开发不友好，我决定用树莓派这种高大上的玩意。

目标确认后，我需要对树莓派、红外模块等相关软硬件做一些调研。

### 硬件

* 树莓派

	Raspberry Pi，只有信用卡大小的微型电脑，其系统基于Linux。麻雀虽小，五脏俱全，CPU、GPU、内存、USB、GPIO、Camera、HDMI等都具备。树莓派在物联网以及智能家居中的应用非常多，相关的开发也十分方便。
	
	目前最新的版本是"树莓派3代B型"，自带Wifi和蓝牙, 某宝上搜索很容易买到，价格在两三百的范围内。我买的套餐价是306RMB，套餐里有电源线、壳子、散热片、小风扇、SD卡、读卡器、和一个收纳盒。其中，刷系统时要用到SD卡，电源线不必说，供电要用。这两个是非常必要的，不然树莓派工作不起来。
	
* 红外发射模块

	常见的红外发射模块，发射距离：1-2m，波长：940nm 
	
	某宝上我买的4.3RMB。如果距离够的话，无需功率放大器。
	
* 红外接收模块

	KY-022红外传感器接收器模块，采用的是1838红外接收头，接收距离18米。某宝上2.41RMB。
	
	 为何要用到红外接收模块，将在后续软件调研中说明。

* 杜邦线

	连接红外模块与树莓派上的GPIO引脚。杜邦线的口子分公母，10个母对母的杜邦线1.23RMB。

### 软件

* [LIRC](http://www.lirc.org/)

	Linux红外遥控(LIRC)是一套控制树莓派红外接口硬件的程序,通过这个包可以实现红外信号的采集以及发射。
	
	为何要采集红外信号？这和空调的红外信号有关系。
	
	若要发出的信号空调能识别，必须要知道空调的红外码率以及码表。红外协议中比较有名的是[NEC协议](http://www.cnblogs.com/yulongchen/archive/2013/04/12/3017409.html)。但是由于空调厂商很多，没有一个规范的协议，所以不同的厂商甚至不同的型号的空调，码率都可能不一样。同时空调的红外信号复杂，一个信号会包含温度、风力、模式、定时等信息，需要知道空掉对应的码表才能发出空调能识别的红外信号。可惜码表在网上基本查不到，资料不开源。于是需要利用LIRC以及红外接收模块录制空调遥控器的信号，来实现简单的空调控制。
	
* [HAP-NodeJS](https://github.com/KhaosT/HAP-NodeJS)

	苹果HomeKit的Accessory服务器的NodeJs版本的实现。感谢HAP-NodeJS,方便了许多智能家居接入HomeKit的开发。今夜我们都是Node程序员。
	
	它是干嘛的呢？它把一个个设备模拟成苹果的HomeKit设备，然后就可以在iOS设备里面使用了。为什么要模拟呢？因为苹果的HomeKit是封闭的，不授权不能使用，这个时候我们只能给自己的设备披上一层外衣，让HomeKit能识别并控制。
	
* [Homebridge](https://github.com/nfarina/homebridge)

	Homebridge是基于HAP-NodeJS开发的NodeJS服务端程序，支持插件化开发。
	
	Homebridge与HomeKit API相关的逻辑完全来自HAP-NodeJS，个人认为Homebridge做的工作就是插件化，将平台程序与各个设备的特殊处理逻辑剥离出来，用户可以通过npm安装一个现有的插件，非常轻松的和快速的完成设备的接入。
	
* [home-assistant](https://home-assistant.io/)

	home-assistant是一个开源的智能家居自动化平台。运行需要Python3环境。
	
	home-assistant有app端和web端的控制平台，功能与HomeKit类似。它从人的操作中接收命令，执行业务逻辑，达到控制智能设备的目的。
	
* [homebridge-homeassistant](https://github.com/home-assistant/homebridge-homeassistant)
	
	homebridge-homeassistant是Homebridge的一个插件，用于连通homeassistant和Homebridge，这样，homeassistant中的设备可以通过HomeKit来操控。
	
以上4个工具的关系如下图
![软件关系](/img/智能家居各软件关系.png)

## 最终方案

1. 使用树莓派3b，连接红外接收模块以及红外发送模块。
2. 使用LIRC来录制和发射空调的开机命令
4. 使用HAP-NodeJS，将树莓派接入HomeKit，从而实现Siri控制空调开机。

## 过程

### 树莓派安装

买回来的崭新树莓派以及SD卡，肯定是没有系统的。我们需要自己刷系统。Windows可以参考 [这个文档](HAP-NodeJS)。我的系统是Mac OSX，以下步骤为mac下的操作，参考的文档是：[Mac OSX下给树莓派安装Raspbian系统](http://shumeipai.nxez.com/2014/05/18/raspberry-pi-under-mac-osx-to-install-raspbian-system.html?variant=zh-cn)，以下的例子均来自这个文档，因为我已经装过一次系统了，所以不太方便重现。

1. 下载系统镜像。

	[树莓派官方](https://www.raspberrypi.org/downloads/)提供了很多镜像,我下载的是`
RASPBIAN JESSIE LITE`, 下载后解压。

	![系统镜像](/img/系统镜像.png)

2. 插入SD卡，用df命令查看当前已挂载的卷

	引用上面参考文档的例子：
	
	```
	[zhangshenjia@mac: pi]$df -h
	Filesystem      Size   Used  Avail Capacity  Mounted on
	/dev/disk0s2   112Gi   96Gi   15Gi    87%    /
	devfs          183Ki  183Ki    0Bi   100%    /dev
	map -hosts       0Bi    0Bi    0Bi   100%    /net
	map auto_home    0Bi    0Bi    0Bi   100%    /home
	/dev/disk1s1    15Gi  2.3Mi   15Gi     1%    /Volumes/未命名
	```
	
3. 使用diskutil unmount将这些分区卸载：

	```
	[zhangshenjia@mac: pi]$diskutil unmount /dev/disk1s1
2
	Volume 未命名 on disk1s1 unmounted
	```
4. 通过diskutil list来确认设备：

	```
	[zhangshenjia@mac: pi]$diskutil list
/dev/disk0
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *121.3 GB   disk0
   1:                        EFI                         209.7 MB   disk0s1
   2:                  Apple_HFS Macintosh HD            120.5 GB   disk0s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk0s3
/dev/disk1
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:     FDisk_partition_scheme                        *15.8 GB    disk1
   1:               Windows_NTFS 未命名                  15.8 GB    disk1s1
	``` 
5. 使用dd命令将系统镜像写入，最后的of后面写diskutil list里显示的dis名字，if后是镜像的文件路径

	```
	dd bs=4m if=2017-07-05-raspbian-jessie-lite.img of=/dev/rdisk1
	```

6. 等待一段时间，镜像会慢慢写进SD卡。
	
7. 使用`diskutil unmountDisk` 卸载设备。

8. 把SD卡插进树莓派，就算是装好系统了。第一次启动会进入Raspi-config。

9. 通过HDMI接口接上显示器，通过USB接上鼠标键盘,接上网线，设置一些东西，具体见文档[使用raspi-config配置树莓派](http://shumeipai.nxez.com/2013/09/07/raspi-config-configuration-raspberry-pie.html) 以及 [树莓派3B初始化后一些必须的设置](http://www.linuxidc.com/Linux/2016-12/138792.htm)

10. 开启ssh

	树莓派3默认关闭ssh连接，解决方案是把SD卡拔下来，进入到根目录，新建一个名为ssh的空白文件就行了。
	
11. 启动wifi

	见文档[启动wifi模块](http://blog.csdn.net/u010900754/article/details/53041791)，核心操作是将wiki的ssid和密码写入wpa_supplicant.conf文件中
	
	```
	sudo nano /etc/wpa_supplicant/wpa_supplicant.conf  
	```
	内容为
	
	```
	network={  
    	ssid="XXXX"  
    	psk="XXXX"  
	}  
	```

### 红外信号处理

使用树莓派控制红外模块，参考自以下文档 [使用树莓派红外控制空调和风扇](https://linux.cn/article-3782-1.html)

不得不吐槽的是，参考的这个文档百度上一搜一大推，内容全都一样。文档里有些关键说明有语病，还得自己多尝试和折腾才弄懂啥意思。

1. 硬件连接。

	红外接收器：
	
	* vcc连pin1(3.3v)
	* gnd连pin6(ground)
	* data连 pin12(gpio18)
	
	红外发射器:
	
	* vcc 连pin2(5v)
	* gnd连pin25(ground)
	* data连pin11(gpio17)

	如果不连GPIO17和18，需要单独配置。
	
2. 软件安装

	1. 安装lirc

		```
		sudo apt-get install lirc
		```
	2. 加载驱动

		```
		sudo modprobe lirc_rpi
		```
		
		此处有坑：报错：
		
		```
		pi@jeffberry:~ $ sudo modprobe lirc_rpi
		modprobe: ERROR: could not insert 'lirc_rpi': No such device
		```
		解决方案来自[CSDN论坛](http://bbs.csdn.net/topics/391944909)
		
		```
		sudo vi /boot/config.txt
		```
		找到#dtoverlay=lirc-rpi，把前面的“#”号去掉， 然后重启系统即可

	3. 测试lirc和红外接收模块

		```
		sudo mode2 -d /dev/lirc0
		```
		然后用遥控器对着红外接收，能看到如下输出：
		
		```
		space 4960669
		pulse 2697
		……
		pulse 2697
		```
		其中，space表示低电平，pulse表示高电平。一般来说，如果space的值和pulse的值差不多，代表0，如果space的值是pulse的值的3倍到4倍，表示1.
		
	4. 修改/etc/lirc/hardware.conf  文件中的 DRIVER和DEVICE
	
		```
		pi@raspberrypi ~ $ cat /etc/lirc/hardware.conf 
		# /etc/lirc/hardware.conf
		#
		# Arguments which will be used when launching lircd
		LIRCD_ARGS="--uinput --listen"
		
		#Don't start lircmd even if there seems to be a good config file
		#START_LIRCMD=false
		 
		#Don't start irexec, even if a good config file seems to exist.
		#START_IREXEC=false
		 
		#Try to load appropriate kernel modules
		LOAD_MODULES=true
		 
		# Run "lircd --driver=help" for a list of supported drivers.
		DRIVER="default"
		 
		# usually /dev/lirc0 is the correct setting for systems using udev 
		DEVICE="/dev/lirc0"
		MODULES=""
		 
		# Default configuration files for your hardware if any
		LIRCD_CONF=""
		LIRCMD_CONF=""
		```
		
	5. 	录制红外信号
		
		```
		irrecord  -f -d /dev/lirc0 ~/fanraw.conf
		```
		
		以上命令会将根据你空调遥控的红外信号生成一个模板文件出来。文件名为fanrow.conf
		
		录制电视、风扇的遥控和空调不一样。空调信号需要采用Row方式录制，具体实现是完完全全记录下红外信号的高低电平以及对应的时间。
		
		操作步骤：输入命令后，会有说明文字出现，这个时候按以下回车，会出现第二段介绍，再按回车，开始录制。录制过程就是不停的按空调遥控器的按钮，随意按，目的是为了获取信号的gap参数。理论上每次按空调遥控器后，控制台会多一个点，当出现足够的点后(大约需要40个点)，系统提示输入你希望的KEY NAME，输入完毕后回车，然后按对应的空调遥控器的键。此时，模板文件里就会保存这个KEY NAME的信息。输入KEY NAME以及采集遥控器信号的过程可以循环多次，目的是方便用户一次采集多个KEY NAME。最后多按几次回车结束过程。
		
		需要注意的是，KEY NAME是lirc预设的，你可以以下通过命令来获取有哪些KEY NAME可用。
		
		```
		 irrecord --list-namespace | grep -i key_ 
		```
		
	6. 录制完后，你录制的文件应该类似以下：

		```
		# Please make this file available to others
		# by sending it to <lirc@bartelmus.de>
		#
		# this config file was automatically generated
		# using lirc-0.9.0-pre1(default) on Thu Jul 20 23:52:11 2017
		#
		# contributed by
		#
		# brand:                       /home/pi/fanraw.conf
		# model no. of remote control:
		# devices being controlled by this remote:
		#
		
		begin remote
		
		  name  /home/pi/fanraw.conf
		  flags RAW_CODES
		  eps            30
		  aeps          100
		
		  gap          3410
		
		      begin raw_codes
		
		          name KEY_POWER
				21340 3400 3413 
		
		      end raw_codes
		
		end remote
		```
		
		这个是生成的一个模板，其中eps、aeps为预设的值，gap是通过你多次按遥控器，程序计算出的。
		
		`flags RAW_CODES`表示是row模式。`name KEY_POWER`表示其后的code用于表示KEY_POWER这个命令。
		
	7. 录制开机信号

		因为我的目的是为了开空调，所以我录制开空调的信号。(要注意，开空调的信号也会包含温度、风力等信息，所以你录制的一定是你所期望的空调温度和风力等设置)
		
		输入命令
		
		```
		sudo mode2 -d /dev/lirc0
		```
		然后对着红外接收头按开机键。
		
		会有很多信息:
		
		```
		space 4960669
		pulse 2697
		……
		pulse 2697
		```
		将第一行删去，然后仅仅取出后面的数字，并用空格分隔。
		
		处理后的结果如下：
		
		```
		3400 3413 420 1291 397 384 461 1291 397 1296 392 386 458 386 459 1282 398 400 453 1292 395 387 458 1292 398 390 454 1291 396 1284 397 1300 397 386 459 383 461 1292 396 385 464 381 459 1284 396 1302 396 384 459 1293 397 383 461 1291 398 384 465 1279 396 401 453 385 459 385 459 1293 395 1294 396 1292 397 1288 392 1301 396 382 463 379 466 385 459 385 459 385 460 1284 396 1305 393 383 462 385 458 385 460 386 459 1292 397 1283 397 1301 396 1297 393 1292 396 1293 397 383 461 384 461 365 470 403 452 383 460 385 461 384 460 384 460 385 461 385 458 387 459 385 459 385 460 384 470 375 460 385 460 386 459 384 460 385 460 384 461 384 460 385 460 385 460 384 460 385 461 383 461 384 461 385 460 383 461 385 461 383 466 379 461 383 461 383 461 385 461 384 460 385 460 385 460 1292 397 385 461 382 461 367 474 398 451 385 461 1292 396 1293 397 385 460 384 460 1283 398 401 452 384 467
		```
	
	8. 将以上结果放在`name KEY_POWER`下面，最终文件如下：

		```
		# Please make this file available to others
		# by sending it to <lirc@bartelmus.de>
		#
		# this config file was automatically generated
		# using lirc-0.9.0-pre1(default) on Thu Jul 20 23:52:11 2017
		#
		# contributed by
		#
		# brand:                       /home/pi/fanraw.conf
		# model no. of remote control:
		# devices being controlled by this remote:
		#
		
		begin remote
		
		  name  /home/pi/fanraw.conf
		  flags RAW_CODES
		  eps            30
		  aeps          100
		
		  gap          3410
		
		      begin raw_codes
		
		          name KEY_POWER
				3400 3413 420 1291 397 384 461 1291 397 1296 392 386 458 386 459 1282 398 400 453 1292 395 387 458 1292 398 390 454 1291 396 1284 397 1300 397 386 459 383 461 1292 396 385 464 381 459 1284 396 1302 396 384 459 1293 397 383 461 1291 398 384 465 1279 396 401 453 385 459 385 459 1293 395 1294 396 1292 397 1288 392 1301 396 382 463 379 466 385 459 385 459 385 460 1284 396 1305 393 383 462 385 458 385 460 386 459 1292 397 1283 397 1301 396 1297 393 1292 396 1293 397 383 461 384 461 365 470 403 452 383 460 385 461 384 460 384 460 385 461 385 458 387 459 385 459 385 460 384 470 375 460 385 460 386 459 384 460 385 460 384 461 384 460 385 460 385 460 384 460 385 461 383 461 384 461 385 460 383 461 385 461 383 466 379 461 383 461 383 461 385 461 384 460 385 460 385 460 1292 397 385 461 382 461 367 474 398 451 385 461 1292 396 1293 397 385 460 384 460 1283 398 401 452 384 467
		
		      end raw_codes
		
		end remote
		```
		
	9. 使用irsend命令发送红外信息

		```
		irsend SEND_ONCE /home/pi/fanraw.conf KEY_POWER
		```	
		
		如果没问题的话，你会看到红外发射模块的灯泡闪出红光，空调发出滴的一声。
		
		这里我遇到一个坑：在使用irsend命令的时候报错，报错信息为无法连接对应设备。
	
		问题的原因是没有开启lirc的服务。解决方案是使用如下命令
		
		```
		sudo lircd -d /dev/lirc0
		```
		
		可以配置一下开机启动的时候就执行这个命令。比如在/etc/rc.local中添加这条命令。
	
	至此，你应该能用树莓派发出开机信号来开启空调了	
### HomeKit控制

	这一部分是调研时间最久的，因为不太了解智能家居的平台，花了比较久的时间才分清楚Homebridge相关软件之间的关系。针对我现在的情况：1. 没有使用智能硬件 2. 没有现成的插件可用 3. js不熟，写插件的话，要看资料的多，花费的时间长。于是，我选择HAP-NodeJs，在别人的例子上改改。
	
	HAP-NodeJS的使用可以参见这篇文档：[如何用Siri与树莓派“交互”](http://www.freebuf.com/geek/119719.html),总结后步骤如下：
	
	1. 安装相关环境
		
		安装依赖
		
		```
		# apt-get install avahi-daemon avahi-discover libnss-mdns libavahi-compat-libdnssd-dev  build-essential -y
# service avahi-daemon start
		```
		
		添加nodejs源
		
		```
		curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash -
		```
		
		安装nodejs
		
		```
		apt-get install nodejs
		```
		
		安装node-gyp
		
		```
		sudo npm install -g node-gyp
		```
		
		安装python-shell,用来调用本地的py脚本
		
		```
		npm install python-shell
		```
		
		下载HAP-NodeJS
		
		```
		git clone https://github.com/KhaosT/HAP-NodeJS.git
		```
		
		然后进入HAP-NodeJS文件夹。运行程序
		
		```
		node Core.js
		```
		
		这个时候会报错：(引用一下教程里的图)
		
		![cord.js报错](http://image.3001.net/images/20161110/14787778002702.png)
		
		这个时候就安装缺少的依赖包
		
		```
		npm install node-persist
		```
		
		一直循环这个过程，直到不报错。这个时候运行Core.js就会启动HAP-NodeJS的服务端，这个时候你的iOS-家庭就能搜到设备了。打开iOS中的家庭app。点击添加配件。
		
		![搜索设备](/img/搜索设备.png)
		
	2. 添加设备逻辑
		
		因为我的需求是开关空调，我就套用了灯开关的模型，只要开关即可。
		
		首先添加发射红外信号的执行脚本，这个地方绕了一圈，用js的python-shell包调用了python脚本，然后用python脚本调用shell发送红外信号，其实可以直接用js调用shell语句的。之所以教程里用python是因为python可以调用树莓派的GPIO引脚，我现在套用了脚本，就懒得做太多改动了。
		
		在HAP-NodeJS文件夹下创建文件夹python，新建文件light1.py.
		
		```
		import os
		os.system('irsend SEND_ONCE /home/pi/fanraw.conf KEY_POWER')
		```
		这个就是调用了irsend命令发送红外信号。
		
		在accessories文件夹下创建LivingLight_accessory.js
		
		```
		//Light_accessory.js
		
		var PythonShell = require('python-shell');
		// HomeKit types required
		var types = require("./types.js")
		var exports = module.exports = {};
		
		var execute = function(accessory,characteristic,value){ console.log("executed accessory: " + accessory + ", and characteristic: " + characteristic + ", with value: " +  value + "."); }
		
		exports.accessory = {
		  displayName: "Living Light",
		  username: "1A:5B:3C:4A:5E:FF",
		  pincode: "031-45-154",
		  services: [{
		    sType: types.ACCESSORY_INFORMATION_STYPE,
		    characteristics: [{
		        cType: types.NAME_CTYPE,
		        onUpdate: null,
		        perms: ["pr"],
		        format: "string",
		        initialValue: "Living Light",
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Bla",
		        designedMaxLength: 255
		    },{
		        cType: types.MANUFACTURER_CTYPE,
		        onUpdate: null,
		        perms: ["pr"],
		        format: "string",
		        initialValue: "Oltica",
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Bla",
		        designedMaxLength: 255
		    },{
		        cType: types.MODEL_CTYPE,
		        onUpdate: null,
		        perms: ["pr"],
		        format: "string",
		        initialValue: "Rev-1",
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Bla",
		        designedMaxLength: 255
		    },{
		        cType: types.SERIAL_NUMBER_CTYPE,
		        onUpdate: null,
		        perms: ["pr"],
		        format: "string",
		        initialValue: "A1S2NASF88EW",
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Bla",
		        designedMaxLength: 255
		    },{
		        cType: types.IDENTIFY_CTYPE,
		        onUpdate: null,
		        perms: ["pw"],
		        format: "bool",
		        initialValue: false,
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Identify Accessory",
		        designedMaxLength: 1
		    }]
		  },{
		    sType: types.LIGHTBULB_STYPE,
		    characteristics: [{
		        cType: types.NAME_CTYPE,
		        onUpdate: null,
		        perms: ["pr"],
		        format: "string",
		        initialValue: "Light 1 Light Service",
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Bla",
		        designedMaxLength: 255
		    },{
		        cType: types.POWER_STATE_CTYPE,
		        onUpdate: function(value)
		    {
		            console.log("Change:",value);
		            if (value) {
		            PythonShell.run('/python/light1.py', function (err) {
		                   console.log('Light1 On Success');
		            });
		            }
		        },
		        perms: ["pw","pr","ev"],
		        format: "bool",
		        initialValue: false,
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Turn On the Light",
		        designedMaxLength: 1
		    },{
		        cType: types.HUE_CTYPE,
		        onUpdate: function(value) { console.log("Change:",value); execute("Test Accessory 1", "Light - Hue", value); },
		        perms: ["pw","pr","ev"],
		        format: "int",
		        initialValue: 0,
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Doesn’t actually adjust Hue of Light",
		        designedMinValue: 0,
		        designedMaxValue: 360,
		        designedMinStep: 1,
		        unit: "arcdegrees"
		    },{
		        cType: types.BRIGHTNESS_CTYPE,
		        onUpdate: function(value) { console.log("Change:",value); execute("Test Accessory 1", "Light - Brightness", value); },
		        perms: ["pw","pr","ev"],
		        format: "int",
		        initialValue: 0,
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Doesn’t actually adjust Brightness of Light",
		        designedMinValue: 0,
		        designedMaxValue: 100,
		        designedMinStep: 1,
		        unit: "%"
		    },{
		        cType: types.SATURATION_CTYPE,
		        onUpdate: function(value) { console.log("Change:",value); execute("Test Accessory 1", "Light - Saturation", value); },
		        perms: ["pw","pr","ev"],
		        format: "int",
		        initialValue: 0,
		        supportEvents: false,
		        supportBonjour: false,
		        manfDescription: "Doesn’t actually adjust Saturation of Light",
		        designedMinValue: 0,
		        designedMaxValue: 100,
		        designedMinStep: 1,
		        unit: "%"
		    }]
		  }]
		}
		```
		
		其中的核心逻辑是
		
		```
		cType: types.POWER_STATE_CTYPE,
		onUpdate: function(value)
		{
		    console.log("Change:",value);
		    if (value) {
		    PythonShell.run('/python/light1.py', function (err) {
		           console.log('Light1 On Success');
		    });
		    }
		}
		```
		
		当改动的类型为types.POWER_STATE_CTYPE时，会根据value是开还是关，执行不同的操作。此处我的操作是当结果为开的时候，发开机的信号。关的时候，不处理。
		
		pin码为"031-45-154"，后续会用到
		
	3. iOS添加设备

		在iOS的家庭app中，添加设备
		
		![添加设备1](/img/添加设备1.png)
		
		选择设备，刚才脚本对应的设备为Living Light
		
		![添加设备1](/img/添加设备2.png)
		
		输入pin码 刚才配置的pin码是031-45-154
		![添加设备1](/img/添加设备3.png)
		
		添加成功之后，点击家庭app中 Living Light对应的开关，可以看到树莓派上，红外发射模块发出了红光。那是胜利的光芒！
		
	4. Siri控制。

		用Siri开空调吧少年！
		
		需要注意的是，在用Siri开空调的时候，一定要给刚才的设备取一个合适的名字。比如我给刚才的Living Light取名为空调。同时把它分配在"卧室"这个房间。
		
		![SIRI](/img/SIRI.png)
	
### 开启远程HomeKit支持

HomeKit默认情况下需要iOS与设备在同一局域网下才可以进行控制。如果想在外网环境控制设备，需要设定一个控制中枢，控制中枢需要iPad或者appleTV。(我当然选择用iPad→_→)详情参见苹果官方文档:[自动化和远程访问 HomeKit 配件](https://support.apple.com/zh-cn/HT207057)

1. 设备

	一台iPad用于作为控制中枢，一台有家庭app的iOS设备，用于远程开启空调。两台设备登录同一个appleID
	
2. 接入设备

	在iPad上加入刚才的Living Light设备，取名为空调。
	
3. 设置iPad为中枢

	前往“设置”>“家庭”并打开“将此 iPad 用作家庭中枢”
	
	坑： 在iPad的设置中找不到家庭。
	
	解决方案：重启大法
	
4. 测试

	iPhone关掉Wifi，用4G，然后再家庭app中，控制空调的开关。
	
5. 添加其他人的权限。

	在家庭app中，可以邀请其他人作为家人，并且给予不同的权限。
	
至此，已经实现远程控制空调开关的目的。


## 结果

亲自尝试：周末出门吃饭，在回家前的一个小时，通过家庭app开启空调。回到家后，空调已经开启，室内温度已经很舒适。


## 更好的方案

红外遥控的智能化有一个更好的解决方案：BroadLink万能遥控 + Homebridge。

BroadLink的万能遥控是一个类似于机顶盒的东西，内置了大量设备的红外码库，对于无码库的部分机型，你也可以通过它的程序来进行录制操作(和我用红外接收模块录信号一样的原理)。HomeBrige有官方提供的BroadLink插件，所以使用BroadLink + Homebridge是一个更好的方案。其中，BroadLink也有其他的替代物品，比如某米的遥控器。

另外，Homebridge不是必须，可以使用HomeAsistant等其他平台的解决方案。但是考虑到安全性，我更建议接入支持HomeKit的平台。HomeAsistant等平台支持app和Web端的控制，但是如果你需要远程控制设备时，你的HomeAsistant需要暴露在外网下，这个时候的安全性就很难保证了。HomeKit仅支持iOS设备控制，同时，你的控制中枢没有主动暴露在外网环境下，相对而言更安全。

## 感受

* 现在智能家居没有一个统一的方案，从我在网上查资料的过程中，给我的感觉就是乱。
* 资料不多，中文资料少，质量驳杂。比如Homebridge，官方的Wiki居然写道：没有编写plugin的教程，给你几个example你自己学着写。手动微笑表情。
* 智能家居不是刚需，但是会解决平时生活中的一些痛点，接触了之后觉得是一种未来的趋势，之后智能家居的规范、控制中枢的市场，都可能会是新的战场。
* 硬件真好玩真好玩，智能家居真好玩真好玩。
