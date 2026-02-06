# python 3.13, cuda 12.9, torch == 2.8.0
## Usage
Build image:
```
git clone https://github.com/ancestor-mithril/pytorch-cuda-dockerfiles.git
cd pytorch-cuda-dockerfiles/build_cuda12.9
docker buildx build --network=host -t cuda12.9:py3.13_torch2.8.0 .
```
Create container from image:
```
docker run --name pytorch_container --gpus all -d -it --ipc=host --network=host --user root -e GRANT_SUDO=yes -v ~/ArtCADe:/home/jovyan/root/ cuda12.9:py3.13_torch2.8.0
```
