# ==================================================================
# module list
# ------------------------------------------------------------------
# python        3.8    (apt)
# pytorch       latest (pip)
# ==================================================================

ARG DEBIAN_FRONTEND=noninteractive
ARG BASE_DIST=ubuntu20.04
ARG CUDA_VERSION=11.4.2
FROM nvcr.io/nvidia/cudagl:${CUDA_VERSION}-devel-${BASE_DIST}

ENV LANG C.UTF-8
ENV TZS=Europe/London
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone


# ==================================================================
# User setting
# ------------------------------------------------------------------

ARG USER
ARG PASSWD
ARG USER_ID

RUN apt-get update && \
    apt-get install -y sudo

ENV HOME /home/$USER

RUN useradd -m $USER -u $USER_ID && \
    gpasswd -a $USER sudo && \
    echo "$USER:$PASSWD" | chpasswd && \
    echo 'Defaults visiblepw' >> /etc/sudoers && \
    echo "$USER ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER $USER
WORKDIR $HOME

RUN APT_INSTALL="sudo apt-get install -y --no-install-recommends" && \
    PIP_INSTALL="python -m pip --no-cache-dir install --upgrade" && \
    GIT_CLONE="git clone --depth 10" && \

    sudo rm -rf /var/lib/apt/lists/* \
           /etc/apt/sources.list.d/cuda.list \
           /etc/apt/sources.list.d/nvidia-ml.list && \

    sudo apt-get update && \

# ==================================================================
# tools
# ------------------------------------------------------------------

    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        build-essential \
        apt-utils \
        ca-certificates \
        wget \
        git \
        vim \
        libssl-dev \
        curl \
        unzip \
        unrar \
        zsh \
        neovim \
        tmux \
        && \

    $GIT_CLONE https://github.com/Kitware/CMake ~/cmake && \
    cd ~/cmake && \
    ./bootstrap && \
    sudo make -j"$(nproc)" install

# ==================================================================
# zsh
# ------------------------------------------------------------------
RUN APT_INSTALL="sudo apt-get install -y --no-install-recommends" && \
    PIP_INSTALL="python -m pip --no-cache-dir install --upgrade" && \
    GIT_CLONE="git clone --depth 10" && \

    zsh && \
    sudo chsh -s $(which zsh) && \

# ==================================================================
# python
# ------------------------------------------------------------------

    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        software-properties-common \
        && \
    sudo add-apt-repository ppa:deadsnakes/ppa && \
    sudo apt-get update && \
    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        python3-pip \
        python3.8 \
        python3.8-dev \
        python3-distutils-extra \
        && \
    wget -O ~/get-pip.py \
        https://bootstrap.pypa.io/get-pip.py && \
    sudo ln -s /usr/bin/python3 /usr/local/bin/python && \
    $PIP_INSTALL \
        numpy \
        scipy \
        pandas \
        cloudpickle \
        scikit-image>=0.14.2 \
        scikit-learn \
        matplotlib \
        Cython \
        tqdm \
        h5py \
        && \

# ==================================================================
# pytorch
# ------------------------------------------------------------------

    $PIP_INSTALL \
        future \
        numpy \
        protobuf \
        enum34 \
        pyyaml \
        typing

ENV LD_LIBRARY_PATH $HOME/.mujoco/mujoco210/bin:${LD_LIBRARY_PATH}

ENV LD_LIBRARY_PATH /usr/local/nvidia/lib64:${LD_LIBRARY_PATH}

# ===================================================================
# Install ROS noetic dependencies.
# ===================================================================

RUN echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" | sudo tee /etc/apt/sources.list.d/ros-latest.list
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

RUN APT_INSTALL="sudo apt-get install -y --no-install-recommends" && \
    sudo apt-get update && \
    DEBIAN_FRONTEND=noninteractive $APT_INSTALL \
        pkg-config \
        libxau-dev \
        libxdmcp-dev \
        libxcb1-dev \
        libxext-dev \
        libx11-dev \
        git-core \
        python-argparse \
        python3-rosdep \
        python3-rosinstall \
        python3-vcstools \
        python3-wstool \
        ros-noetic-desktop-full \
        ros-noetic-actionlib \
        ros-noetic-actionlib-msgs \
        ros-noetic-control-msgs \
        ros-noetic-cv-bridge \
        ros-noetic-rospy-message-converter \
        ros-noetic-rviz \
        ros-noetic-tf2-ros \
        ros-noetic-trajectory-msgs \
        ros-noetic-xacro \
        ros-noetic-moveit \
        ros-noetic-mocap-optitrack \
        ros-noetic-joy \
        ros-noetic-combined-robot-hw \
        ros-noetic-dynamic-reconfigure \
        build-essential \
        cmake \
        libpoco-dev \
        libeigen3-dev \
        dbus-x11 && \
    sudo apt-get clean && \
    sudo rm -rf /var/lib/apt/lists/*


RUN \
    sudo rosdep init && \
    rosdep update

RUN mkdir -p ~/catkin_ws/src


RUN pip install numpy-quaternion


# nvidia-container-runtime
ENV NVIDIA_VISIBLE_DEVICES \
    ${NVIDIA_VISIBLE_DEVICES:-all}
ENV NVIDIA_DRIVER_CAPABILITIES \
    ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics
ENV LIBGL_ALWAYS_INDIRECT 0


RUN sudo apt-get update && \
    sudo apt-get -y install \
    libosmesa6-dev

Run pip install tensorboardX \
                moviepy \
                imageio \
                funcsigs
# ==================================================================
# config & cleanup
# ------------------------------------------------------------------

RUN mkdir -p ~/projects

RUN sudo ldconfig && \
    sudo apt-get clean && \
    sudo apt-get autoremove && \
    sudo rm -rf /var/lib/apt/lists/* /tmp/*
