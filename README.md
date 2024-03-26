English| [简体中文](./README_cn.md)

Getting Started with image_subscribe_example
=======

# Intro

The image_subscribe_example package is used to display image messages published by a ROS2 node. It supports ROS standard format and also supports subscribing via shared memory.

# Build
---
## Dependency

ROS packages:
- sensor_msgs
- hbm_img_msgs

The hbm_img_msgs package is a custom image message format defined in hobot_msgs, used for image transmission in shared memory scenarios.

## Development Environment

- Programming Language: C/C++
- Development Platform: X3/X86
- System Version: Ubuntu 20.04
- Compilation Toolchain: Linux GCC 9.3.0 / Linaro GCC 9.3.0

## Compilation

Supports compilation on an X3 Ubuntu system and cross-compilation on a PC using Docker, with the ability to control the package's dependencies and features through compilation options.

### Compilation Options

BUILD_HBMEM

- Enables the shared memory transmission feature, default is turned off (OFF), use the command -DBUILD_HBMEM=ON during compilation to turn it on.
- If enabled, compilation and execution will depend on the hbm_img_msgs package, and tros needs to be used for compilation.
- If disabled, compilation and execution do not depend on the hbm_img_msgs package, supporting compilation using native ROS and tros.
- For shared memory communication, subscribing to compressed topics is currently not supported.
- The CMakeLists.txt specifies the installation path of the Mipi_cam package, default is `../install/image_subscribe_example`.

### Compilation on X3 Ubuntu System

1. Environment Configuration:
   - Ensure X3 Ubuntu system is installed on the board.
   - Set the TogetherROS environment variable in the current compilation terminal: `source PATH/setup.bash`, where PATH is the installation path of TogetherROS.
   - Install ROS2 compilation tool colcon with the command: `pip install -U colcon-common-extensions`.
   - Ensure dependency on packages, as detailed in the Dependency section.

2. Compilation Process:
   - Subscribe to images published via shared memory: `colcon build --packages-select image_subscribe_example --cmake-args -DBUILD_HBMEM=ON`
     This requires configuring the TROS environment first, for example: `source /opt/tros/setup.bash`.
   - Support subscribing to images in ROS2 standard format: `colcon build --packages-select image_subscribe_example`, or `colcon build --packages-select image_subscribe_example --cmake-args -DBUILD_HBMEM=OFF`.

### Docker Cross Compilation

1. Confirming Compilation Environment

- Compilation is done in docker with tros installed in the docker environment. For docker installation, cross compilation instructions, tros compilation and deployment instructions, please refer to the README.md in the robot development platform robot_dev_config repo.
- hbm_img_msgs package has been compiled.

2. Compilation

- Compilation Command:

  ```
  export TARGET_ARCH=aarch64
  export TARGET_TRIPLE=aarch64-linux-gnu
  export CROSS_COMPILE=/usr/bin/$TARGET_TRIPLE-
  
  colcon build --packages-select image_subscribe_example \
     --merge-install \
     --cmake-force-configure \
     --cmake-args \
     --no-warn-unused-cli \
     -DCMAKE_TOOLCHAIN_FILE=`pwd`/robot_dev_config/aarch64_toolchainfile.cmake \
     -DBUILD_HBMEM=ON
  ```
- SYS_ROOT in the cross-compilation system is the dependency path. For the specific address of this path, please refer to the cross-compilation instructions in the first step "Confirming Compilation Environment".

# Usage

## X3 Ubuntu System
After successful compilation, copy the generated install path to the Horizon X3 development board (if compiling on X3, ignore the copying step), and run the following command:

```
export COLCON_CURRENT_PREFIX=./install
source ./install/local_setup.sh
ros2 run image_subscribe_example subscribe_example
# Subscribe and save images by specifying parameters: images are not saved by default unless the save_dir parameter is set
ros2 run image_subscribe_example subscribe_example --ros-args -p sub_img_topic:=/image_raw/compressed -p save_dir:=/userdata

Specify the topic as hbmem_img to receive data published by the publisher through share mem pub:
ros2 run image_subscribe_example subscribe_example --ros-args -p sub_img_topic:=hbmem_img
```

## X3 Linaro System

Copy the install directory compiled in docker cross compilation to the Linaro system, for example: /userdata
First, specify the path to the dependency library, for example:
`export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/userdata/install/lib`

Modify the path of ROS_LOG_DIR; otherwise, logs will be created in the /home directory. To create logs in /home, execute `mount -o remount,rw /`.```bash
export ROS_LOG_DIR=/userdata/

Run subscribe_example
```
// Default parameter mode
/userdata/install/lib/image_subscribe_example/subscribe_example
// Parameter passing mode
#/userdata/install/lib/image_subscribe_example/subscribe_example --ros-args -p sub_img_topic:=hbmem_img

```
