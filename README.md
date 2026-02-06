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
pip install torch torchvision
pip install transformers datasets accelerate peft sentence-transformers
pip install deepspeed liger-kernel
pip install flash-attn --no-build-isolation
pip install unsloth
```

## Skel
```
sudo nano /etc/skel/.bashrc
export CUDA_VISIBLE_DEVICES=0,1
export PATH="/usr/local/cuda-12.1/bin/:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-12.1/lib64/:$LD_LIBRARY_PATH"
export PATH=/opt/anaconda3/bin:$PATH
```

## Bashrc
```
sudo bash -c 'for user_dir in /home/*; do
    if [ -d "$user_dir" ]; then
        user_name=$(basename "$user_dir")
        
        # 1. Sauvegarde (si le fichier existe)
        if [ -f "$user_dir/.bashrc" ]; then
            cp "$user_dir/.bashrc" "$user_dir/.bashrc.bak"
        fi
        
        # 2. Copie du nouveau fichier
        cp /etc/skel/.bashrc "$user_dir/.bashrc"
        
        # 3. Réattribution des droits à l utilisateur
        chown "$user_name:$user_name" "$user_dir/.bashrc"
        
        echo "Succès pour $user_name"
    fi
done'
```
