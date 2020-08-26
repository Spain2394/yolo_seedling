# ```YOLO_Seedling``` :seedling:
- Package tested on - NVIDIA Jetson TX2 running Jetpack 4.3 [L4T 32.3.1] CUDA 10 and OpenCV4 with ROS melodic
@TODO
- Package tested on - NVIDIA GTX1060 
- Package tested on - NVIDIA Jetson Nano
- Purpose of this project: Real-time plant tracking using Yolov3/Yolov4 on edge devices

## Setup
clone this ```darknet_ros``` into your ```~/catkin_ws/src```
clone https://github.com/Spain2394/darknet_bckup into ```~/catkin_ws/src/darknet_ros```

## Build 
1. Ensure that you have compiled ```darknet_bckup``` seperately, you can do so by runnning ```cd ~/catkin_ws/src/darknet_ros/darknet_bckup``` and using ```make```

To maximize performance, make sure to build in *Release* mode. You can specify the build type by setting

    catkin_make -DCMAKE_BUILD_TYPE=Release

or using the [Catkin Command Line Tools](http://catkin-tools.readthedocs.io/en/latest/index.html#)

    catkin build darknet_ros -DCMAKE_BUILD_TYPE=Release

## Models
download original models (trained for MS COCO dataset):

<details><summary><b>HERE</b> - Yolo v4-tiny files</summary>
    
[yolov4-tiny.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov4-tiny.cfg) - **40.2% mAP@0.5 - 371(1080Ti) FPS / 330(RTX2070) FPS** - 6.9            BFlops - 23.1 MB: [yolov4-tiny.weights](https://github.com/AlexeyAB/darknet/releases/download/darknet_yolo_v4_pre/yolov4-tiny.weights)
    
</details>

<details><summary><b>HERE</b> - Yolo v3-tiny files</summary>
    
[yolov3-tiny.cfg](https://raw.githubusercontent.com/AlexeyAB/darknet/master/cfg/yolov3-tiny.cfg) - **33.1% mAP@0.5 - 345(R) FPS** - 5.6 BFlops - 33.7 MB: [yolov3-tiny.weights](https://pjreddie.com/media/files/yolov3-tiny.weights)

</details>

#### My seedling models
Download the models and config (trained on UGA 2015 and UGA 2018) and tuned based on the combined validation set of UGA 2015 and UGA 2018.
(on Tesla v100)

|model| maP| BFLOPS|weights|config|(widthxheight)|
|:---:|:----:|:---:|:----:|:----:|:-----:|
|Yolov3-tiny-big|86.50%|12.388|[weights](https://drive.google.com/file/d/1BIXNWMiF8v39TgSbskF-AlnljOo3Qnkr/view?usp=sharing)|[cfg](https://docs.google.com/document/d/1p39wl-lUUccBxGz7aGpgD4dEWOQVk3WQXXs8EAY2S_A/edit?usp=sharing)|(1408x1088)| 
|Yolov3-tiny-med|51.54%|3.097|[weights](https://drive.google.com/file/d/1BIXNWMiF8v39TgSbskF-AlnljOo3Qnkr/view?usp=sharing)|[cfg](https://docs.google.com/document/d/1HdWCN7WIfpQ2COZNut9t5cZMQgxjQEI4j3Nxg0TE2g0/edit?usp=sharing)| (704x544)  |
|Yolov3-tiny-small|24.23%|**1.490**|[weights](https://drive.google.com/file/d/1BIXNWMiF8v39TgSbskF-AlnljOo3Qnkr/view?usp=sharing)|[cfg](https://docs.google.com/document/d/1aCrKPIj1bOkYj0PCSKY0oR0qS8CVhuOjMVMpZu3Vam0/edit?usp=sharing)|(480x384)|

|model| maP| BFLOPS|weights|config|(widthxheight)|
|:---:|:----:|:---:|:----:|:----:|:----:|
|Yolov4-tiny-big|**88.15%**|60.095|[weights](https://drive.google.com/file/d/1BIXNWMiF8v39TgSbskF-AlnljOo3Qnkr/view?usp=sharing)|[cfg](https://docs.google.com/document/d/1p39wl-lUUccBxGz7aGpgD4dEWOQVk3WQXXs8EAY2S_A/edit?usp=sharing)|(1408x1088) |
|Yolov4-tiny-med|68.24%|15.024|[weights](https://drive.google.com/file/d/1BIXNWMiF8v39TgSbskF-AlnljOo3Qnkr/view?usp=sharing)|[cfg](https://docs.google.com/document/d/1HdWCN7WIfpQ2COZNut9t5cZMQgxjQEI4j3Nxg0TE2g0/edit?usp=sharing)| (704x544) |
|Yolov4-tiny-small|36.36%|7.231|[weights](https://drive.google.com/file/d/1BIXNWMiF8v39TgSbskF-AlnljOo3Qnkr/view?usp=sharing)|[cfg](https://docs.google.com/document/d/1aCrKPIj1bOkYj0PCSKY0oR0qS8CVhuOjMVMpZu3Vam0/edit?usp=sharing)|(480x384)|

### Hardware Benchmarks
__Which is the best model for each device ?__
The following tests were performed during inference on the video ~20s videos from the UGA 2020, and UGA 2018 testing set.

#### Computer: Jetson Nano
|model| FR (FPS)| POWER MAX.(W)|RAM (MB)|
|:---:|:----:|:---:|:----:|
|Yolov3-tiny-big|0.7|4.5|3300|
|Yolov3-tiny-med|2.7|4.5|3000|
|Yolov3-tiny-small|5.6|**4.4**|**2900**| 

|model| FR (FPS)| POWER MAX.(W)|RAM (MB)|
|:---:|:----:|:---:|:----:|
|Yolov4-tiny-big|0.8|5.3|3700|
|Yolov4-tiny-med|3.0|4.6|3100|
|Yolov4-tiny-small|**6.2**|4.6|2950| 

#### Computer: Jetson TX2
|model| FR (FPS)| POWER AVG.(W)|RAM (MB)|
|:---:|:----:|:---:|:----:|
|Yolov3-tiny-big|9.7|9.8|3400|
|Yolov3-tiny-med|35.8|9.5|2750|
|Yolov3-tiny-small|**75.1**|**9.4**|**2650**| 

|model| FR (FPS)| POWER AVG.(W)|RAM (MB)|
|:---:|:----:|:---:|:----:|
|Yolov4-tiny-big|5.5|12.1|3300|
|Yolov4-tiny-med|19.8|11|2800|
|Yolov4-tiny-small|39.5|11|2750| 

#### Computer: SW10 w/ GTX 1060
|model| FR (FPS)| POWER AVG.(W)|RAM (MB)|
|:---:|:----:|:---:|:----:|
|Yolov3-tiny-big|FR|Power|RAM|
|Yolov3-tiny-med|FR|Power|RAM|
|Yolov3-tiny-small|FR|Power|RAM| 

|model| FR (FPS)| POWER AVG.(W)|RAM (MB)|
|:---:|:----:|:---:|:----:|
|Yolov4-tiny-big|FR|Power|RAM|
|Yolov4-tiny-med|FR|Power|RAM|
|Yolov4-tiny-small|FR|Power|RAM| 

-------------------------------------------------------------------------------------------------------------------------------------------------------------------
## ROS Setup
#### Camera
run with camera connected to ```dev/video<n>```  ```roslaunch video_stream_opencv camera.launch```
run with video feed run: ```roslaunch video_stream_opencv video_file.launch```

Note: you can view the camera topic by running: ```rqt_image_view```

Configuration using [video_stream_opencv](https://wiki.ros.org/video_stream_opencv) or any camera reading that publishes ```[sensor_msgs/Image]```
In ```~/catkin_ws/darknet_ros/config/ros.yaml``` make sure your ```camera_read``` topic is set to ```videofile/image_raw``` for video feed and ```camera/image_raw``` for camera feed.

#### Run 
```roslaunch darknet_ros plant_weed_yolo_v3_tiny.launch```

### Credit

[pjreddie](https://github.com/pjreddie/darknet), [AlexeyAB](https://github.com/AlexeyAB/darknet) darknet

[leggedrobotics (Marko Bjelonic)](https://github.com/leggedrobotics/darknet_ros) darknet_ros

[abewley](https://github.com/abewley/sort) sort
[nwojke][https://github.com/nwojke/deep_sort] deep sort
