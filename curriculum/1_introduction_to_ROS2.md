# Introduction to ROS2
## Therory
Slides can be found in the gdrive folder.

## Environment setup
Fork and clone the following repository: [ros2-intro](#)
Follow the instructions in the README.md file for building, running and using the container.

## Excersises
- [Using turtlesim, ROS2 and rqt](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Introducing-Turtlesim/Introducing-Turtlesim.html)
  - 1 turtlesim node
  - 2 teleop node
  - 3 rqt
  - 4 spawn service    
- [Understanding ROS2 nodes](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Nodes/Understanding-ROS2-Nodes.html)
  - 5 what is node
  - 6 remap a node
  - 7 node introspection
- [Understanding ROS2 topics](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Topics/Understanding-ROS2-Topics.html)
  - 8 read rqt graph
  - 9 topics introspection
  - 10 commandline tools for tasks of topics
- [Understanding ROS2 sevices](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Services/Understanding-ROS2-Services.html)
  - 11 what is service
  - 12 commandline tools for tasks of services
- [Understanding ROS2 parameters](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Parameters/Understanding-ROS2-Parameters.html)
  - 13 what is parameter
  - 14 commandline tools for tasks of parameters
- [Understanding ROS2 actions](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Understanding-ROS2-Actions/Understanding-ROS2-Actions.html)
  - 15 what is action
  - 16 commandline tools for tasks of actions
- [File system in ROS2](https://docs.ros.org/en/humble/Tutorials/Beginner-Client-Libraries/Colcon-Tutorial.html)
  - 17 new build tools in ROS2
  - 18 Package structure after build
  - 19 workspace file structure
- [Data record in ROS2](https://docs.ros.org/en/humble/Tutorials/Beginner-CLI-Tools/Recording-And-Playing-Back-Data/Recording-And-Playing-Back-Data.html) 
  - 20 record selected topics
  - 21 play the bag
- [DDS middleware of ROS2](https://docs.ros.org/en/humble/Concepts/Intermediate/About-Domain-ID.html)
  - check environment variables
  Use the following command to check the environment settings of ROS2

  ```
  printenv | grep -i ROS
  ```
  Assign the ROS2 domain a value
  ```
  export ROS_DOMAIN_ID=1
  ```
  To keep the commnication to localhost
  ```
  export ROS_LOCALHOST_ONLY=1
  ```
## Assignments

In order to edit code/ files inside the container you will need to attach vscode to the container.

- click on the docker extension in vscode
- select the running container (green play button)
- right click and select attach to container

- Create a package called `my_package` with a launch file called `my_launch.launch.py` that launches the following nodes:
- `ros2 run turtlesim turtlesim_node`
- `ros2 run turtlesim turtle_teleop_key`

## Reference
- [ROS2 Humble Tutorials](https://docs.ros.org/en/humble/Tutorials/Intermediate/Launch/Launch-Main.html)
- [ROS2 Humble for beginners](https://www.youtube.com/watch?v=0aPbWsyENA8&list=PLLSegLrePWgJudpPUof4-nVFHGkB62Izy)  
