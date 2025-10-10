# Installing cuda on machine where the nvidia driver for CUDA 13.0 is already available
```
# Creating a temporary directory
mkdir -p temp
cd temp

# Installing CUDA 13.0. Install other versions of CUDA depending on your usecase
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2404/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-13-0

# Adding nvcc to path
echo 'export PATH=/usr/local/cuda/bin:$PATH' | sudo tee -a ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH' >> sudo tee -a ~/.bashrc
source ~/.bashrc


# Testing installation
nvcc --version


# Docker related configurations
sudo apt update
sudo apt install docker.io

sudo apt-get install ca-certificates curl vim
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update

sudo apt install docker-buildx-plugin
sudo chmod 777 /var/run/docker.sock

# Nvidia docker related configurations
curl -fsSL https://nvidia.github.io/libnvidia-container/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/nvidia-container-toolkit-keyring.gpg \
  && curl -s -L https://nvidia.github.io/libnvidia-container/stable/deb/nvidia-container-toolkit.list | \
    sed 's#deb https://#deb [signed-by=/usr/share/keyrings/nvidia-container-toolkit-keyring.gpg] https://#g' | \
    sudo tee /etc/apt/sources.list.d/nvidia-container-toolkit.list

Uncomment this for experimental
# sudo sed -i -e '/experimental/ s/^#//g' /etc/apt/sources.list.d/nvidia-container-toolkit.list
sudo apt-get update

export NVIDIA_CONTAINER_TOOLKIT_VERSION=1.17.8-1
sudo apt-get install -y \
    nvidia-container-toolkit=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    nvidia-container-toolkit-base=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container-tools=${NVIDIA_CONTAINER_TOOLKIT_VERSION} \
    libnvidia-container1=${NVIDIA_CONTAINER_TOOLKIT_VERSION}


sudo nvidia-ctk runtime configure --runtime=docker
sudo systemctl restart docker

# Creating docker image

git clone https://github.com/ancestor-mithril/pytorch-cuda-dockerfiles.git
cd pytorch-cuda-dockerfiles/build_cuda12.9
docker buildx build --network=host -t cuda12.9:py3.13_torch2.8.0 .

# Creating docker container
# Select the project directory (e.g. ArtCADe)
docker run --name pytorch_container --gpus all -d -it --ipc=host --network=host --user root -e GRANT_SUDO=yes -v ~/ArtCADe:/home/jovyan/root/data cuda12.9:py3.13_torch2.8.0
```
