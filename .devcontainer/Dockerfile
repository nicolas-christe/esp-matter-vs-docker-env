FROM ubuntu:22.04

# install required tools
ENV DEBIAN_FRONTEND=noninteractive
RUN : \
  && apt-get update \
  && apt-get install -y \
    git wget flex bison gperf python3 python3-pip python3-venv cmake ninja-build ccache libffi-dev libssl-dev dfu-util libusb-1.0-0 \
    gcc g++ pkg-config libdbus-1-dev libglib2.0-dev libavahi-client-dev  python3-dev unzip libgirepository1.0-dev libcairo2-dev libreadline-dev \
  && apt-get autoremove -y \
  && rm -rf /var/lib/apt/lists/* \
  && update-alternatives --install /usr/bin/python python /usr/bin/python3 10 \
  && :

# ESP-IDF
ENV ISP_VERSION=v5.1.2
ENV IDF_PATH=/opt/esp/idf
ENV IDF_TOOLS_PATH=/opt/esp
ENV ISP_TARGETS=esp32h2

RUN : \
  && git clone --recursive --depth=1 --shallow-submodules --jobs 8 -b $ISP_VERSION https://github.com/espressif/esp-idf.git $IDF_PATH \
  && git config --system --add safe.directory $IDF_PATH \
  && :

RUN : \
  && $IDF_PATH/tools/idf_tools.py --non-interactive install required --targets=${ISP_TARGETS} \
  && $IDF_PATH/tools/idf_tools.py --non-interactive install cmake \
  && $IDF_PATH/tools/idf_tools.py --non-interactive install-python-env \
  && rm -rf $IDF_TOOLS_PATH/dist \
  && :

# Ccache is installed, enable it by default
ENV IDF_CCACHE_ENABLE=1

# The constraint file has been downloaded and the right Python package versions installed. No need to check and
# download this at every invocation of the container.
ENV IDF_PYTHON_CHECK_CONSTRAINTS=no

# ESP MATTER
ENV ESP_MATTER_VERSION=release/v1.2
ENV ESP_MATTER_PATH=/opt/esp/matter

WORKDIR $ESP_MATTER_PATH
RUN : \
  && git clone --depth 1 --jobs 8 https://github.com/espressif/esp-matter.git -b $ESP_MATTER_VERSION . \
  && git submodule update --init --depth 1 \
  && cd ./connectedhomeip/connectedhomeip \
  && ./scripts/checkout_submodules.py --platform esp32 linux --shallow \
  && :

WORKDIR $ESP_MATTER_PATH
RUN : \  
  && . $IDF_PATH/export.sh \
  && ./install.sh \
  && :

# Add main user
ENV CONTAINER_USER=esp
ENV USER_UID=1000
ENV USER_GID=$USER_UID
RUN groupadd --gid $USER_GID $CONTAINER_USER \
    && adduser --uid $USER_UID --gid $USER_GID --disabled-password --gecos "" ${CONTAINER_USER} \
    && usermod -a -G dialout $CONTAINER_USER
USER ${CONTAINER_USER}
ENV USER=${CONTAINER_USER}
WORKDIR /home/${CONTAINER_USER}

# don't care about safe directory in a encapsulateds docker env
RUN git config --global --add safe.directory *

RUN echo "source $IDF_PATH/export.sh > /dev/null 2>&1" >> ~/.bashrc
RUN echo "source $ESP_MATTER_PATH/export.sh > /dev/null 2>&1" >> ~/.bashrc

CMD ["/bin/bash", "-c"]