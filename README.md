# Installation Guide - NVIDIA CUDA & Conda Environment

> **Purpose**: Complete setup guide for installing NVIDIA CUDA drivers, Conda environment, and SSH tunneling for a multi-GPU machine learning server.

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [NVIDIA Driver Installation](#nvidia-driver-installation)
3. [Conda Environment Setup](#conda-environment-setup)
4. [Environment Variables Configuration](#environment-variables-configuration)
5. [SSH Tunnel Configuration](#ssh-tunnel-configuration)

---

## Prerequisites

- Ubuntu-based Linux system
- NVIDIA GPU(s) installed
- Sudo privileges
- Internet connection

---

## NVIDIA Driver Installation

### Step 1: Clean Previous Installations

Remove any existing NVIDIA drivers to avoid conflicts:

```bash
sudo apt-get remove --purge '^nvidia-.*'
```

### Step 2: Install Build Dependencies

Required for compiling kernel modules:

```bash
sudo apt-get install ubuntu-desktop
sudo apt install linux-headers-$(uname -r) build-essential dkms
```

> **Note**: For troubleshooting NVIDIA driver issues, see: https://www.reddit.com/r/comfyui/comments/1li8cly/troubleshooting_nvidia_driver_installation_for_a/

### Step 3: Update System Packages

```bash
sudo apt update
sudo apt upgrade
```

### Step 4: Install NVIDIA Drivers

**Option A: Automatic Installation (Recommended)**

```bash
# Install the Ubuntu drivers common package
sudo apt install ubuntu-drivers-common

# List available drivers for your device
sudo ubuntu-drivers devices

# Install the recommended driver (e.g., version 580 for CUDA >= 13.0)
sudo apt install nvidia-driver-580
```

**Option B: Manual Download from NVIDIA**

Download drivers and CUDA toolkit manually from the official NVIDIA website:

- **CUDA Toolkit (all versions)**: https://developer.nvidia.com/cuda-toolkit-archive
- **CUDA Toolkit (latest)**: https://developer.nvidia.com/cuda-downloads

> **Example**: CUDA 13.0.0 for Ubuntu 24.04 x86_64 (runfile local):  
> https://developer.nvidia.com/cuda-13-0-0-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=24.04&target_type=runfile_local

> **Note**: When downloading CUDA manually, choose the **runfile (local)** installer for more control over the installation location.

**Installation Commands:**

```bash
# Download the CUDA runfile installer
wget https://developer.download.nvidia.com/compute/cuda/13.0.0/local_installers/cuda_13.0.0_580.65.06_linux.run

# Run the installer
sudo sh cuda_13.0.0_580.65.06_linux.run
```

> **Tip**: During the runfile installation, you can choose which components to install (Driver, CUDA Toolkit, samples, etc.). Uncheck the driver if you already installed it separately.

> **Tip**: Verify installation with `nvidia-smi` after reboot. Check CUDA version with `nvcc --version`.

---

## Conda Environment Setup

### Step 1: Download and Install Anaconda

```bash
# Download Anaconda installer to home directory
curl -O https://repo.anaconda.com/archive/Anaconda3-2025.12-2-Linux-x86_64.sh

# Run the installer with sudo for system-wide installation
sudo bash ~/Anaconda3-2025.12-2-Linux-x86_64.sh
```

During the interactive installation prompt:
- **License agreement**: Type `yes` and press Enter
- **Installation path**: When prompted, enter `/opt/anaconda3` (default is `~/anaconda3`)
- **Init conda**: Answer `yes` when asked to prepend the path to `.bashrc`

### Step 2: Configure PATH

Add Conda to your PATH:

```bash
export PATH=/opt/anaconda3/bin:$PATH
```

### Step 3: Create and Activate Environment

```bash
# Create environment with Python 3.12 and scientific packages
conda create --name myenv python=3.12 numpy scipy matplotlib scikit-learn

# Activate the environment
conda activate myenv
```

### Step 4: Install PyTorch and ML Libraries

```bash
# Core deep learning frameworks
pip install torch torchvision

# Hugging Face ecosystem
pip install transformers datasets accelerate peft sentence-transformers

# Optimization libraries
pip install deepspeed liger-kernel

# Flash Attention (requires CUDA, no build isolation)
pip install flash-attn --no-build-isolation

# Additional optimization
pip install unsloth
```

---

## Environment Variables Configuration

### Step 1: Configure Default Shell Environment

Add environment variables to `/etc/skel/.bashrc` for new users:

```bash
sudo nano /etc/skel/.bashrc
```

Add the following lines:

```bash
# CUDA Configuration
export CUDA_VISIBLE_DEVICES=0,1                    # Enable GPUs 0 and 1
export PATH="/usr/local/cuda-13.0/bin/:$PATH"      # CUDA binaries
export LD_LIBRARY_PATH="/usr/local/cuda-13.0/lib64/:$LD_LIBRARY_PATH"  # CUDA libraries

# Conda Configuration
export PATH=/opt/anaconda3/bin:$PATH
```

### Step 2: Deploy to Existing Users

Copy the configured `.bashrc` to all existing user directories:

```bash
sudo bash -c 'for user_dir in /home/*; do
    if [ -d "$user_dir" ]; then
        user_name=$(basename "$user_dir")
        
        # 1. Backup existing .bashrc (if it exists)
        if [ -f "$user_dir/.bashrc" ]; then
            cp "$user_dir/.bashrc" "$user_dir/.bashrc.bak"
        fi
        
        # 2. Copy the new .bashrc from /etc/skel
        cp /etc/skel/.bashrc "$user_dir/.bashrc"
        
        # 3. Restore ownership to the user
        chown "$user_name:$user_name" "$user_dir/.bashrc"
        
        echo "Success for $user_name"
    fi
done'
```

> **Warning**: This will overwrite existing `.bashrc` files. Backups are created automatically (`.bashrc.bak`).

---

## SSH Tunnel Configuration

### Overview

This section configures a persistent SSH tunnel for accessing remote services (e.g., LiteLLM on port 4000).

### Step 1: Generate SSH Key (if not already done)

```bash
# Generate a new SSH key pair
ssh-keygen -t rsa

# Copy the public key to the remote server
ssh-copy-id utilisateur@IP_DU_SERVEUR_A
```

### Step 2: Test Manual Tunnel

Establish a temporary tunnel to verify connectivity:

```bash
# -f: Run in background
# -N: No remote command execution
# -L: Local port forwarding (local:remote)
ssh -f -N -L 4000:localhost:4000 utilisateur@IP_DU_SERVEUR_A
```

### Step 3: Create Systemd Service for Auto-Start

Create a systemd service file:

```bash
sudo nano /etc/systemd/system/litellm-tunnel.service
```

Add the following configuration:

```ini
[Unit]
Description=Tunnel SSH permanent pour LiteLLM (Port 4000)
After=network-online.target

[Service]
# Replace with your actual username
User=ton_nom_utilisateur

# SSH tunnel command:
# -N: No remote command
# -T: Disable pseudo-terminal allocation
# -o ServerAliveInterval=60: Keep connection alive (heartbeat every 60s)
# -o ExitOnForwardFailure=yes: Fail if port is already in use
# -L: Local port forwarding
ExecStart=/usr/bin/ssh -NT -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -L 4000:localhost:4000 utilisateur@IP_DU_SERVEUR_A

# Automatically restart if the tunnel drops
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### Step 4: Enable and Start the Service

```bash
# Reload systemd to recognize the new service
sudo systemctl daemon-reload

# Enable auto-start on boot
sudo systemctl enable litellm-tunnel

# Start the service immediately
sudo systemctl start litellm-tunnel

# Verify the service status
sudo systemctl status litellm-tunnel
```

---

## Verification

After completing all steps, verify the setup:

```bash
# Check NVIDIA driver
nvidia-smi

# Check CUDA availability in Python
python -c "import torch; print(f'CUDA available: {torch.cuda.is_available()}')"

# Check SSH tunnel
curl http://localhost:4000
```

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `nvidia-smi` not found | Ensure driver is installed and system was rebooted |
| CUDA not available in PyTorch | Reinstall `torch` with correct CUDA version |
| SSH tunnel fails | Check network connectivity and SSH key permissions |
| Port 4000 already in use | Kill existing process or change port in tunnel config |
