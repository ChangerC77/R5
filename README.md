# VR + ARX R5 bimanual teleoperation
## Reference
1. 00-readme/00-配置CAN手册.pdf
2. 00-readme/05-R5-ROS2-aloha.pdf
## Hardware Connection
### ARX R5
<img src='img/15.png' width="70%" >

1. 电源线连接
2. CAN转USB， 白色的`type-c`转`USB`线，尽量用原装线，否则可能导致无法连接

### VR
1. VR串口通讯设备
2. 1根白色的双头type-c线和一根白色的type-c线转usb口

#### charge
<img src='img/14.png' width="70%" >

#### communication
see `4. VR connection` for more details

## 1. Environment Setting
### ROS2
system requirement: `Ubuntu 22.04 + ROS humble`
```
wget http://fishros.com/install -O fishros && . fishros
```
### Conda
```
conda create -n arx python=3.8 
```
```
conda activate arx # activate environment
pip install empy==3.3.4 catkin_pkg lark
python3 -m pip install --upgrade numpy # install the numpy
```
#### autoactivate conda environment
- bash
```
echo "conda activate arxr5" >> ~/.bashrc
source ~/.bashrc
```
- zsh
```
echo "conda activate arxr5" >> ~/.zshrc
source ~/.zshrc
python --version # check python version whether 3.8 or not
```
## 2. Download
```
mkdir ~/ARX && cd ~/ARX
git clone https://github.com/ChangerC77/R5.git 
```
## 3. 配置CAN
### 1. 环境依赖
```
sudo apt install can-utils
sudo apt install net-tools
```
### 2. ARX_CAN文件夹
```
├── arx_can
│   ├── arx_can0.sh
│   ├── arx_can1.sh
│   ├── arx_can2.sh
│   ├── arx_can3.sh
│   └── arx_can5.sh
├── arx_can.rules
├── can.sh
├── search.sh
└── set.sh

1 directory, 9 files
```
这些文件是配置can和打开can的，用来让SDK通过can与机器人通信。
```
整个过程分4步
1、执行search.sh
2、修改arx_can.rules
3、执行set.sh
4、执行can.sh
```
### 3. C-VR双臂CAN设置
在使用两台臂（VR+两从臂）进行数据采集时，需要用到“arx_can.rules”文件中的can1 和 can3。
#### 1. 执行search.sh
搜索CAN设备（黑色can板）的ID，注意要把USB插入电脑才能搜索到，且一次只能插入一个USB!!! 除键鼠
```
cd ~/ARX/R5/ARX_CAN
./search.sh
```
##### 1. left arm
先将左臂与电脑连接，执行`search.sh`配置`arxcan1`
```
cd ~/ARX/ARX_R5/ARX_CAN
./search.sh # 若无法运行则执行：sh search.sh
```
然后终端会显示（不同的can设备显示的ID不同，下图的ID号只是一个例子）​
```
lsusb
```
output
```
    ATTRS{serial}=="207B32AE5052"
    ATTRS{serial}=="0000:00:14.0"
```
##### 2. right arm
拔出左臂usb，将右臂与电脑连接，执行`search.sh` 设置右臂`arxcan3` 的配置
```
cd ~/ARX/ARX_R5/ARX_CAN
./search.sh # 若无法运行则执行：sh search.sh
```
output
```
    ATTRS{serial}=="206A32B75052"
    ATTRS{serial}=="0000:00:14.0"
```
#### 2. arx_can.rules
```
sudo vim arx_can.rules
```
> 左右臂分别对应arxcan1和arxcan3
```
SUBSYSTEM=="tty", ATTRS{idVendor}=="16d0", ATTRS{idProduct}=="117e", ATTRS{serial}=="207B32AE5052", SYMLINK+="arxcan1"
SUBSYSTEM=="tty", ATTRS{idVendor}=="16d0", ATTRS{idProduct}=="117e", ATTRS{serial}=="206A32B75052", SYMLINK+="arxcan3"
```
#### 3. set.sh
```
cd ~/ARX/R5/ARX_CAN
./set.sh
```
输入密码后出现类似窗口：只要没报错就行。
```
root@pc-MS-7D89:/home/pc/ARX/ARX_R5/ARX_CAN# 
```
<img src='img/1.png'>

