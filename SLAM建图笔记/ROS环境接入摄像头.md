# ROS环境接入摄像头

基于libuvc库搭建

https://wiki.ros.org/libuvc_camera



## 1.判别摄像头是否为UVC摄像头

通过以下指令检查摄像头是否已经连接

```bash
lsusb
```

通过以下指令查看更详细的参数

```bash
lsusb -v
```

通过以下命令来验证Linux内核是否已将其识别为UVC设备：

```bash
dmesg | grep uvc
```

如果看到类似 `uvcvideo: Found UVC 1.00 device <你的设备名>` 的记录，那就完全确认了。



## 2.安装软件包

下载对应库

```
sudo apt-get install ros-noetic-libuvc-camera
```



## 3.配置对USB设备的访问权限

首先通过 `lsusb -v`查看设备的idVendor和 idProduct 

> [!NOTE]
>
> 使用**Terminator终端终结者**可能产生问题：
>
> 终端输出内容太长导致无法查看部分内容
>
> 解决方案：使用**系统自带的终端**

翻阅输出内容，找到有`Video`字样的那一块USB设备

> [!IMPORTANT]
>
> xxxxxxxxxx <launch>    <!-- 设置地图的配置文件 -->    <arg name="map" default="nav.yaml" />    <!-- 运行地图服务器，并且加载设置的地图-->    <node name="map_server" pkg="map_server" type="map_server" args="$(find mycar_nav)/map/$(arg map)"/>    <!-- 启动AMCL节点 -->    <include file="$(find mycar_nav)/launch/amcl.launch" />​    <!-- 运行move_base节点 -->    <include file="$(find mycar_nav)/launch/path.launch" />    <!-- 运行rviz -->    <node pkg="rviz" type="rviz" name="rviz" args="-d $(find mycar_nav)/rviz/nav.rviz" />​</launch>xml
>
> 解决方案：通过插拔USB摄像头 检查前后增加的Video设备

e.g.

```
Bus 003 Device 002: ID 0408:1020 Quanta Computer, Inc. hm1091_techfront
Couldn't open device, some information will be missing
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass          239 Miscellaneous Device
  bDeviceSubClass         2 
  bDeviceProtocol         1 Interface Association
  bMaxPacketSize0        64
  idVendor           0x0408 Quanta Computer, Inc. <<<<<idVendor为0408 不要复制公司名称 去掉0x
  idProduct          0x1020 <<<<<<<<<<<<<<<<<<<<<<<<<<<idProduct为1020
  bcdDevice            0.13
  iManufacturer           1 
  iProduct                2 
  iSerial                 0 
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength       0x025f
    bNumInterfaces          2
    bConfigurationValue     1
    iConfiguration          0 
    bmAttributes         0x80
      (Bus Powered)
    MaxPower              500mA
    Interface Association:
      bLength                 8
      bDescriptorType        11
      bFirstInterface         0
      bInterfaceCount         2
      bFunctionClass         14 Video
      bFunctionSubClass       3 Video Interface Collection <<<<<<Video设备
      bFunctionProtocol       0 
      iFunction               4 
省略后面的内容....
```

**创建一个新的udev规则文件**：

```bash
sudo nano /etc/udev/rules.d/99-uvc-camera.rules
```

Ctrl+Shift+V 粘贴

```
SUBSYSTEM=="usb", ATTR{idVendor}=="<修改为你的idVendor>", ATTR{idProduct}=="<修改为你的idProduct>", MODE="0666", GROUP="plugdev"
```

Ctrl+O保存

Enter 确认

Ctrl+X退出

**重新加载udev规则并重新插拔设备**：

```bash
sudo udevadm control --reload-rules
sudo udevadm trigger
```



## 4.检查支持的输出格式

安装工具

```bash
sudo apt update
sudo apt install v4l-utils
```

查看输出格式

```bash
v4l2-ctl --list-formats-ext
```

会输出以下内容

```bash
zhengtuo@zhengtuo-1:~$ v4l2-ctl --list-formats-ext
ioctl: VIDIOC_ENUM_FMT
	Type: Video Capture

	[0]: 'MJPG' (Motion-JPEG, compressed)
		Size: Discrete 1280x720
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 960x540
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 848x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 640x360
			Interval: Discrete 0.033s (30.000 fps)
	[1]: 'YUYV' (YUYV 4:2:2)
		Size: Discrete 640x480
			Interval: Discrete 0.033s (30.000 fps)
		Size: Discrete 640x360
			Interval: Discrete 0.033s (30.000 fps)

```

可以看到支持这些`压缩格式` `分辨率` `帧率`

> [!IMPORTANT]
>
> 笔记本用户需要注意默认`v4l2-ctl --list-formats-ext`会输出笔记本自带的摄像头
>
> 解决方案
>
> ```
> v4l2-ctl --list-devices #列出可用摄像头
> ```
>
> 会输出类似以下内容
>
> ```
> Integrated Camera: Integrated C (usb-0000:00:14.0-8):
> 	/dev/video0
> 	/dev/video1
> 
> HD Pro Webcam C920 (usb-0000:01:00.0-1.2):
> 	/dev/video2
> 	/dev/video3
> 	/dev/media0
> ```
>
> 找到外接摄像头的设备节点后（例如 `/dev/video2`），使用 `-d` 参数来指定设备：
>
> ```
> v4l2-ctl -d /dev/video2 --list-formats-ext
> ```
>
> 请将 `/dev/video2` 替换为你实际查到的外接摄像头设备节点



## 5.屏蔽冲突驱动（可选）

官方Wiki讲道：

