# install-nvidia-cuda

## Drivers
```bash
sudo apt-get remove --purge '^nvidia-.*'
sudo apt-get install ubuntu-desktop

sudo apt update
sudo apt upgrade

sudo apt install ubuntu-drivers-common
sudo ubuntu-drivers devices
sudo apt install nvidia-driver-535
```

## Conda
```bash
curl -O https://repo.anaconda.com/archive/Anaconda3-2024.10-1-Linux-x86_64.sh
sudo bash ~/Anaconda3-2024.10-1-Linux-x86_64.sh
yes
/opt/anaconda3

export PATH=/opt/anaconda3/bin:$PATH
conda create --name transformers python=3.12 numpy scipy matplotlib scikit-learn
```
