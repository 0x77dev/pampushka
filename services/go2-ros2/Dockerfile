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

# Create workspace
WORKDIR /ros2_ws

# Install Python requirements with pip cache
COPY ./src/requirements.txt /ros2_ws/src/requirements.txt
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

WORKDIR /ros2_ws

# Copy source first for rosdep
COPY ./src /ros2_ws/src

# Install system dependencies and ROS dependencies in a single layer
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && apt-get install -y --no-install-recommends \
    python3-pip \
    portaudio19-dev \
    mesa-utils \
    libgl1-mesa-glx \
    libgl1-mesa-dri \
    ros-humble-rviz2 \
    && . /opt/ros/${ROS_DISTRO}/setup.sh \
    && rosdep install --from-paths src --ignore-src -r -y

# Install runtime Python dependencies
RUN --mount=type=cache,target=/root/.cache/pip \
    pip install 'numpy<2.0.0'

# Copy built workspace from builder stage
COPY --from=builder /ros2_ws/install /ros2_ws/install

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
