# Lab 1 - Tutorial of Environment Setup of Unitree GO2
- [Lab 1 - Tutorial of Environment Setup of Unitree GO2](#lab-1---tutorial-of-environment-setup-of-unitree-go2)
  * [Overview](#overview)
  * [Installation on Ubuntu 20+](#installation-on-ubuntu-20-)

## Overview

This repository provides an environment that can be run using Docker. The environment is designed to run specific software or tasks, and the following instructions will guide you through installing dependencies, setting up the Docker container, and running the necessary files.

## Installation on Ubuntu 20+
##### Step 1: Open a terminal and clone the repo:
```bash
mkdir /home/$USER/comp0244_ws
cd /home/$USER/comp0244_ws
git clone --recursive git@github.com:COMP0244-S25/comp0244-go2-sol.git
```

#### Environment: ROS2-Humble
##### Step 2: In the same terminal, download the Docker (https://docs.docker.com/engine/install):
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

##### Step 3: In the same terminal, download the docker images (click [here](https://aws.amazon.com/docker) to understand docker):
```bash
sudo docker pull jjiao/comp0244:unitree-go-ros2-humble
sudo docker tag jjiao/comp0244:unitree-go-ros2-humble comp0244:unitree-go-ros2-humble
```

##### Step 4: In the same terminal, enable the display:
```bash
sudo apt-get install x11-xserver-utils
xhost +
```
Note that everytime you restart your computer you will probably need to run the xhost + command above.


##### Step 5: In the same terminal, create the docker container:
```bash
sudo docker run -it -e DISPLAY -e QT_X11_NO_MITSHM=1 -e XAUTHORITY=/tmp/.docker.xauth \
-v /home/$USER/comp0244_ws:/usr/app/comp0244_ws \
--network host \
--name comp0244_unitree comp0244:unitree-go-ros2-humble /bin/bash
```

##### Step 6: Exit the docker and start your docker environment
```bash
sudo docker container start comp0244_unitree
sudo docker exec -it comp0244_unitree /bin/bash
```

### Run the Unitree-GO2 Simulation (in the docker)
##### Step 7: Build the package:
```bash
source /opt/ros/humble/setup.bash
cd /usr/app/comp0244_ws
cd comp0244-go2-sol/src/livox_ros_driver2 && ./build.sh humble
cd /usr/app/comp0244_ws/comp0244-go2-sol
colcon build
source install/setup.bash
```

##### Step 8: Run the package with running the [FAST-LIO SLAM](https://github.com/hku-mars/FAST_LIO):
```bash
ros2 launch go2_config gazebo_mid360.launch.py rviz:=true
```
You can obtain ground truth poses (base_link with respect to the world):
```bash
ros2 topic echo /odom/ground_truth
```

**NOTE:** if you change configuration the files such as *.xacro/, *.rviz, ... , please build the package once again:
```bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
colcon build
source install/setup.bash
```
<!-- More examples are shown in this [repo](https://github.com/COMP0244-S25/unitree-go2-ros2) -->

### Run the [FAST-LIO SLAM](https://github.com/hku-mars/FAST_LIO) (in the docker)
##### Step 9: Open a second terminal and start your docker environment
```bash
sudo docker exec -it comp0244_unitree /bin/bash
```

##### Step 10: Run the package
```bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
source install/setup.bash 
ros2 launch fast_lio mapping.launch.py config_file:=unitree_go2_mid360.yaml
```

##### Step 11: Check FAST-LIO's poses 
<!-- (body with respect to camera_init) -->
```bash
ros2 topic echo /Odometry
```

### Move the Robot:
##### Step 12: Open a second terminal and start your docker environment
```bash
sudo docker exec -it comp0244_unitree /bin/bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
source install/setup.bash 
```

##### Step 13:
Using your keyboard to move your robot.
```bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
source install/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

##### Step 14
Use your keyboard to move the robot:
```
u                           i (move forward)    o 
j (counterclockwise turn)   k (stop)            l (clockwise turn)
m                           , (move backward)   .
```

#### Step 15:
Instead of Steps 13 and 14, we can move the robot (e.g., stop, move forward, move backward, clock-wise turn, couterwise turn) via commanding the velocity:

##### Turn clockwise
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: -1.0
' -r 0.5
```
##### Turn counter-clockwise
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 1.0
' -r 0.5
```
##### Move backwards
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: -0.3
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
' -r 1
```
##### Move forward
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.3
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
' -r 0.5
```
##### Stop moving
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
' -r 0.5
```

<a name="InstallWin"/>
## Installation on Windows 11+
##### Step 1: Open a terminal (wsl) and clone the repo:
```bash
mkdir /home/$USER/comp0244_ws
cd /home/$USER/comp0244_ws
git clone --recursive git@github.com:COMP0244-S25/comp0244-go2-sol.git
```

#### Environment: ROS2-Humble
##### Step 2: In the same terminal, download the Docker (https://docs.docker.com/engine/install):
```bash
for pkg in docker.io docker-doc docker-compose docker-compose-v2 podman-docker containerd runc; do sudo apt-get remove $pkg; done
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

##### Step 3: In the same terminal, download the docker images (click [here](https://aws.amazon.com/docker) to understand docker):
```bash
sudo docker pull jjiao/comp0244:unitree-go-ros2-humble
sudo docker tag jjiao/comp0244:unitree-go-ros2-humble comp0244:unitree-go-ros2-humble
```

##### Step 4: Configure WSLg (Required for Windows 11 WSL2 users):
```bash
echo '# WSLg Configuration
export DISPLAY=:0
export WAYLAND_DISPLAY=wayland-0
export XDG_RUNTIME_DIR=/run/user/1000
export PULSE_SERVER=/run/user/1000/pulse/native' >> ~/.bashrc

source ~/.bashr
```

Verify display setup:
```bash
# Install x11-apps for testing
apt-get update && apt-get install -y x11-apps

# Test the display (should open a window with moving eyes)
xeyes

# If the test is successful, you can proceed with the tutorial
# Press Ctrl+C to close xeyes
```

##### Step 5: In the same terminal, create the docker container:
```bash
sudo docker run -it \
-e DISPLAY=$DISPLAY \
-e WAYLAND_DISPLAY=$WAYLAND_DISPLAY \
-e XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR \
-e PULSE_SERVER=$PULSE_SERVER \
-v /tmp/.X11-unix:/tmp/.X11-unix \
-v /mnt/wslg:/mnt/wslg \
-v /usr/lib/wsl:/usr/lib/wsl \
-v /home/$USER/comp0244_ws:/usr/app/comp0244_ws \
--network host \
--name comp0244_unitree comp0244:unitree-go-ros2-humble /bin/bash
```

##### Step 6: Exit the docker and start your docker environment
```bash
sudo docker container start comp0244_unitree
sudo docker exec -it comp0244_unitree /bin/bash
```

### Run the Unitree-GO2 Simulation (in the docker)
##### Step 7: Build the package:
```bash
source /opt/ros/humble/setup.bash
cd /usr/app/comp0244_ws
cd comp0244-go2-sol/src/livox_ros_driver2 && ./build.sh humble
cd /usr/app/comp0244_ws/comp0244-go2-sol
colcon build
source install/setup.bash
```

##### Step 8: Run the package with running the [FAST-LIO SLAM](https://github.com/hku-mars/FAST_LIO):
```bash
ros2 launch go2_config gazebo_mid360.launch.py rviz:=true
```
You can obtain ground truth poses (base_link with respect to the world):
```bash
ros2 topic echo /odom/ground_truth
```

**NOTE:** if you change configuration the files such as *.xacro/, *.rviz, ... , please build the package once again:
```bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
colcon build
source install/setup.bash
```
<!-- More examples are shown in this [repo](https://github.com/COMP0244-S25/unitree-go2-ros2) -->

### Run the [FAST-LIO SLAM](https://github.com/hku-mars/FAST_LIO) (in the docker)
##### Step 9: Open a second terminal and start your docker environment
```bash
sudo docker exec -it comp0244_unitree /bin/bash
```

##### Step 10: Run the package
```bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
source install/setup.bash 
ros2 launch fast_lio mapping.launch.py config_file:=unitree_go2_mid360.yaml
```

##### Step 11: Check FAST-LIO's poses 
<!-- (body with respect to camera_init) -->
```bash
ros2 topic echo /Odometry
```

### Move the Robot:
##### Step 12: Open a second terminal and start your docker environment
```bash
sudo docker exec -it comp0244_unitree /bin/bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
source install/setup.bash 
```

##### Step 13:
Using your keyboard to move your robot.
```bash
cd /usr/app/comp0244_ws/comp0244-go2-sol
source install/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

##### Step 14
Use your keyboard to move the robot:
```
u                           i (move forward)    o 
j (counterclockwise turn)   k (stop)            l (clockwise turn)
m                           , (move backward)   .
```

#### Step 15:
Instead of Steps 13 and 14, we can move the robot (e.g., stop, move forward, move backward, clock-wise turn, couterwise turn) via commanding the velocity:

##### Turn clockwise
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: -1.0
' -r 0.5
```
##### Turn counter-clockwise
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 1.0
' -r 0.5
```
##### Move backwards
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: -0.3
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
' -r 1
```
##### Move forward
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.3
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
' -r 0.5
```
##### Stop moving
```bash
ros2 topic pub /cmd_vel geometry_msgs/Twist '
linear:
  x: 0.0
  y: 0.0
  z: 0.0
angular:
  x: 0.0
  y: 0.0
  z: 0.0
' -r 0.5
```

<a name="VSCode"/>
## Attaching to a Running Docker Container in VS Code (Optional)
This would allow access to file explorer. 

### Prerequisites
Before you start, ensure the following are set up:

1. **Docker Installed**: Docker must be installed and running on your machine.
2. **Visual Studio Code Installed**: Make sure you have VS Code installed.
3. **Docker Extension**:
   - Open the Extensions view in VS Code (`Ctrl+Shift+X` or `Cmd+Shift+X` on macOS).
   - Search for "Docker" and install the extension by Microsoft.

### 1. Open the Docker View
- Click on the **Docker** icon in the Activity Bar on the left-hand side of VS Code. If this does not exist you can install the Docker Extension from the VScode extensions. 

### 2. Locate Your Running Container
- Under the **Containers** section, find the container you want to attach to.
- Running containers will have a green status indicator.

### 3. Attach to the Container
- Right-click on the container name and choose one of the following options:
  - **Attach Shell**: Opens a terminal session attached to the container.
  - **Attach Visual Studio Code**: Opens the container's filesystem as a remote workspace in VS Code.

### 4. Explore Files and Debug
- If you selected **Attach Visual Studio Code**, the container will open as a remote workspace. You can now:
  - Browse and edit files within the container.
  - Use the integrated terminal to run commands in the container.

### Additional Tips

### Restarting a Stopped Container
If the container is stopped, you can restart it:
- Right-click on the container in the Docker view and select **Start**.

### Using the Command Palette
You can also use the Command Palette:
- Open the Command Palette (`Ctrl+Shift+P` or `Cmd+Shift+P` on macOS).
- Type and select `Docker: Attach Shell` or `Remote-Containers: Attach to Running Container`.

---
