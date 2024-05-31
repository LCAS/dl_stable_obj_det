FROM nvidia/cudagl:11.3.0-devel-ubuntu20.04 as base

##############################################
# You should modify this to match your GPU compute capability
ENV TORCH_CUDA_ARCH_LIST="6.0 6.1 6.2 7.0 7.2 7.5 8.0 8.6"

ENV TORCH_NVCC_FLAGS="-Xfatbin -compress-all"

# For faster build, use more jobs, but not too much! 
ENV MAX_JOBS=6

ENV ROS_DISTRO foxy

##############################################
# Minimal ubuntu setup
RUN apt update --fix-missing && apt install -y locales lsb-release && apt clean

ARG DEBIAN_FRONTEND=noninteractive
RUN dpkg-reconfigure locales

##############################################
# Setup torch, cuda for the model and other dependencies
RUN apt install -y python3-pip && \
    pip install torch==1.12.0+cu113 torchvision==0.13.0+cu113 torchaudio==0.12.0 \
        --extra-index-url https://download.pytorch.org/whl/cu113

# Update the package repository and install dependencies
RUN apt update && apt install -y --no-install-recommends \
    git curl ninja-build cmake libopenblas-dev xauth openssh-server \
    && apt clean && rm -rf /var/lib/apt/lists/*

##############################################
# Install MinkowskiEngine
RUN git clone --recursive "https://github.com/NVIDIA/MinkowskiEngine" \
    && cd MinkowskiEngine \
    && python3 setup.py install --force_cuda --blas=openblas

##############################################
# Install ROS
RUN apt update && apt install locales -y software-properties-common 
RUN add-apt-repository universe
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# RUN sh -c 'echo "deb https://lcas.lincoln.ac.uk/apt/lcas $(lsb_release -sc) lcas" > /etc/apt/sources.list.d/lcas-latest.list' && \
#     curl -s https://lcas.lincoln.ac.uk/apt/repo_signing.gpg | tee /etc/apt/trusted.gpg.d/lcas-latest.gpg

RUN apt update && apt upgrade -y && apt install --no-install-recommends -y ros-foxy-desktop python3-argcomplete

RUN apt-get update \
    && apt-get install -qq -y --no-install-recommends \
    python3-pip \
    python3-rosdep \
    python3-colcon-common-extensions

RUN apt-get update && apt-get install -qq -y --no-install-recommends \
    python-is-python3 \
    less python3-pip net-tools iproute2 \
    xterm ros-dev-tools \
    openvpn \
    ros-${ROS_DISTRO}-interactive* ros-${ROS_DISTRO}-twist*  libompl-dev \
    ros-${ROS_DISTRO}-tf2-tools locate \
    ros-${ROS_DISTRO}-rclpy-message-converter v4l-utils ffmpeg \
    libomp-dev libboost-all-dev ros-${ROS_DISTRO}-pcl-ros 

RUN pip3 install tensorboard install scipy

# add user ros with sudo rights
RUN useradd -ms /bin/bash ros && echo "ros:ros" | chpasswd && adduser ros sudo

# add sudo without password
RUN echo "ALL ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

RUN echo "source /opt/ros/foxy/setup.bash" >> /etc/bash.bashrc

USER ros
