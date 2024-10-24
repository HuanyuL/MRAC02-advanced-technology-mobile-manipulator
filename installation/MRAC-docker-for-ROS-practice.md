# ROS/ROS2 with Docker for Robots and Sensors at IAAC

Author: [Huanyu Li](https://github.com/HuanyuL)

## Attribution

Sections of this README, including "Challenges in Software Deployment" and "Docker to the Rescue," are adapted from content originally found in [docker-for-robotics](https://github.com/2b-t/docker-for-robotics/tree/main). Full credit goes to the original authors for their contributions to the explanation of these topics.
## 1. Overview

The repository demonstrates how containerization can streamline the management of ROS/ROS2 environments, drivers, and dependencies for the various robots and sensors at IAAC.

By using Docker, we create isolated, reproducible environments that allow developers and researchers to easily deploy, manage, and interact with robotics systems and their sensors. This helps overcome the complexities of managing dependencies, multiple software versions, and different setups for each robot or sensor.

## 2. Challenges in Software Deployment

Deploying a piece of software in a portable manner is a non-trivial task. Clearly there are different operating system and different software architectures which require different binary code, but even if these match you have to make sure that the compiled code can be executed on another machine by supplying all its **dependencies**.

Over the years several different packaging systems for different operating systems have emerged that provide methods for installing new dependencies and managing existing ones in an coherent manner. The low-level package manager for Debian-based Linux operating systems is [`dpkg`](https://wiki.debian.org/Teams/Dpkg), while for high-level package management, fetching packages from remote locations and resolving complex package relations, generally [`apt`](https://wiki.debian.org/Apt) is chosen. `apt` handles retrieving, configuring, installing as well as removing packages in an automated manner. When installing an  `apt` package it checks the existing dependencies and installs only those that are not available yet on the system. The dependencies are shared, making the packages smaller but not allowing for multiple installations of the same library and potentially causing issues between applications requiring different versions of the same library. Contrary to this the popular package manager [`snap`](https://snapcraft.io/) uses self-contained packages which pack all the dependencies that a program requires to run, allowing for multiple installations of the same library.  **Self-contained** boxes like these are called **containers**, as they do not pollute the rest of the system and might only have limited access to the host system. The main advantage of containers is that they provide clean and conistent environments as well as isolation from the hardware.

## 3. Docker to the rescue

[**Docker**](https://www.docker.com/) is another **framework** for working with **containers**. A [Docker - contrary to `snap`](https://www.youtube.com/watch?v=0z3yusiCOCk) - is not integrated in terms of hardware and networking but instead has its own IP address, adding an extra layer of abstraction. A Docker container is similar to a virtual machine but the containers share the same kernel like the host system: Docker does not **virtualise** on a hardware level but on an **app level** (OS-level virtualisation). For this Docker builds on a virtualization feature of the Linux kernel, [namespaces](https://en.wikipedia.org/wiki/Linux_namespaces), that allows to selectivelty grant processes access to kernel resources. As such Docker has its own namespaces for `mnt`, `pid`, `net`, `ipc` as well as `usr` and its own root file system. As a Docker container uses the same kernel, and as a result also the same scheduler one might achieve native performance. At the same time this results in issues with graphic user interfaces as these are not part of the kernel itself and thus not shared between the container and the host system. These problems can be worked around though mostly.

Using Docker brings a couple of advantages as it strongly leverages on the decoupling of the kernel and the rest of the operating system:

- **Portability**: You can run code not intended for your particular Linux distribution (e.g packages for Ubuntu 20.04 on Ubuntu 18.04 and vice versa) and you can mix them, launching several containers with different requirements on the same host system by means of dedicated [orchestration tools](https://docs.docker.com/get-started/orchestration/) such as [Kubernetes](https://kubernetes.io/) or [Docker Swarm](https://docs.docker.com/engine/swarm/). This is a huge advantage for robotics applications as one can mix containers with different ROS distributions on the same computer running in parallel, all running on the same kernel of the host operating system, governed by the same scheduler.
- **Performance**: Contrary to a virtual machine the performance penalty is very small and for most applications is indistinguishable from running code on the host system: After all it uses same kernel and scheduler as the host system.
- Furthermore one can also run a **Linux container on a Windows or MacOS operating system**. This way you lose though a couple of advantages of Docker such as being able to run real-time code as there will be a light-weight virtual machine underneath emulating a Linux kernel. Furthermore you can also use it from the **Windows Subsystem for Linux** which allows you to stream graphic user interfaces onto the host system through X-Server.

This way one can guarantee a **clean, consistent and standardised build environment** while maintaining encapsulation and achieving native performance.

The core component of Docker are so called **images**, *immutable read-only templates*, that hold source code, libraries and dependencies. These can be layered over each other to form more complex images. **Containers** on the other hand are the *writable layer* on top of the read-only images. By starting an image you obtain a container: Images and containers are not opposing objects but they should rather be seen as different phases of building a containerised application.

The Docker **daemon** software manages the different containers that are available on the system: The generation of an image can be described by a so called **`Dockerfile`**. A Dockerfile is like a recipe describing how an image can be created from scratch. This file might help also somebody reconstruct the steps required to get a code up and running on a vanilla host system without Docker. It is so to speak self-documenting and does not result in an additional burden like a wiki. Similarly one can recover the steps performed to generate an image with [`$ docker history --no-trunc <image_id>`](https://docs.docker.com/engine/reference/commandline/history/). Dedicated servers, so calle **Docker registries** (such as the [Docker Hub](https://hub.docker.com/) or [Github's GHCR](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-container-registry)), allow you to store and distribute your Docker images. These image repositories hold different images so that one does not have to go through the build process but instead can upload and download them directly, speeding up deployment. Uploads might also be triggered by a continuous integration workflow like outlined [here](https://docs.github.com/en/actions/publishing-packages/publishing-docker-images).

On top of this there go other toolchains for managing the lifetime of containers and orchestration multiple of them such as [Docker-Compose](https://docs.docker.com/compose/), [Swarm](https://docs.docker.com/engine/swarm/) or [Kubernetes](https://kubernetes.io/).

This makes Docker in particular suitable for **deploying source code in a replicable manner** and will likely speed-up your development workflow. Furthermore one can use the description to perform tests or compile the code on a remote machine in terms of [continuous integration](https://en.wikipedia.org/wiki/Continuous_integration). This means for most people working professional on code development it comes at virtually no cost.

## 4. Installation
### 4.1 Install docker engine on Ubuntu
The installation guide for **Ubuntu** can be found [here](https://docs.docker.com/engine/install/ubuntu/). Recommend to install using the `apt` repository
### 4.2 Post-Installation steps
Create the group to allow docker can be run without sudo, the instructions can be found [here](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user). Please ensure the systemd initiates the docker service when the system boots.
```
sudo systemctl enable docker.service
sudo systemctl enable containerd.service
```
### 4.3 Hardware acceleration with NVIDIA cards
This section is only relevant for NVIDIA graphic cards managed by the NVIDIA driver or if you want to have hardware acceleration inside the Docker, e.g. for using CUDA or OpenGL. Graphic user interfaces that do not require it will work fine in any case. Please ensure your graphic card driver metapackage is from nvidia-driver-(proprietary version). You can install NVIDIA contianer toolkit from following command.

```
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list
```
Update the packages list from the repository:
```
sudo apt-get update
```
Install the NVIDIA Container Toolkit packages:
```
sudo apt-get install -y nvidia-container-toolkit
```
Configure the container:
```
sudo nvidia-ctk runtime configure --runtime=docker
```
Restart docker daemon:
```
sudo systemctl restart docker
```

## 5. Visual studio code setup
Setting up VSCode to work with Docker involves using the Remote - Containers extension, which allows you to open any folder inside a Docker container, giving you the ability to use all of Visual Studio Code's features (debugging, IntelliSense, extensions, etc.) within the container. This is particularly useful for creating isolated, reproducible development environments.

### 5.1 Install the required extensions
In VSCode:
- Open the Extensions View by clicking the Extension icon on you sidebar(or press ctrl+shift+X).
- Search for **"Docker"** and **"Remote Development"** and install it

### 5.2 Usage
- Once your container is up and running, find the container which is running from the docker icon on the sidebar, **right click** on the container and select **attach to visual studio code**. 
- A new window will establish and you will find the **Dev Container: <YOUR CONTAINER>** on the bottom left corner. 
- Click on the **Open Folder** from the **File** menu, change your work directory to `/dev_ws`(please make sure there's no parent folder in the path)

## 6. Docker file system and code breakdown

The file system of docker folder is designed to provide a clear, streamlined workflow for students to easily build and run a ROS/ROS2 environment using Docker. By organizing the project with key components like the `Dockerfile`, `build_image.sh`, `entrypoint.sh`, `run_user.sh` and `setup.bash`, it ensures that both Docker-based and local development environments are configured consistently and with minimal effort.

This workflow, crafted by [VincentHuyghe](https://github.com/vinceHuyghe), is designed to simplify the setup process, automating complex tasks like ROS networking, dependency installation, and workspace initialization, allowing students to focus on learning and development without dealing with setup issues.

### 6.1 `DockerFile` - Configuring the container's environment

The `Dockerfile` is a script used to define the environment and dependencies inside your Docker container. It ensures that the container has the necessary libraries, tools, and configurations to run your ROS-based project.

Now that we have seen how to start a container from an existing image let us build a `Dockerfile` that defines steps that should be executed on the image

The example is structured with comments explaining each part of the setup in detail. These comments are added in line with the relevant commands to clarify their purpose and how they contribute to building a ROS development environment.
```
# ARG sets a build-time argument. We define ROS_DISTRO as "noetic" by default,
# but it can be overridden during the build process.
ARG ROS_DISTRO=noetic

# Use the official ROS image as the base, with the specified ROS distribution (e.g., "noetic")
FROM ros:$ROS_DISTRO-ros-base

# Re-declare the ROS_DISTRO argument for use after the FROM command
ARG ROS_DISTRO

# Set environment variables to ensure consistent UTF-8 encoding and 
# a non-interactive frontend for apt-get to prevent prompts during installs.
ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
ENV TERM xterm-256color

# Install basic utilities and development tools needed for building and managing ROS projects
RUN apt-get update && apt-get install -y --no-install-recommends\
    ssh \                    # Secure Shell for remote access
    git \                    # Version control tool
    curl \                   # Command-line tool for transferring data
    wget \                   # Network downloader
    build-essential \        # GCC, G++, make, and other development tools
    cmake \                  # CMake for building C++ projects
    python3-pip \            # Python package manager
    python3-flake8 \         # Python linter for code quality
    terminator \             # Advanced terminal emulator
    && apt-get clean && rm -rf /var/lib/apt/lists/*  # Clean up to reduce image size

# Install ROS tools and additional dependencies specific to the ROS environment.
RUN apt-get update && apt-get install -y --no-install-recommends\
    pkg-config \                              # Tool for managing compilation of libraries
    python3-catkin-tools \                    # Catkin workspace build tool for ROS
    python3-rosdep \                          # ROS dependency management tool
    python3-rosinstall-generator \            # Tool to generate .rosinstall files
    ros-$ROS_DISTRO-rqt \                     # ROS graphical tool for introspection
    ros-$ROS_DISTRO-rqt-common-plugins \      # Common plugins for rqt
    ros-$ROS_DISTRO-rqt-robot-plugins \       # Plugins for visualizing robots in rqt
    ros-$ROS_DISTRO-roslint \                 # ROS linter for ensuring style consistency
    ros-$ROS_DISTRO-rqt-gui \                 # ROS graphical user interface
    ros-$ROS_DISTRO-rqt-gui-py \              # Python support for rqt GUI
    ros-$ROS_DISTRO-rqt-py-common \           # Common Python plugins for rqt
    ros-$ROS_DISTRO-rviz \                    # 3D visualization tool for ROS
    ros-$ROS_DISTRO-diagnostics \             # ROS diagnostic tools
    ros-$ROS_DISTRO-turtlesim \               # ROS turtlesim tutorial simulation
    ros-$ROS_DISTRO-ros-tutorials \           # Official ROS tutorials
    && apt-get clean && rm -rf /var/lib/apt/lists/*  # Clean up to reduce image size

# Set the working directory to the root directory of the container
WORKDIR /

# Create a workspace directory for ROS development
RUN mkdir -p dev_ws/src

# Copy the current directory's contents (from the host) into the container's ROS workspace
COPY . /dev_ws/src/

# Set the working directory to the ROS workspace
WORKDIR /dev_ws

# Source the ROS environment, configure the catkin workspace, and build the workspace
RUN ["/bin/bash", "-c", "source /opt/ros/$ROS_DISTRO/setup.bash &&\
    catkin config --cmake-args -DCMAKE_BUILD_TYPE=Release &&\
    catkin build \
    "]

# Copy the entrypoint script from the source directory to the container's root,
# and give it executable permissions.
RUN ["/bin/bash", "-c", "cp /dev_ws/src/.docker/entrypoint.sh /entrypoint.sh && chmod 777 /entrypoint.sh"]

# Copy the setup.bash script from the source directory to the ROS workspace,
# and give it executable permissions.
RUN ["/bin/bash", "-c", "cp /dev_ws/src/.docker/setup.bash /dev_ws/setup.bash && chmod 777 /dev_ws/setup.bash"]

# Set the container entrypoint to the entrypoint.sh script, ensuring the ROS environment is set up
ENTRYPOINT ["bash", "/entrypoint.sh" ]

# Default command that will be run if no other command is specified when starting the container.
# It opens a Bash shell to allow interaction with the container.
CMD ["bash"]
```
**Key sections**:
- **Base image**: The official ROS image provides a stable, pre-configured environment. Using the $ROS_DISTRO argument allows flexibility to switch between ROS distributions like noetic or melodic.
- **System Utilities**: Essential tools like `git`, `curl`, `build-essential`, and others are installed for compiling software and managing dependencies. These utilities are essential for software development and troubleshooting, whether it's installing additional packages, cloning repositories, or managing processes inside the container.
- **ROS Tools and Packages**: Key ROS development tools (e.g., `rqt`, `rviz`, `catkin-tools`) and distribution-specific packages are installed to create a complete ROS environment, ready for development and simulation.
- **Copying Project Files (COPY . /dev_ws/src)**: This command copies the source code and project files from the local system into the Docker container's workspace. By doing this, the container has access to the complete project structure, allowing it to build and run the ROS nodes inside. Additionally, since Git is installed, you can modify the project inside the container and push changes directly to a repository, making the Docker environment fully integrated with your version control workflow.

### 6.2 `build_image.sh` – Script to simplify building the docker image
The build_image.sh is a Bash script that simplifies the process of building the Docker image for students. Instead of remembering Docker commands, they can just run this script to build the container.This approach makes it easy for anyone to set up the development environment with a single command (./build_image.sh), rather than dealing with Docker commands directly. Let's see the example below.

```
#!/usr/bin/env bash
# This tells the system to use the bash shell to execute the script.

echo -e "Building ros_basic:latest image"
# Prints a message to the terminal indicating that the Docker image is being built.

DOCKER_BUILDKIT=1 \
# Enables Docker BuildKit, which offers better performance and additional features for building Docker images.

docker build --pull --rm -f ./.docker/Dockerfile \
# Starts the Docker build process.
# --pull: Ensures the latest version of the base image is pulled from the repository.
# --rm: Removes intermediate containers after the build to save disk space.
# -f ./.docker/Dockerfile: Specifies the path to the Dockerfile inside the .docker directory.

--build-arg BUILDKIT_INLINE_CACHE=1 \
# Passes a build argument to enable BuildKit's inline caching, which allows better use of cached image layers to speed up rebuilds.

--tag ros_basic:latest .
# Tags the final Docker image as 'ros_basic:latest', so it is easily identifiable locally.
# The dot (.) at the end specifies the build context, i.e., the current directory.

```

### 6.3 `entrypoint.sh` – Script to set up ROS environment
The entrypoint.sh script is responsible for setting up the environment when the container is started. This script sources the ROS setup files, initializes the workspace, and configures networking variables essential for ROS communication, such as ROS_MASTER_URI and ROS_IP.

When a container starts, it uses this script to ensure that the ROS environment is fully loaded, meaning users can immediately start working with ROS without additional configuration. This script also supports executing additional commands, allowing the container to run indefinitely or launch specific ROS nodes or processes based on the command passed when starting the container.

```
set -e
# This ensures that the script exits immediately if any command fails (non-zero exit status).
# It's a safety measure to prevent the script from continuing if something goes wrong.

# Setup ROS environment
source "/opt/ros/noetic/setup.bash"
# Sources the global ROS environment for the specified ROS distribution (in this case, Noetic).
# This sets up all necessary environment variables like ROS_PACKAGE_PATH.

source "/dev_ws/devel/setup.bash"
# Sources the workspace-specific environment, enabling the container or system
# to recognize custom packages and configurations built in the workspace (`/dev_ws`).

exec "$@"
# This replaces the current shell with the command provided as arguments to the script (`"$@"`).
# It allows the container or script to run whatever command was passed after the environment is set up.
```

### 6.4 `run_user.sh` or `run_user_nvidia.sh` - Launch the docker container with graphic features
This script launches a Docker container that provides a full ROS (Robot Operating System) environment while giving the user access to their home directory, graphical applications (via X server), and hardware devices on the host machine. By running the container with the same user ID and group as the host, it avoids permission issues when accessing files. It also sets up the container to interact with ROS tools and run processes with elevated privileges. The differences between the `run_user.sh` and `run_user_nvidia.sh` are the graphical acceleration from nvidia graphic card. If you have a nvidia graphic card and finished the [Nvidia Container Toolkit Installation](#43-hardware-acceleration-with-nvidia-cards), prioritize the launching method through graphic card.
```
#!/usr/bin/env bash
# The shebang instructs the system to use bash to interpret this script.

# Inform the user that the ros_basic container is being started
echo -e "Starting up ros_basic container \n >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>"
# Explains that the container will access the user's home directory and log in with their credentials
echo -e "This container will access the users home directory and log in as the user with their password and X server access.\nYou will not own the workspace though, use sudo chown -R \$USER /dev_ws"
# Reminder to source the workspace after entering the container
echo -e "Source the workspace with source devel/setup.bash"

# Run the Docker container with several options:
docker run -it --privileged \
    --user=$(id -u $USER):$(id -g $USER) \
    # The --user flag runs the container with the current user's ID and group from the host system,
    # ensuring that files created in the container will have correct ownership when accessed on the host.

    --group-add sudo \
    # Adds the 'sudo' group, allowing the user to run commands with sudo inside the container.

    --env="DISPLAY" \
    # Passes the DISPLAY variable to the container, which is needed to run GUI applications from the container
    # and display them on the host's screen.

    --env="QT_X11_NO_MITSHM=1" \
    # This environment variable avoids a known issue with shared memory in X11, ensuring better performance for
    # graphical applications in the container.

    --workdir="/dev_ws" \
    # Sets the working directory to the ROS workspace in the container, where ROS code and builds will be stored.

    --volume="/home/$USER:/home/$USER" \
    # Mounts the host user's home directory into the container, enabling access to personal files and configurations.

    --volume="/etc/group:/etc/group:ro" \
    # Mounts the /etc/group file in read-only mode, providing access to user group information from the host.

    --volume="/etc/passwd:/etc/passwd:ro" \
    # Mounts the /etc/passwd file in read-only mode, providing access to user account information from the host.

    --volume="/etc/shadow:/etc/shadow:ro" \
    # Mounts the /etc/shadow file in read-only mode, providing access to user password information from the host.

    --volume="/etc/sudoers.d:/etc/sudoers.d:ro" \
    # Mounts the /etc/sudoers.d directory in read-only mode, providing access to sudoers configuration files from the host.

    --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" \
    # Mounts the X11 Unix socket for graphical display, allowing GUI applications from the container to be displayed on the host.

```

## 7.Docker workflow for ROS projects
The workflow for any ROS project using Docker follows three layers or stacks: **Hardware Layer**, **Operation Layer**, and **Development Layer**. Each layer will be represented as a submodule with its specific Dockerfile modifications. The project file system will be in following structure

You have to modify the Dockerfile to configure dependencies specific to the layers. Each layer—hardware, operation, and development—may require different libraries, tools, or ROS packages. To ensure the container has all necessary dependencies.
```
Project/
│
├── .git/                         # Git repository folder (hidden by default)
│
├── .gitmodules                   # Git submodules configuration file
│
├── .docker/                      # Docker configuration files
│   ├── Dockerfile          
│   ├── build_image.sh      
|   ├── entrypoints.sh      
|   ├── ...
│
├── HARDWARE_LAYER(SUBMODULE)/    # First hardware layer, submodules can be found from this repository
│   ├── launch/
│   ├── src/
│   └── ...
│
├── OPERATION_LAYER(SUBMODULE)/   # Operation layer, submodules can be found from this repository
│   ├── launch/
│   ├── src/
│   └── ...
│
├── DEVELOPMENT_LAYER/            # Development layer, this layer is where you implement and containerize new features.
│   ├── launch/
│   ├── src/
│   └── ...
└── README.md                     # Project documentation and setup instructions
```

### 7.1 Initial consideration: Development vs. Data Extraction
In ROS projects, "development" refers to the process of creating, modifying, and improving components that allow robots to perform tasks. This encompasses writing node scripts that carry out specific functions, such as navigation or motor control, and building and managing packages to organize code, launch files, configuration files, and dependencies. It also includes creating custom message types for inter-node communication and defining services for request/response interactions. 

Conversely, if you are just tweaking data, such as adjusting parameters in a configuration file or fine-tuning a launch file for existing nodes, you may not need to create a new package, as this usually does not constitute full development. Similarly, assembling messages, which involves combining or processing data sent via topics, might not require a new package if it is a minor modification to existing code or nodes. However, if your efforts to assemble messages include writing new logic for message handling, creating new pipelines, or introducing new nodes that process or transform data, then it does count as development. While simple parameter adjustments may not qualify as full development, they remain part of the overall engineering process in ROS.

### 7.2 Building the container and managing changes with Git
When working with Docker containers, it’s important to remember that any changes made inside the container will be lost once you shut it down. To ensure continuous development, we use Git as our storage solution. Here’s the overall workflow for managing your development process:

- **Pull the last changes**: Begin by pulling the latest updates from the repository to ensure you have the most recent code. You can do this using the command:
```
git pull
```
- **Build the docker image**: Next, build the Docker image using the provided script. This can be done with the following command:
```
.docker/build_image.sh
```
- **Run the Container**: Depending on your requirements, you can run the container with one of the following scripts:
    - For a standard user container:
    ```
    .docker/run_user.sh
    ```
    - For a container with NVIDIA support (if you’re using GPUs):
    ```
    .docker/run_user_nvidia.sh
    ```
- **Change the ownership of the workspace**: The container will log in as the user with their password and x server access, to take the ownership of the workspace you have to run:
```
sudo chown -R $USER /dev_ws
```

- **Push Changes to Git**: After you’ve completed your development work, remember to push your changes back to the repository.

## 8. Future improvements

Using a single Docker image for all your ROS development can lead to significant limitations as project complexity increases. Managing dependencies and configurations becomes challenging, often resulting in a monolithic structure that is harder to maintain and troubleshoot. Performance issues may arise since all services run together, potentially causing resource-intensive nodes to bottleneck others. The lack of isolation means that problems in one service can impact the entire application, complicating debugging efforts. Scaling specific components becomes difficult, as you would need to rebuild the image to accommodate changes, hindering deployment and continuous integration practices. Development flexibility is limited, making it challenging to test individual components or configurations without affecting the entire system. Moreover, the growing image size can lead to longer build times, complicating image lifecycle management, while managing different versions of ROS or dependencies can result in conflicts.

Docker Compose development offers numerous benefits, particularly in facilitating a multi-container architecture that isolates different components, such as various ROS nodes, without conflict. By defining all services in a single docker-compose.yml file, you simplify configuration and networking, allowing seamless communication between containers. For example, a basic setup could include multiple ROS nodes running different commands while sharing a common workspace through mounted volumes. This approach not only makes it easier for students to experiment and modify individual components without impacting the entire system but also ensures a reproducible and collaborative development environment. Additionally, once the dependencies and industrial packages have stable version or finished we can move the contianers to docker hub which can be easily used by all users and running in both linux and windows system.
