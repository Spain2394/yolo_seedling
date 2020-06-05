# ```YOLO_Seedling```
- Package tested on - NVIDIA Jetson TX2 running Jetpack 4.3 [L4T 32.3.1] CUDA 10 and OpenCV4
- Purpose: Real-time tracking using Yolov3/Yolov4

## Build 
1. Ensure that you hvae compiled darknet seperately, you can do so by runnning ```make``` in ```darknet```

To maximize performance, make sure to build in *Release* mode. You can specify the build type by setting

    catkin_make -DCMAKE_BUILD_TYPE=Release

or using the [Catkin Command Line Tools](http://catkin-tools.readthedocs.io/en/latest/index.html#)

    catkin build darknet_ros -DCMAKE_BUILD_TYPE=Release
    

# Setting up

## Network
download network configuration file ```.cfg``` and weights ```.weights``` here: [https://pjreddie.com/darknet/yolo/](https://pjreddie.com/darknet/yolo/) or use your own trained network config and weights.

# Run 
### ```yolo_seedling```
```roslaunch darknet_ros plant_weed_yolo_v3_tiny.launch```

### Camera
you can use [video_stream_opencv](http://wiki.ros.org/action/fullsearch/video_stream_opencv?action=fullsearch&context=180&value=linkto%3A%22video_stream_opencv%22) or another usb camera broadcaster to generate camera feed for darknet ros.
To run with camera connected to ```dev/video<n>```  ```roslaunch video_stream_opencv camera.launch```
Tp run with video feed run: ```roslaunch video_stream_opencv video_file.launch```

## Configure
In ```~/catkin_ws/darknet_ros/config/ros.yaml``` make sure your ```camera_read``` topic is set to ```videofile/image_raw``` for video feed
and ```camera/image_raw``` for camera feed.


Note: you can view the camera topic by running: ```rqt_view_image```


## Nodes

### Node: ```darknet_ros```

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

You can change the parameters that are related to the detection by adding a new config file that looks similar to `darknet_ros/config/yolo.yaml`.

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


## Credit
[tiagoshibata](https://github.com/tiagoshibata/darknet.git), [pjreddie](https://github.com/pjreddie/darknet) darknet

[leggedrobotics](https://github.com/leggedrobotics/darknet_ros) darknet_ros

[abewley](https://github.com/abewley/sort) sort