`以上操作仅在第一次运行程序前进行，只要can设备不更改，以后及无需再次配置。`
#### 4. set.sh

<img src='img/2.png'>

##### left arm
```
~/ARX/R5/ARX_CAN/arx_can
./arx_can1.sh
```
##### right arm
```
~/ARX/R5/ARX_CAN/arx_can
./arx_can3.sh
```
如果关掉上述窗口，can口不会关闭。
如果不关，这些脚本会有自动重连功能，可以自己拔掉usb再插上试试。
# 4. VR connection
<img src='img/12.png' width="70%">
<img src='img/13.png' width="70%">

VR的连接方式如上图所示
连接后要看VR串口线是否亮起

<img src="img/16.png" width="70%">

# 5. ARX_VR_SDK
## 1. build
### 01make.sh
```
cd ~/ARX/R5/00-sh/ROS2
./01make.sh
```
output 1
```
Starting >>> arm_control
Starting >>> arx5_arm_msg
--- stderr: arm_control                                              
CMake Warning:
  Manually-specified variables were not used by the project:

    CATKIN_INSTALL_INTO_PREFIX_ROOT


---
Finished <<< arm_control [2.98s]
Finished <<< arx5_arm_msg [3.05s]                  
Starting >>> arx_r5_controller
Finished <<< arx_r5_controller [7.88s]                      

Summary: 3 packages finished [11.2s]
  1 package had stderr output: arm_control
```
output 2
```
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成                 
ros-humble-serial-driver 已经是最新版 (1.2.0-2jammy.20250325.213120)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 7 个软件包未被升级。
正在读取软件包列表... 完成
正在分析软件包的依赖关系树... 完成
正在读取状态信息... 完成                 
ros-humble-asio-cmake-module 已经是最新版 (1.2.0-2jammy.20250325.172400)。
升级了 0 个软件包，新安装了 0 个软件包，要卸载 0 个软件包，有 7 个软件包未被升级。
crw-rw-rw- 1 root dialout 166, 0  4月 11 11:31 /dev/ttyACM0
crw-rw-rw- 1 root dialout 166, 1  4月 11 11:31 /dev/ttyACM1
udev 规则文件已存在: /etc/udev/rules.d/99-ttyACM.rules
重新加载 udev 规则...
确保当前用户已被添加到 dialout 组...
操作完成。请重新登录，或运行 'newgrp dialout' 使更改生效。
您可以通过以下命令验证组成员信息：
groups tars
```
全部子窗口编译结束后
### 01make.sh
```
./02make.sh
```
output
```
Starting >>> arm_control
Finished <<< arm_control [2.32s]                    
Starting >>> serial_port
Finished <<< serial_port [3.26s]                     

Summary: 2 packages finished [5.71s]
```
等待编译结束，并无报错，关闭终端即可
### 2. VR程序启动
<font color=#FF0000 >启动程序前连接VR和双臂到电脑</font>
```
~/ARX/R5/00-sh/ROS2
./05double_vr.sh
```

