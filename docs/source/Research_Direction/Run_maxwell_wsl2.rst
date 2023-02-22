wsl2安装CUDA+Docker
================================

安装win10最新版本或者win11更新驱动至最新
----------------------------------------------------------------

安装wsl2
----------------------------------------------------------------

安装CUDA
----------------------------------------------------------------

::

    wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-wsl-ubuntu.pin
    sudo mv cuda-wsl-ubuntu.pin /etc/apt/preferences.d/cuda-repository-pin-600
    wget https://developer.download.nvidia.com/compute/cuda/11.4.2/local_installers/cuda-repo-wsl-ubuntu-11-4-local_11.4.2-1_amd64.deb
    sudo dpkg -i cuda-repo-wsl-ubuntu-11-4-local_11.4.2-1_amd64.deb
    sudo apt-key add /var/cuda-repo-wsl-ubuntu-11-4-local/7fa2af80.pub
    sudo apt-get update
    sudo apt-get -y install cuda

安装CUDA
----------------------------------------------------------------

::

    export PATH=$PATH:/usr/lib/wsl/lib
    distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
    curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
    curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
    curl -s -L https://nvidia.github.io/libnvidia-container/experimental/$distribution/libnvidia-container-experimental.list | sudo tee /etc/apt/sources.list.d/libnvidia-container-experimental.list
    sudo apt-get update


