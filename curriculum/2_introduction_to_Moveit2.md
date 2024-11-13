# Introduction to MoveIt

## Theory

Slides can be found in the software II gdrive folder.

- What is MoveIt!
- URDF / SRDF
- Kinematic solvers
- Motion planners
- ROS manipulator architecture
- Moveit components
- ROS control

## Exercises

### Environment setup

[moveit2-introduction](https://github.com/MRAC-IAAC/moveit2-introduction)

Fork and clone the following repo: moveit2-introduction
See repo README for instructions on building, running and using the container.

If you are experienced a version of the repository with git submodules is available here: [Universal_Robots_ROS2_Driver](https://github.com/HuanyuL/Universal_Robots_ROS2_Driver/tree/humble), [pymoveit2](https://github.com/HuanyuL/pymoveit2)

### Overview of environment

Contents of container:

- ROS Humble
- Moveit

  - IK
    - KDL

  - Planning pipelines
    - OMPL
    - Pilz Industrial Motion Planner

Overview of the environment:

- universal_robot/moveit_configs
- pymoveit2
- rviz


### Interfacing with Moveit using Python (pymoveit2_examples)

- Issue of python binding for ROS2 Humble
- Action node, basic python interface from pymoveit2
- Planning with a goal
- Planning with joint position
- Add collison mesh
- Moveit GUI experience

## References

### Moveit2

- [moveit2 humble documentation](https://moveit.picknik.ai/humble/index.html)
- [moveit2 examples](https:/https://moveit.picknik.ai/humble/doc/examples/examples.html)
- [pymoveit2](https://github.com/AndrejOrsula/pymoveit2)
- [pilz_robot_programming repo](https://github.com/PilzDE/pilz_industrial_motion/tree/melodic-devel/pilz_robot_programming)

### UR

- [ur IO service](https://github.com/ros-industrial/ur_msgs/tree/humble?tab=readme-ov-file)