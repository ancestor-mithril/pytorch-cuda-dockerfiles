```
# Installing cuda on virtual machine where the nvidia driver is already available
wget https://developer.download.nvidia.com/compute/cuda/12.2.2/local_installers/cuda_12.2.2_535.104.05_linux.run

# Attention! Don't select "Driver" (was already installed, please don't update it) and "Kernel Objects" (nvidia-fs doesn't work without additional configurations)
sudo sh cuda_12.2.2_535.104.05_linux.run 

# Adding cuda to path
echo 'export PATH=/usr/local/cuda-12.2/bin:$PATH' | sudo tee -a ~/.bashrc
echo 'export LD_LIBRARY_PATH=/usr/local/cuda-12.2/lib64:$LD_LIBRARY_PATH' >> sudo tee -a ~/.bashrc

# Restart machine
sudo reboot

# Testing installation
nvcc --version

# Docker related configurations
sudo apt update
sudo apt install docker.io
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
cd pytorch-cuda-dockerfiles/build_cuda12.1
docker buildx build --network=host -t cuda12.1:py3.11_torch2.2.2 .

# Creating docker container
mkdir -p ~/data
docker run --name pytorch_container --gpus all -d -it --ipc=host -p 8888:8192 --user root -e GRANT_SUDO=yes -v ~/data:/home/jovyan/root/data cuda12.1:py3.11_torch2.2.2
```
