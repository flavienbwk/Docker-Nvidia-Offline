# Docker-Nvidia-Offline
A documentation explaining how you can install Docker Nvidia on your offline network to benefit the power of your GPU in your containers.

> :warning: The following commands are for Ubuntu 18.04 only. If you download it for another distro, please change the URLs below.

## On the internet computer

```bash
# Get Nvidia Cuda Toolkit (~2 Go)
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
wget http://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb

# Download deb packages for docker-nvidia
curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey > nvidia-docker-gpgkey.txt

wget https://github.com/NVIDIA/nvidia-docker/archive/gh-pages.zip -O nvidia-docker-gh-pages.zip
wget https://github.com/NVIDIA/nvidia-container-runtime/archive/gh-pages.zip -O nvidia-container-runtime-gh-pages.zip
wget https://github.com/NVIDIA/libnvidia-container/archive/gh-pages.zip -O libnvidia-container.zip
unzip nvidia-docker-gh-pages.zip
unzip nvidia-container-runtime-gh-pages.zip
unzip libnvidia-container.zip
```

## On the offline computer

```bash
# Nvidia Cuda Toolkit install
sudo mv cuda-ubuntu1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
sudo dpkg -i cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-2-local-10.2.89-440.33.01/7fa2af80.pub
sudo apt update
sudo apt-get -y install cuda

# Install Nvidia Docker
cat nvidia-docker-gpgkey.txt | sudo apt-key add -

sudo mv nvidia-docker-gh-pages /var/nvidia-docker-gh-pages
sudo mv nvidia-container-runtime-gh-pages /var/nvidia-container-runtime-gh-pages
sudo mv libnvidia-container-gh-pages /var/libnvidia-container-gh-pages

echo "deb file:///var/nvidia-docker-gh-pages/ubuntu18.04/amd64/ /" | sudo tee -a /etc/apt/sources.list.d/docker-nvidia.list
echo "deb file:///var/nvidia-container-runtime-gh-pages/ubuntu18.04/amd64/ /" | sudo tee -a /etc/apt/sources.list.d/docker-nvidia.list
echo "deb file:///var/libnvidia-container-gh-pages/ubuntu18.04/amd64/ /" | sudo tee -a /etc/apt/sources.list.d/docker-nvidia.list
sudo apt update

# Warning : the following command will overwrite 
# your /etc/docker/daemon.json file, back it up !
sudo apt install nvidia-docker2 -y
```

In `/etc/docker/daemon.json`, place :

```
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },
    "default-runtime": "nvidia"
}
```

Now all your containers will have the ability to see your GPU(s). If you use docker-compose, there is **NO NEED** to specify anything inside as the `default-runtime` will be considered for **ANY** of your containers.

## Test !

Run the following to see if your graphic card is seen by your containers :

```
docker run --runtime=nvidia --rm nvidia/cuda nvidia-smi
```

<img src="https://speeload.com/uploads/wDjHwuFeUf.png"/>
