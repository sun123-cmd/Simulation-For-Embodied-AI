##### 说明 #####
在服务器上搭建了husky+ur5+robotiq_85+d435的gazebo仿真环境，目的是用于移动机器人在仓储环境的抓取任务仿真

基于此开源代码：QualiaT/husky_ur3_simulator: ROS(melodic and noetic) package for mobile manipulator simulation (github.com)
##### 代码 #####
# 在服务器上运行
# 切换到文件存放目录 
cd disk1/wangzixuan/catkin_ws

# 进入虚拟环境 robottest-env
conda activate robottest-env

# 启动机器人的gazebo仿真
# HRI_lab.world搭建了四个桌子和相关物品以及一些障碍物（3-14）
roslaunch husky_ur3_gazebo husky_ur3_HRI_lab.launch

# 启动机器人的rviz以及moveit
source devel/setup.bash
roslaunch husky_ur3_gripper_moveit_config Omni_control.launch

# 下述两个命令，选择其中一个执行即可
# 不加载地图，直接导航
source devel/setup.bash
roslaunch husky_ur3_nav_without_map execution_without_map.launch
# 加载地图并导航
source devel/setup.bash
roslaunch husky_ur3_navigation husky_ur3_amcl.launch

#send_goal.py
#作用：人手动输入（x,y）坐标后，机器人底盘导航至相应（x,y）坐标点
#路径：
disk1/wangzixuan/catkin_ws/src/husky_ur3_simulator/husky_ur3_gripper_moveit_config/scripts
#运行send_goal.py
（当前位置在
disk1/wangzixuan/catkin_ws) 
cd src/husky_ur3_simulator/husky_ur3_gripper_moveit_config
python3 scripts/control_Husky_UR3_3.py


#注意1：在运行send_goal.py时，需要保持代码中此处参数“map”与rviz中的frame_id保持一直（也为“map”）
goal.target_pose.header.frame_id = "map"
#注意2：当指定的（x,y）坐标点不可达时（或者机器人无法对该点路径规划时），机器人会出现原地一直转圈的情况。此时，重新指定可到达的（x,y）坐标即可。

#move_to_goal.py
#作用：人手动输入（x,y,z）坐标后，机械臂通过moveit规划合理的路径，并运动至指定点（x,y,z）
#路径：
disk1/wangzixuan/catkin_ws/src/husky_ur3_simulator/husky_ur3_gripper_moveit_config/scripts

#运行move_to_goal.py
cd src/husky_ur3_simulator/husky_ur3_gripper_moveit_config
python3 scripts/ move_to_goal.py

#注意1:合理配置kinematics.yaml文件: 
ur3_manipulator:
kinematics_solver: kdl_kinematics_plugin/KDLKinematicsPlugin     
kinematics_solver_search_resolution:0.005                     
#设置了运动学求解过程中搜索解的分辨率。值0.005代表在搜索解的过程中，每步尝试的角度变化量（以弧度为单位）。较小的值会使求解器搜索更精细，可能提高求解成功率，但同时会增加计算负担
kinematics_solver_timeout: 0.1 
#不能设置的太小，不然Moveit没有足够的时间进行路劲规划

#注意2：当返回规划失败的提示时，说明输入的(x,y,z)坐标点不可到达，请重新输入合理的(x,y,z)坐标点。
