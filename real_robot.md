
# 导航+抓取demo准备工作
demo功能说明：车臂机器人在室内环境建图后，导航+抓取的工作流程：自主导航到A点，触发anygrasp抓取位姿估计后执行抓取操作，导航到B点后执行放置操作。

工作站：连接底盘、机械臂、夹爪、相机等硬件。
服务器：主要跑算法和应用,需要GPU。

注意：工作站和服务器在同一个局域网下，需要设置服务器ip地址与工作站在同一网段：192.168.6.xxx。

## 远程登录工作站
```
ssh maiya@192.168.6.102
密码：a
```

## ros网络通信环境配置

工作站IP=192.168.6.102,假设服务器IP=192.168.6.103，以工作站为主机举例子：
### 主机设置
```
export ROS_HOSTNAME=192.168.6.102
export ROS_MASTER_URI=http://192.168.6.102:11311/
```
### 从机设置
```
export  HOSTNAME=192.168.6.103
export  ROS_MASTER_URI=http://192.168.6.102:11311/
```

# 建图
```
roslaunch dalu robot.launch
roslaunch dalu cartographer_mapping.launch use_sim_time:=false
roslaunch coffe_teleops keyboard_teleop.launch

#rosservice call /write_state "{filename: '<绝对路径存放地图信息>/<地图保存的名字>.pbstream'}"
rosservice call /write_state "{filename: '/home/maiya/maps/demo0425.pbstream'}"
```

# 导航
```
roslaunch dalu location.launch
```

可视化
```
rviz -d /home/maiya/demo_2d.rviz
```

注意：运行导航抓取之前利用键盘控制底盘旋转进行定位初始化。
```
roslaunch coffe_teleops keyboard_teleop.launch
```

<!-- # 抓取和放置
```
roslaunch realsense2_camera rs_aligned_depth.launch
rosrun robotiq_2f_gripper_control Robotiq2FGripperRtuNode.py /dev/ttyUSB1
roslaunch visual_servo uGraspControl.launch // zhijiezhua
``` -->

# 导航+抓取
```
roslaunch dalu location.launch
roslaunch dalu GripperControl.launch
roslaunch visual_servo tNavArmGripperControl.launch
```

# 手眼标定
```
roslaunch visual_servo uHandEyeCalibrate.launch
```
标定结果保存在cali_result.json文件中。

****

# 其他
## 单独编译visual_servo功能包
catkin_make -DCATKIN_WHITELIST_PACKAGES="visual_servo"

## 启动anygrasp命令
`cd ~/anaconda3/bin/ && source activate && conda activate anygrasp && cd /home/maiya/anygrasp_ws/src/anygrasp_sdk/grasp_live && demo_camera.sh`


## 搭建房子demo启动节点服务
### 启动夹爪动作服务
```
rosrun robotiq_2f_gripper_control Robotiq2FGripperRtuNode.py /dev/ttyUSB1
# roslaunch robotiq_2f_gripper_action_server robotiq_2f_gripper_action_server.launch

动作客户端调用测试：
rosrun robotiq_2f_gripper_action_server robotiq_2f_gripper_action_server_client_test  
```

### 启动UR3控制服务
```
roslaunch visual_servo tArmControlServer.launch

服务调用测试：
roslaunch visual_servo tArmControlClient.launch
```

## 真机实验
### 简单pick-place的demo

```shell
roslaunch visual_servo tNavArmGripperControl.launch
roslaunch visual_servo GripperControl.launch
```
启动AnyGrasp,即可对输出的pose执行动作

### 复杂pick-place

master is robot:

robot:
```shell
roslaunch dalu localization.launch
roslaunch coffe_teleops keyboard_teleop.launch
roslaunch visual_servo GripperControl.launch
roslaunch visual_servo uGraspControl.launch 
```

server:
```shell
./demo_test.sh
```
robot:
```shell
________monitor rostopic via [rostopic echo /detect_grasps/pose_grasps]
rostopic  pub /grasp_trigger std_msgs/UInt8 "data: 1"
```

