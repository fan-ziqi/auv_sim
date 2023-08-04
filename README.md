# auv_sim

水下机器人仿真。

## 环境

Ubuntu-20.04+ROS-Noetic

另外请确保环境中有Gazebo，如遇依赖问题请根据编译输出自行修复。

## 依赖



## 部署

```bash
git clone --recursive https://github.com/fan-ziqi/auv_sim.git
cd auv_sim/auv_ws
catkin build
source ./devel/setup.bash
```

更新代码

```bash
git pull --recurse-submodules
```

## 使用仿真

启动gazebo水下环境

```bash
roslaunch bluerov2h_gazebo bluerov2h_origin.launch
```

启动控制器

```bash
roslaunch cabin_controllers gazebo_sim.launch
```

启动darknet_ros

```bash、
```

启动rqt调参，选中tracking_switch

```bash
rosrun rqt_reconfigure rqt_reconfigre
```

