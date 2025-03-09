FROM ros:humble-ros-base AS builder

# Install build dependencies with cache mount
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    curl \
    gnupg2 \
    lsb-release \
    python3-pip \
    portaudio19-dev \
    software-properties-common

# Create workspace and copy only necessary files for dependency installation
WORKDIR /ros2_ws
COPY ./src/requirements.txt /ros2_ws/src/requirements.txt

# Install Python requirements with pip cache
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install 'numpy<2.0.0' && \
    pip install -r /ros2_ws/src/requirements.txt

# Copy source code
COPY ./src /ros2_ws/src

# Install ROS dependencies with cache
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    . /opt/ros/${ROS_DISTRO}/setup.sh && \
    apt-get update && \
    rosdep install --from-paths src --ignore-src -r -y

# Build the workspace
RUN . /opt/ros/${ROS_DISTRO}/setup.sh && \
    colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release

# Runtime stage
FROM ros:humble-ros-base

# Install runtime dependencies only
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    python3-pip \
    portaudio19-dev \
    mesa-utils \
    libgl1-mesa-glx \
    libgl1-mesa-dri \
    ros-humble-rviz2 \
    ros-humble-foxglove-bridge \
    ros-humble-slam-toolbox \
    ros-humble-nav2-bringup \
    ros-humble-pointcloud-to-laserscan \
    ros-humble-joy \
    ros-humble-teleop-twist-joy \
    ros-humble-twist-mux \
    && rm -rf /var/lib/apt/lists/*

# Install runtime Python dependencies
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install 'numpy<2.0.0'

# Copy built workspace from builder stage
COPY --from=builder /ros2_ws/install /ros2_ws/install

# Copy source for any interpreted files or scripts that might be needed
COPY . /ros2_ws/src

# Set environment variables
ENV ROBOT_IP=""
ENV CONN_TYPE="webrtc"
ENV WEBRTC_SERVER_HOST="0.0.0.0"
ENV WEBRTC_SERVER_PORT="9991"

# Setup entrypoint
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> /root/.bashrc && \
    echo "source /ros2_ws/install/setup.bash" >> /root/.bashrc && \
    sed --in-place --expression \
    '$isource "/ros2_ws/install/setup.bash"' \
    /ros_entrypoint.sh

CMD ["ros2", "launch", "go2_robot_sdk", "robot.launch.py"]
