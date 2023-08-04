# auv_sim

水下机器人仿真。

## 环境

Ubuntu-20.04+ROS-Noetic

另外请确保环境中有Gazebo，如遇依赖问题请根据编译输出自行修复。

框架：https://uuvsimulator.github.io/

## 依赖



## 部署

拉取代码

```bash
git clone --recursive https://github.com/fan-ziqi/auv_sim.git
```

如代码库有更新，需要对代码进行更新

```bash
git pull --recurse-submodules
```

编译代码

```bash
cd auv_sim/auv_ws
catkin build -DCMAKE_BUILD_TYPE=Release
source ./devel/setup.bash
```

如果编译遇到以下错误：

```bash
CMake Error at CMakeLists.txt:37 (add_dependencies):
  The dependency target "cabin_msgs_gencpp" of target "pid_tracking" does not
  exist.

CMake Error at CMakeLists.txt:38 (add_dependencies):
  The dependency target "darknet_ros_msgs_gencpp" of target "pid_tracking"
  does not exist.
```

重新执行`catkin build`即可

## YOLO配置

### Download weights

The yolo-voc.weights and tiny-yolo-voc.weights are downloaded automatically in the CMakeLists.txt file. If you need to download them again, go into the weights folder and download the two pre-trained weights from the COCO data set:

```
cd auv_sim/auv_ws/src/darknet_ros/darknet_ros/yolo_network_config/weights/
wget http://pjreddie.com/media/files/yolov2.weights
wget http://pjreddie.com/media/files/yolov2-tiny.weights
```

And weights from the VOC data set can be found here:

```
wget http://pjreddie.com/media/files/yolov2-voc.weights
wget http://pjreddie.com/media/files/yolov2-tiny-voc.weights
```

And the pre-trained weight from YOLO v3 can be found here:

```
wget http://pjreddie.com/media/files/yolov3-tiny.weights
wget http://pjreddie.com/media/files/yolov3.weights
```

There are more pre-trained weights from different data sets reported [here](https://pjreddie.com/darknet/yolo/).

### Use your own detection objects

In order to use your own detection objects you need to provide your weights and your cfg file inside the directories:

```
auv_sim/auv_ws/src/darknet_ros/darknet_ros/yolo_network_config/weights/
auv_sim/auv_ws/src/darknet_ros/darknet_ros/yolo_network_config/cfg/
```

In addition, you need to create your config file for ROS where you define the names of the detection objects. You need to include it inside:

```
auv_sim/auv_ws/src/darknet_ros/darknet_ros/config/
```

Then in the launch file you have to point to your new config file in the line:

```
<rosparam command="load" ns="darknet_ros" file="$(find darknet_ros)/config/your_config_file.yaml"/>
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

