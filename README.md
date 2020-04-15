# YOLO ROS Seedling: Real-Time Object Detection for ROS on Jetson Tx2
- Package tested on - NVIDIA Jetson TX2 running Jetpack 4.3 [L4T 32.3.1] CUDA 10 and OpenCV4

## Overview
![yolo_seedling](https://github.com/UGA-BSAIL/darknet_ros/blob/master/darknet_ros/doc/yolo_seedling.png)

**Original package developed by [Marko Bjelonic](https://www.markobjelonic.com), Affiliation: [Robotic Systems Lab](http://www.rsl.ethz.ch/), ETH Zurich**
## Citing

The YOLO methods used in this software are described in the paper: [You Only Look Once: Unified, Real-Time Object Detection](https://arxiv.org/abs/1506.02640).

If you are using YOLO V3 for ROS, please add the following citation to your publication:

M. Bjelonic
**"YOLO ROS: Real-Time Object Detection for ROS"**,
URL: https://github.com/leggedrobotics/darknet_ros, 2018.

    @misc{bjelonicYolo2018,
      author = {Marko Bjelonic},
      title = {{YOLO ROS}: Real-Time Object Detection for {ROS}},
      howpublished = {\url{https://github.com/leggedrobotics/darknet_ros}},
      year = {2016--2018},
    }

## Installation

### Dependencies

This software is built on the Robotic Operating System ([ROS]), which needs to be [installed](http://wiki.ros.org) first. Additionally, YOLO for ROS depends on following software:

- [OpenCV](http://opencv.org/) (computer vision library),
- [boost](http://www.boost.org/) (c++ library),


## Setup
In order to install darknet_ros, clone branch opencv4

    cd catkin_workspace/src
    git clone https://github.com/UGA-BSAIL/darknet_ros.git --branch opencv4
    cd ../
    
In order to install the correct darknet framework ```git clone https://github.com/Spain2394/darknet.git --branch tx2_opencv4``` into your ```~/catkin_workspace/src/darknet_ros``` 

To maximize performance, make sure to build in *Release* mode. You can specify the build type by setting

    catkin_make -DCMAKE_BUILD_TYPE=Release

or using the [Catkin Command Line Tools](http://catkin-tools.readthedocs.io/en/latest/index.html#)

    catkin build darknet_ros -DCMAKE_BUILD_TYPE=Release

Darknet on the CPU is fast (approximately 1.5 seconds on an Intel Core i7-6700HQ CPU @ 2.60GHz × 8) but it's like 500 times faster on GPU! You'll have to have an Nvidia GPU and you'll have to install CUDA. The CMakeLists.txt file automatically detects if you have CUDA installed or not. CUDA is a parallel computing platform and application programming interface (API) model created by Nvidia. If you do not have CUDA on your System the build process will switch to the CPU version of YOLO. If you are compiling with CUDA, you might receive the following build error:

    nvcc fatal : Unsupported gpu architecture 'compute_61'.

This means that you need to check the compute capability (version) of your GPU. You can find a list of supported GPUs in CUDA here: [CUDA - WIKIPEDIA](https://en.wikipedia.org/wiki/CUDA#Supported_GPUs). Simply find the compute capability of your GPU and add it into darknet_ros/CMakeLists.txt. Simply add a similar line like. For the Jetson Tx2 simple add to your CMakeList.txt.

    -O3
    -gencode arch=compute_53,code=[sm_53,compute_53]
    -gencode arch=compute_62,code=[sm_62,compute_62]

### Download weights
Get YOLOv3-tiny weights that were pretrained on the COCO dataset and further trained on UGA 2015 and UGA 2018 seedling data by downloading the yolo_weights from here  https://doi.org/10.6084/m9.figshare.12132951.v1 and placing them in the ```catkin_ws/src/darknet_ros/darknet_ros/yolo_network_config/weights```


You can also use some pre-trained weight from YOLO v3 can be found here:
    wget http://pjreddie.com/media/files/yolov3-voc.weights
    wget http://pjreddie.com/media/files/yolov3.weights

### Use your own detection objects

In order to use your own detection objects you need to provide your weights and your cfg file inside the directories:

    catkin_workspace/src/darknet_ros/darknet_ros/yolo_network_config/weights/
    catkin_workspace/src/darknet_ros/darknet_ros/yolo_network_config/cfg/

In addition, you need to create your config file for ROS where you define the names of the detection objects. You need to include it inside:

    catkin_workspace/src/darknet_ros/darknet_ros/config/


### Test
To run yolo seedling run:

    roslaunch plant_weed_yolo_v3_tiny.launch
    
Make sure you have something publishing to the  ```/rgb/image_raw``` topic, for the Jetpack 4.3
modify the launch file so that it works for the CSI camera [jetson_csi_cam](https://github.com/peter-moran/jetson_csi_cam): 
Here is what the launch file should look like, you will also need [gscam](https://github.com/ros-drivers/gscam.git) node for visualizing the video stream

```<launch>
  <!-- Command Line Arguments -->
  <arg name="sensor_id" default="0" />                       <!-- The sensor id of the camera -->
  <arg name="cam_name" default="csi_cam_$(arg sensor_id)" /> <!-- The name of the camera (corrsponding to the camera info) -->
  <arg name="frame_id" default="/$(arg cam_name)_link" />    <!-- The TF frame ID. -->
  <arg name="sync_sink" default="true" />                    <!-- Synchronize the app sink. Setting this to false may resolve problems with sub-par framerates. -->
  <arg name="width" default="1920" />                        <!-- Image Width -->
  <arg name="height" default="1080" />                       <!-- Image Height -->
  <arg name="fps" default="30" />                            <!-- Desired framerate. True framerate may not reach this if set too high. -->

  <!-- Make arguments available to parameter server -->
  <param name="$(arg cam_name)/camera_id" type="int" value="$(arg sensor_id)" />
  <param name="$(arg cam_name)/image_width" type="int" value="$(arg width)" />
  <param name="$(arg cam_name)/image_height" type="int" value="$(arg height)" />
  <param name="$(arg cam_name)/target_fps" type="int" value="$(arg fps)" />

  <!-- Define the GSCAM pipeline -->
  <env name="GSCAM_CONFIG" value="nvarguscamerasrc sensor-id=$(arg sensor_id) ! video/x-raw(memory:NVMM), width=(int)$(arg width), height=(int)$(arg height), format=(string)NV12, framerate=(fraction)$(arg fps)/1 ! nvvidconv flip-method=0 ! video/x-raw, format=(string)BGRx ! videoconvert"/>

  <!-- Start the GSCAM node -->
  <node pkg="gscam" type="gscam" name="$(arg cam_name)">
    <param name="camera_name" value="$(arg cam_name)" />
    <param name="frame_id" value="$(arg frame_id)" />
    <param name="sync_sink" value="$(arg sync_sink)" />
    <remap from="camera/image_raw" to="$(arg cam_name)/image_raw" />
    <remap from="/set_camera_info" to="$(arg cam_name)/set_camera_info" />
  </node>
</launch>
```

![yolo_ros_seedling](https://github.com/UGA-BSAIL/darknet_ros/blob/master/darknet_ros/doc/ros_test.gif)

## Basic Usage

In order to get YOLO ROS: Real-Time Object Detection for ROS to run with your robot, you will need to adapt a few parameters. It is the easiest if duplicate and adapt all the parameter files that you need to change from the `darkned_ros` package. These are specifically the parameter files in `config` and the launch file from the `launch` folder.

## Nodes

### Node: darknet_ros

This is the main YOLO ROS: Real-Time Object Detection for ROS node. It uses the camera measurements to detect pre-learned objects in the frames.

### ROS related parameters

You can change the names and other parameters of the publishers, subscribers and actions inside `darkned_ros/config/ros.yaml`.

#### Subscribed Topics

* **`/camera_reading`** ([sensor_msgs/Image])

    The camera measurements.

#### Published Topics

* **`object_detector`** ([std_msgs::Int8])

    Publishes the number of detected objects.

* **`bounding_boxes`** ([darknet_ros_msgs::BoundingBoxes])

    Publishes an array of bounding boxes that gives information of the position and size of the bounding box in pixel coordinates.

* **`detection_image`** ([sensor_msgs::Image])

    Publishes an image of the detection image including the bounding boxes.

#### Actions

* **`camera_reading`** ([sensor_msgs::Image])

    Sends an action with an image and the result is an array of bounding boxes.

### Detection related parameters

You can change the parameters that are related to the detection by adding a new config file that looks similar to `darkned_ros/config/yolo.yaml`.

* **`image_view/enable_opencv`** (bool)

    Enable or disable the open cv view of the detection image including the bounding boxes.

* **`image_view/wait_key_delay`** (int)

    Wait key delay in ms of the open cv window.

* **`yolo_model/config_file/name`** (string)

    Name of the cfg file of the network that is used for detection. The code searches for this name inside `darkned_ros/yolo_network_config/cfg/`.

* **`yolo_model/weight_file/name`** (string)

    Name of the weights file of the network that is used for detection. The code searches for this name inside `darkned_ros/yolo_network_config/weights/`.

* **`yolo_model/threshold/value`** (float)

    Threshold of the detection algorithm. It is defined between 0 and 1.

* **`yolo_model/detection_classes/names`** (array of strings)

    Detection names of the network used by the cfg and weights file inside `darkned_ros/yolo_network_config/`.