You may need to disable your operating system's builtin USB video or audio drivers. On Linux, the `snd-usb-audio` and `uvcvideo` modules conflict with libuvc. Try unloading them with `sudo rmmod snd-usb-audio; sudo rmmod uvcvideo` and consider blacklisting them -- e.g., add the lines `blacklist uvcvideo` and `blacklist snd-usb-audio` to an `/etc/modprobe.d/uvc.conf` file. (Applications that don't use libuvc will be unable to stream from the camera.)

您可能需要禁用操作系统内置的 USB 视频或音频驱动程序。在 Linux 上，`snd-usb-audio`和`uvcvideo模块与 libuvc 冲突。请尝试使用``sudo rmmod snd-usb-audio 和 sudo rmmod uvcvideo`卸载它们，并考虑将它们列入黑名单——例如，`将 blacklist uvcvideo`和`blacklist snd-usb-audio`行添加到`/etc/modprobe.d/uvc.conf`文件。（不使用 libuvc 的应用程序将无法从摄像头进行流式传输。）

我的电脑疑似不屏蔽也能用



## 6.创建launch文件

在工作空间下的launch文件夹创建uvc_cam.launch (名称随意)

```xml
<launch>
  <!-- 创建一个名为'camera'的命名空间，所有节点和参数将位于此命名空间下 -->
  <group ns="camera">
    <!-- 启动libuvc_camera包的相机节点，节点名为mycam -->
    <node pkg="libuvc_camera" type="camera_node" name="mycam">
      <!-- 用于识别相机的参数 -->
      
      <!-- 厂商ID（十六进制格式），此处为0x09da -->
      <param name="vendor" value="0x09da"/>
      <!-- 产品ID（十六进制格式），此处为0x2700 -->
      <param name="product" value="0x2700"/>
      <!-- 相机序列号，如果为空则匹配任意序列号 -->
      <param name="serial" value=""/>
      <!-- 当有多个相机匹配时，选择索引为0的相机（第一个匹配的相机） -->
      <param name="index" value="0"/>

      <!-- 图像尺寸设置 -->
      <param name="width" value="1920"/>  <!-- 图像宽度：1920像素 -->
      <param name="height" value="1080"/> <!-- 图像高度：1080像素 -->
      
      <!-- 视频模式设置：可选择yuyv/nv12/mjpeg/uncompressed等相机支持的未压缩格式 -->
      <param name="video_mode" value="mjpeg"/> <!-- 使用MJPEG格式 -->
      <param name="frame_rate" value="30"/>    <!-- 帧率：30fps -->

      <!-- 时间戳方法：设置为帧开始时间 -->
      <param name="timestamp_method" value="start"/>
      <!-- 相机标定信息文件路径，需要用户提供相应的YAML文件 -->
      <param name="camera_info_url" value="file:///tmp/cam.yaml"/>

      <!-- 自动曝光设置：3表示使用光圈优先自动曝光模式 -->
      <param name="auto_exposure" value="3"/>
      <!-- 自动白平衡设置：false表示关闭自动白平衡 -->
      <param name="auto_white_balance" value="false"/>
    </node>
  </group>
</launch>
```



## 7.运行

```bash
source devel/setup.bash
roslaunch ros_uart_protocol uvc_cam.launch #注意修改工作空间名称
```



## 8.查看输出

| 查看方式                  | 命令                                                         | 特点                                                 |
| :------------------------ | :----------------------------------------------------------- | :--------------------------------------------------- |
| **rqt_image_view (推荐)** | `rosrun rqt_image_view rqt_image_view`                       | **图形化界面**，可选择多个话题，**灵活方便**         |
| **image_view 节点**       | `rosrun image_view image_view image:=/camera/image_raw`      | 打开一个固定窗口显示指定话题的图像                   |
| **在 rviz 中查看**        | `rosrun rviz rviz` 然后添加 `Image` 显示类型，并设置话题为 `/camera/image_raw` | 功能强大的可视化工具，通常用于综合显示多种传感器数据 |

```bash
rosrun rqt_image_view rqt_image_view
```

话题：

```bash
zhengtuo@zhengtuo-1:~$ rostopic list
/camera/camera_info
/camera/image_raw
/camera/image_raw/compressed
/camera/image_raw/compressed/parameter_descriptions
/camera/image_raw/compressed/parameter_updates
/camera/image_raw/compressedDepth
/camera/image_raw/compressedDepth/parameter_descriptions
/camera/image_raw/compressedDepth/parameter_updates
/camera/image_raw/theora
/camera/image_raw/theora/parameter_descriptions
/camera/image_raw/theora/parameter_updates
/camera/mycam/parameter_descriptions
/camera/mycam/parameter_updates
/rosout
/rosout_agg
#普通相机是看不到/compressedDepth的 这很正常
```



## 9.Bug Fix



launch报错`[ERROR] [1756267714.231273295]: Permission denied opening /dev/bus/usb/001/038`

原因：访问权限出错 

解决方案：检查3



launch警告`[WARN] [1756267714.231483270]: Camera calibration file /tmp/cam.yaml not found.`

原因 ：这个警告关于相机校准文件缺失，只会影响图像的几何校正（比如去除畸变）。

解决方案：将launch文件这一行注释掉

```xml
<param name="camera_info_url" value="file:///tmp/cam.yaml"/>
```



launch报错`[ERROR] [1756268093.388780130]: check video_mode/width/height/frame_rate are available`

原因 ：`分辨率` `帧率` `视频模式`无效

解决方案：`分辨率` `帧率` `视频模式`要根据 4.检查支持的输出格式 来设置 否则报错

