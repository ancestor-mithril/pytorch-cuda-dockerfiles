# pytorch-cuda-dockerfiles

Adapted from https://github.com/iot-salzburg/gpu-jupyter and specialised the Dockerfile.

## Usage
Build image:
```
git clone https://github.com/ancestor-mithril/pytorch-cuda-dockerfiles.git
cd pytorch-cuda-dockerfiles/build_cuda12.1
docker buildx build --network=host -t cuda12.4:py3.13_torch2.6.0 .
```
Create container from image:
```
docker run --name pytorch_container --gpus all -d -it --ipc=host --network=host --user root -e GRANT_SUDO=yes -v ~/data:/home/jovyan/root/data cuda12.4:py3.13_torch2.6.0
```
