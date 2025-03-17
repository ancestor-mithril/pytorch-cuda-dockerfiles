```
# Installing cuda on virtual machine where the nvidia driver is already available
wget https://developer.download.nvidia.com/compute/cuda/12.4.0/local_installers/cuda_12.4.0_550.54.14_linux.run

# Attention! Don't select "Driver" (was already installed, please don't update it) and "Kernel Objects" (nvidia-fs doesn't work without additional configurations)
sudo sh cuda_12.4.0_550.54.14_linux.run

# Adding cuda to path
echo 'export PATH=/usr/local/cuda-12.4/bin:$PATH' | sudo tee -a ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.4/lib64:$LD_LIBRARY_PATH' >> sudo tee -a ~/.bashrc

# Restart machine
sudo reboot

# Testing installation
nvcc --version

# Docker related configurations
sudo apt update
sudo apt install docker.io

sudo apt-get install ca-certificates curl
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
# ubuntu2204 on an x86_64 architecture
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt install -y nvidia-docker2
sudo systemctl daemon-reload
sudo systemctl restart docker

# Cudnn
# We could install cudnn8 instead
sudo apt-get -y install cudnn9-cuda-12

# Creating docker image

git clone https://github.com/ancestor-mithril/pytorch-cuda-dockerfiles.git
cd pytorch-cuda-dockerfiles/build_cuda12.4_mini
docker buildx build --network=host -t cuda12.4:py3.12_torch2.4.2 .

# Creating docker container
mkdir -p ~/data
docker run --name pytorch_container --gpus all -d -it --ipc=host --network=host --user root -e GRANT_SUDO=yes -v ~/data:/home/jovyan/root/data cuda12.4:py3.13_torch2.6.0
```
