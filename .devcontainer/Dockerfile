FROM osrf/ros:jazzy-desktop

# 개발 도구
RUN apt-get update && apt-get install -y --no-install-recommends \
      sudo git nano bash-completion \
      ros-dev-tools \
      ros-jazzy-turtlesim \
      python3-pip \
    && rm -rf /var/lib/apt/lists/*

# 일반 사용자(dev, UID 1000) 생성: 마운트한 폴더가 root 소유가 되는 것을 방지
# (Ubuntu 24.04 기본 이미지에는 UID 1000 사용자가 이미 있을 수 있어 이름을 바꿔 사용)
RUN if id -u 1000 >/dev/null 2>&1; then \
      OLD_USER=$(id -un 1000); OLD_GROUP=$(id -gn 1000); \
      usermod -l dev -d /home/dev -m "$OLD_USER" && groupmod -n dev "$OLD_GROUP"; \
    else \
      groupadd --gid 1000 dev && useradd --uid 1000 --gid 1000 -m -s /bin/bash dev; \
    fi \
 && echo "dev ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/dev \
 && chmod 0440 /etc/sudoers.d/dev

USER dev

# 셸 시작 시 ROS 2 환경 자동 로드
RUN echo "source /opt/ros/jazzy/setup.bash" >> /home/dev/.bashrc \
 && echo "[ -f /workspaces/ros2-learning/ws/install/setup.bash ] && source /workspaces/ros2-learning/ws/install/setup.bash" >> /home/dev/.bashrc

WORKDIR /workspaces/ros2-learning