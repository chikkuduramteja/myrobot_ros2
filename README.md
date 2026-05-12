# myrobot_ros2

ROS 2 Humble workspace packages for simulating a small robotic arm in Gazebo with ROS 2 Control.

This project demonstrates:

- loading a robot model in Gazebo
- publishing robot state to RViz
- starting ROS 2 Control controllers
- moving the arm to a goal joint position
- closing the gripper
- returning the arm to home
- opening the gripper
- running the same motion sequence from a Python node

The Gazebo world includes simple primitive shapes for scene context. The main demo focuses on commanded arm motion and gripper open/close behavior, not physical object grasping. We tested object picking, but reliable grasping requires additional Gazebo contact tuning or an attach/grasp plugin, so that is listed as future work.

## Requirements

- Ubuntu 22.04
- ROS 2 Humble
- Gazebo / ros_gz packages
- colcon

Install the common dependencies:

```bash
sudo apt update && sudo apt install -y ros-humble-ament-cmake ros-humble-ament-lint-auto ros-humble-ament-lint-common ros-humble-control-msgs ros-humble-controller-manager ros-humble-gripper-controllers ros-humble-gz-ros2-control ros-humble-gz-ros2-control-demos ros-humble-moveit-configs-utils ros-humble-moveit-kinematics ros-humble-moveit-planners-ompl ros-humble-moveit-ros-move-group ros-humble-moveit-ros-planning-interface ros-humble-moveit-ros-visualization ros-humble-moveit-simple-controller-manager ros-humble-pilz-industrial-motion-planner ros-humble-rclcpp ros-humble-rclcpp-action ros-humble-ros-gz ros-humble-ros-gz-bridge ros-humble-ros-gz-image ros-humble-ros-gz-sim ros-humble-ros2-control ros-humble-ros2-controllers ros-humble-rviz-visual-tools ros-humble-rviz2 ros-humble-trajectory-msgs ros-humble-urdf-tutorial ros-humble-xacro python3-numpy
```

## Build

From the workspace root:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
rosdep install -i --from-path src --rosdistro humble -y --skip-keys="gz_msgs_vendor"
colcon build
source install/setup.bash
```


## Launch The Demo

In terminal 1:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
export LIBGL_ALWAYS_SOFTWARE=1
export MESA_GL_VERSION_OVERRIDE=4.5
bash ~/ros2_ws/src/myrobot_ros2/myrobot_bringup/scripts/myrobot_280_gazebo.sh
```

The `LIBGL_ALWAYS_SOFTWARE` and `MESA_GL_VERSION_OVERRIDE` lines help avoid Gazebo OGRE/OpenGL crashes on some WSL systems.

## Inspect The Running System

In terminal 2:

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
source install/setup.bash
ros2 control list_controllers
ros2 control list_hardware_components
ros2 topic list
```

Expected active controllers:

- `joint_state_broadcaster`
- `arm_controller`
- `gripper_action_controller`

## Manual Motion Commands

Move the arm to the demo goal position:

```bash
ros2 topic pub --once /arm_controller/joint_trajectory trajectory_msgs/msg/JointTrajectory "{joint_names: ['link1_to_link2', 'link2_to_link3', 'link3_to_link4', 'link4_to_link5', 'link5_to_link6', 'link6_to_link6_flange'], points: [{positions: [1.295, -1.85, 1.05, -1.05, 0.389, -1.5], time_from_start: {sec: 3, nanosec: 0}}]}"
```

Close the gripper:

```bash
ros2 action send_goal /gripper_action_controller/gripper_cmd control_msgs/action/GripperCommand "{command: {position: -0.7, max_effort: 100.0}}"
```

Move the arm back home:

```bash
ros2 topic pub --once /arm_controller/joint_trajectory trajectory_msgs/msg/JointTrajectory "{joint_names: ['link1_to_link2', 'link2_to_link3', 'link3_to_link4', 'link4_to_link5', 'link5_to_link6', 'link6_to_link6_flange'], points: [{positions: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0], time_from_start: {sec: 2, nanosec: 0}}]}"
```

Open the gripper:

```bash
ros2 action send_goal /gripper_action_controller/gripper_cmd control_msgs/action/GripperCommand "{command: {position: 0.15, max_effort: 100.0}}"
```

## Python Motion Demo

The Python node runs the same overall behavior in a loop:

1. move to the goal joint position `[1.295, -1.85, 1.05, -1.05, 0.389, -1.5]`
2. close the gripper
3. move to the home joint position `[0.0, 0.0, 0.0, 0.0, 0.0, 0.0]`
4. open the gripper

Run it from terminal 2 while Gazebo is running:

```bash
ros2 run myrobot_system_tests arm_gripper_loop_controller.py
```

Stop with `Ctrl+C`.

## Demo Explanation

This demo shows the ROS 2 Control pipeline:

1. A trajectory command is sent to `/arm_controller/joint_trajectory`.
2. The joint trajectory controller commands the simulated arm joints.
3. Gazebo applies those commands through `gz_ros2_control`.
4. The gripper action controller receives open and close commands.
5. RViz and Gazebo visualize the robot state and motion.

The simple shapes in the scene are visual context only. The current demo does not guarantee physical pick-and-place because reliable grasping in Gazebo needs contact tuning, friction tuning, or an attach/grasp plugin.

## Future Work

- Add a reliable Gazebo grasp or attach plugin.
- Tune gripper contact/friction for physical object lifting.
- Add MoveIt planning for target-based pick-and-place.
- Add camera-based perception for detecting the object pose.
