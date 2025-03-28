FROM ros:melodic

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update \
  && apt-get install --no-install-recommends -y software-properties-common gnupg ca-certificates \
  && add-apt-repository ppa:apt-fast/stable \
  && apt-get update \
  && apt-get install --no-install-recommends -y apt-fast \
  && apt-fast clean \
  && rm -rf /var/lib/apt/lists/*

RUN echo debconf apt-fast/maxdownloads string 16 | debconf-set-selections && \
  echo debconf apt-fast/dlflag boolean true | debconf-set-selections && \
  echo debconf apt-fast/aptmanager string apt-get | debconf-set-selections

RUN apt-fast update \
  && apt-fast install -y --no-install-recommends \
  git wget nano libomp-dev build-essential python3-pip

RUN apt-fast install -y --no-install-recommends \
  ros-${ROS_DISTRO}-navigation \
  ros-${ROS_DISTRO}-robot-localization \
  ros-${ROS_DISTRO}-robot-state-publisher \
  ros-${ROS_DISTRO}-jsk-recognition-msgs \
  ros-${ROS_DISTRO}-jsk-rviz-plugins \
  ros-${ROS_DISTRO}-xacro 

WORKDIR /root
RUN git clone -b 4.0.3 https://github.com/borglab/gtsam.git
RUN mkdir /root/gtsam/build
WORKDIR /root/gtsam/build
RUN cmake .. && make -j$(nproc) install

# for melodic, resolve conflicting
RUN mv /usr/include/flann/ext/lz4.h /usr/include/flann/ext/lz4.h.bak
RUN mv /usr/include/flann/ext/lz4hc.h /usr/include/flann/ext/lz4hc.h.bak
RUN ln -s /usr/include/lz4.h /usr/include/flann/ext/lz4.h
RUN ln -s /usr/include/lz4hc.h /usr/include/flann/ext/lz4hc.h


RUN mkdir -p /root/catkin_ws/src/LIO-SEGMOT
WORKDIR /root/catkin_ws/src/LIO-SEGMOT
COPY . /root/catkin_ws/src/LIO-SEGMOT

WORKDIR /root/catkin_ws
RUN /bin/bash -c '. /opt/ros/${ROS_DISTRO}/setup.bash; catkin_make -DCMAKE_BUILD_TYPE=Release'

RUN sed -i "6i source \"/root/catkin_ws/devel/setup.bash\"" /ros_entrypoint.sh

WORKDIR /

ENTRYPOINT ["/ros_entrypoint.sh"]
CMD ["bash"]