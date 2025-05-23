# Use Ubuntu Focal as the base image
FROM ubuntu:focal

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive

# Install dependencies
RUN apt-get update && apt-get install -y \
    git \
    sudo \
    net-tools \
    python3-pip \
    vim \
    nano

# Pre-configure the keyboard layout options to avoid interactive prompts
RUN apt-get update && \
    apt-get install -y debconf-utils && \
    echo 'keyboard-configuration keyboard-configuration/layoutcode select us' | debconf-set-selections && \
    echo 'keyboard-configuration keyboard-configuration/model select Generic 105-key PC (intl.)' | debconf-set-selections

# Setup locale
RUN apt-get update && apt-get install -y locales tzdata && \
    locale-gen en_US en_US.UTF-8 && \
    update-locale LC_ALL=en_US.UTF-8 LANG=en_US.UTF-8 && \
    ln -fs /usr/share/zoneinfo/America/New_York /etc/localtime && \
    dpkg-reconfigure --frontend noninteractive tzdata
ENV LANG=en_US.UTF-8
ENV LC_ALL=en_US.UTF-8

# Set up mocap user
RUN useradd -m mocap

# Install Dependencies for ROS2 Humble
RUN sudo apt-get update && sudo apt-get install -y \
    gnupg \
    wget \
    software-properties-common && \
    sudo add-apt-repository universe

# Setup Source
RUN wget -qO - https://isaac.download.nvidia.com/isaac-ros/repos.key | sudo apt-key add - && \
    grep -qxF "deb https://isaac.download.nvidia.com/isaac-ros/ubuntu/main $(lsb_release -cs) main" /etc/apt/sources.list || \
    echo "deb https://isaac.download.nvidia.com/isaac-ros/ubuntu/main $(lsb_release -cs) main" | sudo tee -a /etc/apt/sources.list && \
    sudo apt-get update && \
    sudo apt-get update && sudo apt-get install curl -y && \
    sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | sudo tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Install core ROS2 Packages
RUN sudo apt-get update && sudo apt-get install -y ros-humble-desktop

# Install Dependencies for Mocap4ROS2
RUN apt-get install -y python3-vcstool \
    python3-colcon-common-extensions

# Clean up
RUN rm -rf /var/lib/apt/lists/* && \
    apt-get clean

# Switch to mocap user
USER mocap

# Set the working directory for the mocap user
WORKDIR /home/mocap/ros2_ws/src

# Clone mocap4ros2 repository
RUN git clone --recursive https://github.com/MOCAP4ROS2-Project/mocap4ros2_qualisys.git

# Install dependencies
RUN vcs import < mocap4ros2_qualisys/dependency_repos.repos

# Build mocap4ros2
WORKDIR /home/mocap/ros2_ws
RUN /bin/bash -c "source /opt/ros/humble/setup.bash && colcon build --symlink-install"

# Add start up commands to .bashrc
RUN echo "source /opt/ros/humble/setup.bash" >> /home/mocap/.bashrc && \
    echo "source /home/mocap/ros2_ws/install/setup.bash" >> /home/mocap/.bashrc && \
    echo "export ROS_IP=$(hostname -I | awk '{print $1}')" >> /home/mocap/.bashrc && \
    echo "export ROS_DOMAIN_ID=1" >> /home/mocap/.bashrc

# Clone VIO-UAV directory
WORKDIR /home/mocap
RUN git clone https://github.com/bandofpv/VIO-UAV.git

# Start in VIO-UAV directory
WORKDIR /home/mocap/VIO-UAV

# Set up entrypoint
ENTRYPOINT ["/bin/bash"]