#### output 1
```
CAN 接口 can1 正常工作
```
#### output 2
```
CAN 接口 can3 正常工作
```
如果`CAN`口不正常，频繁掉线，那么重新执行`ARX_CAN`的部分
+ left arm
```
~/ARX/R5/ARX_CAN/arx_can
./arx_can1.sh
```
+ right arm
```
~/ARX/R5/ARX_CAN/arx_can
./arx_can3.sh
```
#### output 3
```
[INFO] [1744343304.435289966] [IoContext::IoContext]: Thread(s) Created: 32
DEV/tty NAME: /dev/serial/by-id/usb-Openlight_Labs_CANable2_b158aa7_github.com_normaldotcom_canable2.git_207B32AE5052-if00 -> ../../ttyACM1
DEV/tty NAME: /dev/serial/by-id/usb-Openlight_Labs_CANable2_b158aa7_github.com_normaldotcom_canable2.git_206A32B75052-if00 -> ../../ttyACM0
DEV/tty NAME: /dev/serial/by-id/usb-1a86_USB_Single_Serial_5909045851-if00 -> ../../ttyACM2
Matching device: /dev/ttyACM2
device choose: /dev/ttyACM2
[INFO] [1744343304.436588324] [serial_port_node]: /dev/ttyACM2 is opened.
```
#### output 4
```
Traceback (most recent call last):
  File "/opt/ros/humble/local/lib/python3.10/dist-packages/rosidl_generator_py/import_type_support_impl.py", line 46, in import_type_support
    return importlib.import_module(module_name, package=pkg_name)
  File "/usr/lib/python3.10/importlib/__init__.py", line 126, in import_module
    return _bootstrap._gcd_import(name[level:], package, level)
  File "<frozen importlib._bootstrap>", line 1050, in _gcd_import
  File "<frozen importlib._bootstrap>", line 1027, in _find_and_load
  File "<frozen importlib._bootstrap>", line 1004, in _find_and_load_unlocked
ModuleNotFoundError: No module named 'arm_control.arm_control_s__rosidl_typesupport_c'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/opt/ros/humble/bin/ros2", line 33, in <module>
    sys.exit(load_entry_point('ros2cli==0.18.12', 'console_scripts', 'ros2')())
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2cli/cli.py", line 91, in main
    rc = extension.main(parser=parser, args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2topic/command/topic.py", line 41, in main
    return extension.main(args=args)
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2topic/verb/echo.py", line 235, in main
    self.subscribe_and_spin(
  File "/opt/ros/humble/lib/python3.10/site-packages/ros2topic/verb/echo.py", line 258, in subscribe_and_spin
    node.create_subscription(
  File "/opt/ros/humble/local/lib/python3.10/dist-packages/rclpy/node.py", line 1374, in create_subscription
    check_is_valid_msg_type(msg_type)
  File "/opt/ros/humble/local/lib/python3.10/dist-packages/rclpy/type_support.py", line 35, in check_is_valid_msg_type
    check_for_type_support(msg_type)
  File "/opt/ros/humble/local/lib/python3.10/dist-packages/rclpy/type_support.py", line 29, in check_for_type_support
    msg_or_srv_type.__class__.__import_type_support__()
  File "/home/tars/ARX/R5/ARX_VR_SDK/ROS2/install/arm_control/lib/python3.8/site-packages/arm_control/msg/_pos_cmd.py", line 35, in __import_type_support__
    module = import_type_support('arm_control')
  File "/opt/ros/humble/local/lib/python3.10/dist-packages/rosidl_generator_py/import_type_support_impl.py", line 48, in import_type_support
    raise UnsupportedTypeSupport(pkg_name)
rosidl_generator_py.import_type_support_impl.UnsupportedTypeSupport: Could not import 'rosidl_typesupport_c' for package 'arm_control'
```
<font color=#FF0000 >output 4的输出不是报错</font>

# 6. VR APP
## 1. 进入APP
带上VR，用右手柄的A键点击打开右下角黑色图标的APP
## 2. 解锁
点进去后会提示

<img src='img/6.png'>

此时VR会一直发出锁定的声音
而后双手按住手柄的AB+ XY解锁
## 3. 整解锁
而后同时长按 左右摇杆键（直至语音提示控制器上线）机械臂完整解锁<img src='img/7.png'>
## 4. 复位
而后按`X`和`A`复位

<img src='img/8.png'>

## 5. 操作
按住扳机开始进行遥操作
<img src='img/9.png'>
<img src='img/10.png'>

## 6. 夹爪
按两侧按钮控制夹爪开合
<img src='img/11.png'>