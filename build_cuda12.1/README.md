# python 3.11, cuda 12.1, torch >= 2.2.2
## Usage
Build image:
```
git clone https://github.com/ancestor-mithril/pytorch-cuda-dockerfiles.git
cd pytorch-cuda-dockerfiles/build_cuda12.1
docker buildx build --network=host -t cuda12.1:py3.11_torch2.3.0 .
```
Create container from image:
```
docker run --name pytorch_container --gpus all -d -it --ipc=host --network=host --user root -e GRANT_SUDO=yes -v ~/data:/home/jovyan/root/data cuda12.1:py3.11_torch2.3.0
```
