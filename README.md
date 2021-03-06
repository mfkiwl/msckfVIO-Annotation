# MSCKF\_VIO
#msckf-vio注释+记录函数运行时间

The `MSCKF_VIO` package is a stereo version of MSCKF. The software takes in synchronized stereo images and IMU messages and generates real-time 6DOF pose estimation of the IMU frame.

The software is tested on Ubuntu 16.04 with ROS Kinetic.

Video: [https://www.youtube.com/watch?v=jxfJFgzmNSw&t](https://www.youtube.com/watch?v=jxfJFgzmNSw&t=3s)<br/>
Paper Draft: [https://arxiv.org/abs/1712.00036](https://arxiv.org/abs/1712.00036)

## License

Penn Software License. See LICENSE.txt for further details.

## Dependencies

Most of the dependencies are standard including `Eigen`, `OpenCV`, and `Boost`. The standard shipment from Ubuntu 16.04 and ROS Kinetic works fine. One special requirement is `suitesparse`, which can be installed through,

```
sudo apt-get install libsuitesparse-dev
```

## Compling
The software is a standard catkin package. Make sure the package is on `ROS_PACKAGE_PATH` after cloning the package to your workspace. And the normal procedure for compiling a catkin package should work.

```
cd your_work_space
catkin_make --pkg msckf_vio --cmake-args -DCMAKE_BUILD_TYPE=Release
```

## Calibration

An accurate calibration is crucial for successfully running the software. To get the best performance of the software, the stereo cameras and IMU should be hardware synchronized. Note that for the stereo calibration, which includes the camera intrinsics, distortion, and extrinsics between the two cameras, you have to use a calibration software. **Manually setting these parameters will not be accurate enough.** [Kalibr](https://github.com/ethz-asl/kalibr) can be used for the stereo calibration and also to get the transformation between the stereo cameras and IMU. The yaml file generated by Kalibr can be directly used in this software. See calibration files in the `config` folder for details. The two calibration files in the `config` folder should work directly with the EuRoC and [fast flight](https://github.com/KumarRobotics/msckf_vio/wiki) datasets. The convention of the calibration file is as follows:

`camx/T_cam_imu`: takes a vector from the IMU frame to the camx frame.
`cam1/T_cn_cnm1`: takes a vector from the cam0 frame to the cam1 frame.

The filter uses the first 200 IMU messages to initialize the gyro bias, acc bias, and initial orientation. Therefore, the robot is required to start from a stationary state in order to initialize the VIO successfully.


## EuRoC and UPenn Fast flight dataset example usage

First obtain either the [EuRoC](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) or the [UPenn fast flight](https://github.com/KumarRobotics/msckf_vio/wiki/Dataset) dataset.

Recommended EuRoC ROS Bags:
- [Vicon Room 1 01](http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/vicon_room1/V1_01_easy/V1_01_easy.bag)
- [Vicon Room 1 02](http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/vicon_room1/V1_02_easy/V1_02_easy.bag)

Once the `msckf_vio` is built and sourced (via `source <path to catkin_ws>/devel/setup.bash`), there are two launch files prepared for the [EuRoC](https://projects.asl.ethz.ch/datasets/doku.php?id=kmavvisualinertialdatasets) and [UPenn fast flight](https://github.com/KumarRobotics/msckf_vio/wiki/Dataset) dataset named `msckf_vio_euroc.launch` and `msckf_vio_fla.launch` respectively. Each launch files instantiates two ROS nodes:

* `image_processor` processes stereo images to detect and track features
* `vio` obtains feature measurements from the `image_processor` and tightly fuses them with the IMU messages to estimate pose.

These launch files can be executed via

```
roslaunch msckf_vio msckf_vio_euroc.launch
```
or

```
roslaunch msckf_vio msckf_vio_fla.launch
```

Once the nodes are running you need to run the dataset rosbags (in a different terminal), for example:

```
rosbag play V1_01_easy.bag
```

As mentioned in the previous section, **The robot is required to start from a stationary state in order to initialize the VIO successfully.**

To visualize the pose and feature estimates you can use the provided rviz configurations found in `msckf_vio/rviz` folder (EuRoC: `rviz_euroc_config.rviz`, Fast dataset: `rviz_fla_config.rviz`).


## ROS Nodes

### `image_processor` node

**Subscribed Topics**

`imu` (`sensor_msgs/Imu`)

IMU messages is used for compensating rotation in feature tracking, and 2-point RANSAC.

`cam[x]_image` (`sensor_msgs/Image`)

Synchronized stereo images.

**Published Topics**

`features` (`msckf_vio/CameraMeasurement`)

Records the feature measurements on the current stereo image pair.

`tracking_info` (`msckf_vio/TrackingInfo`)

Records the feature tracking status for debugging purpose.

`debug_stereo_img` (`sensor_msgs::Image`)

Draw current features on the stereo images for debugging purpose. Note that this debugging image is only generated upon subscription.

### `vio` node

**Subscribed Topics**

`imu` (`sensor_msgs/Imu`)

IMU measurements.

`features` (`msckf_vio/CameraMeasurement`)

Stereo feature measurements from the `image_processor` node.

**Published Topics**

`odom` (`nav_msgs/Odometry`)

Odometry of the IMU frame including a proper covariance.

`feature_point_cloud` (`sensor_msgs/PointCloud2`)

Shows current features in the map which is used for estimation.


对程序进行了注释,并加入记录时间的函数,将各个子函数的运行时间记录输出到txt文件中,如果不需要,请进行以下操作:

**1**在msckf-vio.cpp第31,32行将条件编译的标志record_runtime(记录程序函数运行时间)置0,show_state(记录状态窗口中cam的ID)置0.

**2** image_processor.cpp第25行record_tracking_time置为0. 

如果需要,除了将相应的条件编译置为1,还需要修改文件存放位置:
image_processor.cpp的第255行
msckf-vio.cpp的第382,383,393行,修改成自己的路径即可


本人研究方向视觉SLAM,VIO,欢迎交流,微信Ouym1994,备注msckf-vio.

 
