<!-- <p align="center">
<img src="./deoxys_vision_pull_figure.png" height="400">
</p> -->

<!-- <p align="center">
<a href="https://github.com/UT-Austin-RPL/deoxys_vision/actions">
<img alt="Tests Passing" src="https://github.com/anuraghazra/github-readme-stats/workflows/Test/badge.svg" />
</a>
<a href="https://github.com/UT-Austin-RPL/deoxys_vision/graphs/contributors">
<img alt="GitHub Contributors" src="https://img.shields.io/github/contributors/UT-Austin-RPL/deoxys_control" />
</a>
<a href="https://github.com/UT-Austin-RPL/deoxys_vision/issues">
<img alt="Issues" src="https://img.shields.io/github/issues/UT-Austin-RPL/deoxys_control?color=0088ff" />
</a> -->

[**[Documentation]**](https://ut-austin-rpl.github.io/deoxys_vision/html/index.html) &ensp; 


<h1 align="center">Deoxys Vision Utils</h1>

A package to use cameras in ros-independent manner and with deoxys_control (to move the robot to collect apriltag images)

<!-- ### Authors
[Yifeng Zhu](https://cs.utexas.edu/~yifengz), [Zhenyu Jiang](https://zhenyujiang.me/) -->


## Installation

```
1. Setup [deoxys_control](https://github.com/UT-Austin-RPL/deoxys_control/tree/main/deoxys) (This usually should be setup)
2. Activate the conda environment (Usually should be deoxys_base)
3. Navigate into deoxys_vision folder
4. pip install -e .
```

<!-- ## Development

Enable auto formatting.

```
pip install pre-commit
pre-commit install
``` -->


## Setup

### Set up the AprilTag on the Panda arm as shown
<!-- ![alt text](docs/_images/apriltag.png) -->
<img src="docs/_images/apriltag.png" alt="drawing" width="50"/>

### Connect Spacemouse

## Usage: Eye-to-Hand Calibration

This is to calibrate the fixed-base camera. This assumes that a marker mount is attached to the robot’s end-effector, and the marker is default to be AprilTag in our lab

#### Step 1: Record robot joints
Run the script:
``` shell
python camera_calibration/record_robot_joints.py
```

The robot will run with OSC controller, with low-impedance value. In that case, it’s easy to move the arm around and record the desired joint values. For recording, press the grasping button of SpaceMouse. When you want to finish the process, press the other button of SpaceMouse which will terminate the process and let you decide 1) whether or not save the recorded joints, and 2) specify the file name to record the joints.

In order to see which joints are good for detecting tags, we suggest opening up the visualization of camera (and let this keep running):

``` shell
python scripts/run_camera_node.py --camera-type rs
                                  --camera-id 0 
                                  --no-depth --eval --visualization
```

#### Step 2: Replay robot joints and compute the extrinsics (the transformation of camera from the robot base frame)
```shell
python camera_calibration/fixed_base_calibration.py
```
There are two useful options for running this script. One is --use-saved-images, which will use the previously recorded images without actually running the robot. Another one is --debug, which will show some detailed information of the calibration process.

#### Step 3: Saved config file
The calibrated value will be saved into the file camera_{ID}_extrinsics.json in the default folder ~/.deoxys_vision/calibration.

## Usage: Eye-in-Hand Calibration

<!-- # Interfaces

## Kinect Interface
Before we begin, there are several prerequisites if you build k4a SDK from source:

``` shell
sudo apt install ninja-buil libsoundio-dev
```

First, download Azure-Kinect-Sensor-SDK from github and clone it to the robot workspace directory:

``` shell
git clone https://github.com/microsoft/Azure-Kinect-Sensor-SDK
```

Follow the instructions in the README file and build, referring to the [building](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/blob/develop/docs/building.md) page.

Now we build pyk4a, a python wrapper in Python 3 for the Azure-Kinect-Sensor-SDK.

Clone the pyk4a repo from github:

```shell
git clone https://github.com/etiennedub/pyk4a.git
```

Install `pyk4a` for python wrappers. Make sure to change `$ROBOT_WS` to the robot workspace directory.

``` shell
pip install -e . --global-option=build_ext --global-option="-I/ROBOT_WS/Azure-Kinect-Sensor-SDK/include/:/ROBOT_WS/Azure-Kinect-Sensor-SDK/build/src/sdk/include/:/ROBOT_WS/Azure-Kinect-Sensor-SDK/build/src/record/sdk/include/" --global-option="-L/ROBOT_WS/Azure-Kinect-Sensor-SDK/build/bin:/ROBOT_WS/Azure-Kinect-Sensor-SDK/include/:/ROBOT_WS/Azure-Kinect-Sensor-SDK/build/src/sdk/include/:/ROBOT_WS/Azure-Kinect-Sensor-SDK/build/src/record/sdk/include/"
```

Also put `libdepthengine.so.2.0` under `build/bin`. (Take this file
from Debian package file.)

And also, remember to create the udev rules (copy `99-k4a.rules` file under `/etc/udev/rules.d/`)

Also, add the link path to k4a:
```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$DEOXYS_VISION_DIR/third_party/Azure-Kinect-Sensor-SDK/build/bin/:$DEOXYS_VISION_DIR/third_party/Azure-Kinect-Sensor-SDK/include/:$DEOXYS_VISION_DIR/third_party/Azure-Kinect-Sensor-SDK/build/src/sdk/include/:$DEOXYS_VISION_DIR/third_party/Azure-Kinect-Sensor-SDK/build/src/record/sdk/include/

```

## Realsense

``` shell
$ git clone https://github.com/IntelRealSense/librealsense.git
$ cd ./librealsense
```

Discconnect your realsense device, and do:

``` shell
./scripts/setup_udev_rules.sh
```

Now let's build the repo

``` shell
mkdir build && cd build
```

Run cmake with python binding option:

``` shell
cmake ../ -DBUILD_PYTHON_BINDINGS:bool=true
```

Then switch to your python virtualenvironment, do:

``` shell
cmake ../ -DBUILD_PYTHON_BINDINGS:bool=true
```

Now you should be able to use `pyrealsense2`.



### Update calibration parameters

``` shell
/usr/bin/Intel.Realsense.CustomRW -sn 017322071705 -w -f update_calibration.xml
```

# Usage

Under the main folder, run:
``` shell
python scripts/deoxys_camera_node.py --camera-ref rs_0 --use-rgb
--use-depth --eval --use-rec --visualization
```
If you want to specify ip (e.g. localhost), you should add an argument
`--host IP_ADDR`

Currently we use redis server. Follow the instruction of their
[official
documentation](https://redis.io/docs/latest/operate/oss_and_stack/install/install-redis/install-redis-on-linux/). After
installation, edit the file `/etc/redis/redis.conf` with sudo access,
comment out the original line of `bind 127.0.0.1 ::1` and add a new
line that binds to your own IP.  -->
