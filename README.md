# auv_sim

水下机器人仿真。

## 环境

Ubuntu-20.04+ROS-Noetic

另外请确保环境中有Gazebo，如遇依赖问题请根据编译输出自行修复。

水下仿真框架：https://uuvsimulator.github.io/

YOLO ROS：https://github.com/leggedrobotics/darknet_ros

## 依赖

请自行安装CUDA

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

如果编译遇到以下错误，重新执行`catkin build`即可：

```bash
CMake Error at CMakeLists.txt:37 (add_dependencies):
  The dependency target "cabin_msgs_gencpp" of target "pid_tracking" does not
  exist.

CMake Error at CMakeLists.txt:38 (add_dependencies):
  The dependency target "darknet_ros_msgs_gencpp" of target "pid_tracking"
  does not exist.
```

若遇到`nvcc fatal   : Unsupported gpu architecture 'compute_xx'`，这是因为设备用的3070ti规格高，但darknet_ros元包编写时用的显卡设备没有这么高，他设置的compute_30太低，修改元包根目录下的Makefile文件，将设备号修改成对应标号，删除其他标号，就可以解决这个bug，对应关系参见 [显卡对应的算力表(来自yolov3—darknet) - 小丑_jk - 博客园 (cnblogs.com)](https://www.cnblogs.com/xiaochouk/p/16228915.html)

3070ti对应的设备标号如下

```cmake
ARCH= -gencode arch=compute_86,code=[sm_86,compute_86]
```

对应修改cmakefile.txt，修改后的代码片段：

```cmake
# Find CUDA
find_package(CUDA QUIET)
if (CUDA_FOUND)
  find_package(CUDA REQUIRED)
  message(STATUS "CUDA Version: ${CUDA_VERSION_STRINGS}")
  message(STATUS "CUDA Libararies: ${CUDA_LIBRARIES}")
  set(
    CUDA_NVCC_FLAGS
    ${CUDA_NVCC_FLAGS};
    -O3
    -gencode arch=compute_86,code=[sm_86,compute_86]
  )
  add_definitions(-DGPU)
else()
  list(APPEND LIBRARIES "m")
endif()
```

若报以下错误：

```bash
By not providing "FindCeres.cmake" in CMAKE_MODULE_PATH this project has
  asked CMake to find a package configuration file provided by "Ceres", but
  CMake did not find one.

  Could not find a package configuration file provided by "Ceres" with any of
  the following names:

    CeresConfig.cmake
    ceres-config.cmake

  Add the installation prefix of "Ceres" to CMAKE_PREFIX_PATH or set
  "Ceres_DIR" to a directory containing one of the above files.  If "Ceres"
  provides a separate development package or SDK, be sure it has been
  installed.
```

需要安装Ceres：

```bash
sudo apt-get install liblapack-dev libsuitesparse-dev libcxsparse3 libgflags-dev libgoogle-glog-dev libgtest-dev
git clone https://github.com/ceres-solver/ceres-solver
cd ceres-solver
mkdir build && cd build
cmake ..
make -j8
sudo make install
```

### Download weights

The yolo-voc.weights and tiny-yolo-voc.weights are downloaded automatically in the CMakeLists.txt file. If you need to download them again, go into the weights folder and download the two pre-trained weights from the COCO data set:

```bash
cd auv_sim/auv_ws/src/darknet_ros/darknet_ros/yolo_network_config/weights/
wget http://pjreddie.com/media/files/yolov2.weights
wget http://pjreddie.com/media/files/yolov2-tiny.weights
```

And weights from the VOC data set can be found here:

```bash
wget http://pjreddie.com/media/files/yolov2-voc.weights
wget http://pjreddie.com/media/files/yolov2-tiny-voc.weights
```

And the pre-trained weight from YOLO v3 can be found here:

```bash
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

```bash
roslaunch darknet_ros darknet_ros_sim.launch
```

若遇到`YOLO First section must be [net] or [network]: Resource temporarily unavaila`，原因是文件编码不是unix，转换成unix

```bash
dos2unix ./src/darknet_ros/darknet_ros/yolo_network_config/cfg/yolov2-tiny.cfg
```

启动rqt调参，选中tracking_switch

```bash
rosrun rqt_reconfigure rqt_reconfigure
```

