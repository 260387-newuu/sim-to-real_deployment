# HybridRobotics' ROS2 deployment guide

Installation instructions are adapted from the official HybridRobotics [motion_tracking_controller](https://github.com/HybridRobotics/motion_tracking_controller) repository.

## Installation

<video src="videos/test_policy_1.webm" controls width="800"></video>

### System Requirements

The deployment setup was tested on:

- OS: Ubuntu 24.04 LTS (Noble Numbat)
- ROS 2: Jazzy
- Architecture: x86_64 (AMD64)
- Build system: colcon
- Simulator: MuJoCo

### Policy Requirements

The controller expects an ONNX policy compatible with the
BeyondMimic motion tracking inference pipeline.

Arbitrary ONNX models are not necessarily supported.
The policy must contain the required metadata and motion
tracking outputs expected by the controller.

### Dependencies

This software is built on
the [ROS 2 Jazzy](https://docs.ros.org/en/jazzy/Installation/Ubuntu-Install-Debs.html#ubuntu-deb-packages), which
needs to be installed first. Additionally, this code base depends on `legged_control2`.

### Install `legged_control2`

Pre-built binaries for `legged_control2` are available on ROS 2 Jazzy. We recommend first reading
the [full documentation](https://qiayuanl.github.io/legged_control2_doc/overview.html).

Specifically, For this repo, follow
the [Debian Source installation](https://qiayuanl.github.io/legged_control2_doc/installation.html#debian-source-recommended).
Additionally, install Unitree-specific packages:

```bash
source /opt/ros/jazzy/setup.bash
sudo rosdep init
rosdep update
```

```bash
# Add debian source
echo "deb [trusted=yes] https://github.com/qiayuanl/unitree_buildfarm/raw/noble-jazzy-amd64/ ./" | sudo tee /etc/apt/sources.list.d/qiayuanl_unitree_buildfarm.list
echo "yaml https://github.com/qiayuanl/unitree_buildfarm/raw/noble-jazzy-amd64/local.yaml jazzy" | sudo tee /etc/ros/rosdep/sources.list.d/1-qiayuanl_unitree_buildfarm.list
sudo apt-get update
```

```bash
# Install packages
sudo apt-get install ros-jazzy-unitree-description
sudo apt-get install ros-jazzy-unitree-systems
```

### Build Package

After installing `legged_control2`, you can build this package. You’ll also need the
`unitree_bringup` repo, which contains utilities not included in the pre-built binaries.

Create a ROS 2 workspace if you don't have one. Below we use `~/colcon_ws` as an example.

```bash
mkdir -p ~/colcon_ws/src
```

Clone two repo into the `src` of workspace.

```bash
cd ~/colcon_ws/src
git clone https://github.com/qiayuanl/unitree_bringup.git
git clone https://github.com/HybridRobotics/motion_tracking_controller.git
cd ../
```

Install dependencies automatically:

```bash
rosdep install --from-paths src --ignore-src -r -y
```

Build the packages:

```bash
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=RelwithDebInfo --packages-up-to unitree_bringup
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=RelwithDebInfo --packages-up-to motion_tracking_controller
source install/setup.bash
```

### Verify Installation
```bash
source /opt/ros/jazzy/setup.bash
source ~/colcon_ws/install/setup.bash

ros2 pkg list | grep motion_tracking_controller
ros2 pkg list | grep unitree_bringup
```

## Basic Usage

### Sim-to-Sim

We provide a launch file for running the policy in MuJoCo simulation.

```bash
# Load policy from WandB
ros2 launch motion_tracking_controller mujoco.launch.py wandb_path:="$HOME/<your_wandb_run_path>"
```

```bash
# OR load policy from local ONNX file (should be absolute or start with `~`)
ros2 launch motion_tracking_controller mujoco.launch.py policy_path:="$HOME/<your_onnx_file_path>"
```