FROM osrf/ros:noetic-desktop-full

##################
# parameters
##################

# Set IP of Franka control PC
ENV FCI_IP=172.31.0.200

##################
# libfranka build
##################

# Download and build the required franka libraries
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    libpoco-dev \
    libeigen3-dev

############################################
# to do pinocchio
############################################
# Install extra dependencies for Pinocchio
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    libpoco-dev \
    libeigen3-dev \
    libfmt-dev \
    lsb-release \
    curl

# Add the robotpkg repository key and list for robotpkg-pinocchio
RUN mkdir -p /etc/apt/keyrings \
    && curl -fsSL http://robotpkg.openrobots.org/packages/debian/robotpkg.asc | tee /etc/apt/keyrings/robotpkg.asc \
    && echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/robotpkg.asc] http://robotpkg.openrobots.org/packages/debian/pub $(lsb_release -cs) robotpkg" | tee /etc/apt/sources.list.d/robotpkg.list\
    && apt-get update \
    && apt-get install -y robotpkg-pinocchio

############################################

RUN git clone --recursive https://github.com/frankaemika/libfranka --branch 0.13.3   #0.15.0
WORKDIR /libfranka
RUN git submodule update
RUN mkdir build
WORKDIR /libfranka/build
RUN cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/opt/openrobots/lib/cmake -DBUILD_TESTS=OFF ..
RUN cmake --build .

# Make a Debian package and install it
RUN cpack -G DEB
RUN dpkg -i libfranka*.deb

##################
# franka_ros build
##################

# Setup ROS catkin workspace
WORKDIR /catkin_ws
RUN mkdir src
SHELL ["/bin/bash", "-c"]
# so we can use `source`
RUN source /opt/ros/noetic/setup.sh && catkin_init_workspace src

# Download and build
RUN git clone --recursive https://github.com/frankaemika/franka_ros src/franka_ros
#COPY cartesian_impedance_example_controller_no_marker.launch src/franka_ros/franka_example_controllers/launch/cartesian_impedance_example_controller_no_marker.launch
RUN rosdep install --from-paths src --ignore-src --rosdistro noetic -y --skip-keys libfranka
RUN source /opt/ros/noetic/setup.sh && catkin_make -DCMAKE_BUILD_TYPE=Release -DFranka_DIR:PATH=/libfranka/build

# Add lines to the bashrc file that source ROS
RUN echo "source /ros_entrypoint.sh" >> /root/.bashrc
RUN echo "source /catkin_ws/devel/setup.bash" >> /root/.bashrc

###############################
# serl-franka_controllers build
###############################

WORKDIR /catkin_ws/src
RUN git clone https://github.com/rail-berkeley/serl_franka_controllers.git
WORKDIR /catkin_ws
RUN source /opt/ros/noetic/setup.sh && source devel/setup.sh && catkin_make --pkg serl_franka_controllers

