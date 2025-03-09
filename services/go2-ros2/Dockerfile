FROM ros:humble-ros-base

# Install system dependencies
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    curl \
    gnupg2 \
    lsb-release \
    python3-pip \
    portaudio19-dev \
    mesa-utils \
    libgl1-mesa-glx \
    libgl1-mesa-dri \
    ros-humble-rviz2 \
    ros-humble-foxglove-bridge \
    software-properties-common

# Create workspace
WORKDIR /ros2_ws

# Install specific numpy version first with pip cache
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install 'numpy<2.0.0'

# Copy source code
COPY ./src /ros2_ws/src

# Install Python requirements with pip cache
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install -r /ros2_ws/src/requirements.txt

# Install ROS dependencies with cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /opt/ros/${ROS_DISTRO}/setup.sh && \
    rosdep install --from-paths src --ignore-src -r -y

# Build the workspace
RUN . /opt/ros/${ROS_DISTRO}/setup.sh && \
    colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release

# Set environment variables
ENV ROBOT_IP="" \
    CONN_TYPE="webrtc" \
    WEBRTC_SERVER_HOST="0.0.0.0" \
    WEBRTC_SERVER_PORT="9991"

# Setup entrypoint
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /root/.bashrc && \
    echo "source /ros2_ws/install/setup.bash" >> /root/.bashrc && \
    sed --in-place --expression \
    '$isource "/ros2_ws/install/setup.bash"' \
    /ros_entrypoint.sh

CMD ["ros2", "launch", "go2_robot_sdk", "robot.launch.py"]
