# nv-kepler-tools
ML Tools for legacy Nvidia Kepler Card some with working docker container.

## Supported Cards
Currently using is <b>GT 720</b>.

Latest NVIDIA Driver: <b>470.256.02</b>

Latest CUDA Version: <b>11.4</b>

Latest CUDA Compute Compatibility: <b>3.5</b>

![image](https://github.com/user-attachments/assets/5fb3f717-a0d2-4ce9-885b-1814bdc4b536)

Additional supported cards can refer to the link below. It should work the same as long as have the same CUDA Compatibility version.

https://en.wikipedia.org/wiki/Kepler_(microarchitecture)

https://nvidia.custhelp.com/app/answers/detail/a_id/5204/~/list-of-kepler-series-geforce-desktop-gpus

## Tested ML Tools
<b>Ollama</b> ->  Currently not working due to incompatibility CUDA Compute version, but progress can be track here 
https://github.com/ollama/ollama/issues/1756

<b>llama.cpp</b> -> Working with force Environment Variable to use CUDA Compute 3.5, and CUDA 11.4. 

Working under docker container https://hub.docker.com/r/liej6799/llama-kepler can supply your gguf model

```
docker run -it -v /root/models:/models -p 8080:8080 liej6799/llama-kepler --host 0.0.0.0 --port 8080 -m /models/qwen2.5-0.5b-instruct-fp16.gguf
```
Change the volume and models to use.

<b>PyTorch (Torch and TorchVision)</b> 

Special thanks to Nelson: https://blog.nelsonliu.me/2020/10/13/newer-pytorch-binaries-for-older-gpus/ for building newer torch version with older compatibility

Check your CUDA Version by running `nvidia-smi`

How to read this name?

torch-1.12.1+cu113-cp310-cp310-linux_x86_64.whl

`torch-1.12.1` is the pytorch version
`cu113` is the CUDA Version (11.3) which is the latest for me (11.4)
`cp310` is the python version (Need to match when installing with pip)

Here is the sample running the official mnist training example https://github.com/pytorch/examples/blob/main/mnist/main.py

![training with kepler](https://github.com/user-attachments/assets/4262ef98-35a8-4a67-b320-e8d4122cac87)

Due to older pytorch version, might need to disable mps when running the official example.

` python3 main.py --no-mps `

Final pip Requirement Looks like
```
certifi==2025.1.31                                                                      
charset-normalizer==3.4.1                                                               
idna==3.10                                                                              
numpy==1.26.4                                                                           
pillow==11.1.0                                                                          
requests==2.32.3                                                                        
torch @ file:///torch-1.11.0%2Bcu113-cp310-cp310-linux_x86_64.whl                       
torchvision==0.12.0                                                                     
typing_extensions==4.12.2                                                               
urllib3==2.3.0

```
To Note:

numpy version need to be installed before 2.0.0. torchvision need to match with the torch version


<b>onnxruntime</b> 

Able to find for CUDA 11.4 version for onnxruntime and install, but it needs CUDA Toolkit installed and missing CUDA library file. Currently still not working, possible to work if there is a container with that CUDA Toolkit 11.4.

https://onnxruntime.ai/docs/execution-providers/CUDA-ExecutionProvider.html


## BONUS : GPU Sharing for multiple Docker/Kubernetes Container

As this GPU is still supported by the NVIDIA Container Toolkit, it is possible to share GPU either in Docker / Kubernetes. 

Currently the official NVIDIA Device Plugins https://github.com/NVIDIA/k8s-device-plugin, does not allow sharing GPU on each node.

Workaround: 

<b>For Docker</b>

It is possible to share GPU Resources with Docker and NVIDIA Container Toolkit 

Can try to pass `--runtime=nvidia --gpus all` on ubuntu container, but dont exit.

`sudo docker run --rm --runtime=nvidia --gpus all ubuntu nvidia-smi`

<b>For Kubernetes</b>

As the system is limited to one gpu per node, we can create mutliple LXC with GPU Access, this way each LXC will be treated as seperate node.

If you use k3s for the kubenetes, use `docker` as the backend.

REF
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/install-guide.html
https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/latest/sample-workload.html



