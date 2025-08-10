# 🎮 Complete Guide: Running BlockAssist on WSL (Windows Subsystem for Linux)

## 📋 Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Step-by-Step Installation](#step-by-step-installation)
- [GPU Setup & Optimization](#gpu-setup--optimization)
- [Running BlockAssist](#running-blockassist)
- [Troubleshooting](#troubleshooting)
- [Environment Variables](#environment-variables)
- [Final Commands](#final-commands)

---

## 🚀 Introduction

**BlockAssist** is an AI assistant for Minecraft that learns from user actions. This guide provides a comprehensive setup for running BlockAssist on WSL with GUI support and GPU acceleration.

**Key Features:**
- AI-powered Minecraft assistance
- Machine learning from gameplay
- Web interface for configuration
- GPU-accelerated training
- Cross-platform compatibility

---

### **Required Permissions:**
When creating your Hugging Face token, make sure to check:
- ✅ **Write** - Required for model uploads to leaderboard
- ✅ **Read** - Required for model downloads

### **Required Permissions:**
When creating your Hugging Face token, make sure to check:
- ✅ **Write** - Required for model uploads to leaderboard
- ✅ **Read** - Required for model downloads
- ❌ **Delete** - Not needed (optional)



## ⚠️ Prerequisites

**System Requirements:**
- Windows 10/11 with WSL2 enabled
- At least 8GB RAM (16GB recommended)
- 20GB free disk space
- NVIDIA GPU (optional, for acceleration)
- Stable internet connection

**WSL Setup:**
```bash
# Enable WSL2 (run in PowerShell as Administrator)
wsl --install -d Ubuntu

# Update WSL
wsl --update
```

---

## 🔧 Step-by-Step Installation

### **Step 1: Prepare WSL System**

```bash
# Update package manager
sudo apt update && sudo apt upgrade -y

# Install essential build tools
sudo apt install -y make build-essential gcc g++ \
    libssl-dev zlib1g-dev libbz2-dev libreadline-dev \
    libsqlite3-dev libncursesw5-dev xz-utils tk-dev \
    libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev \
    curl git wget unzip software-properties-common
```

**What these packages do:**
- `build-essential`: Compiler and development tools
- `libssl-dev`: SSL/TLS support for Python
- `libreadline-dev`: Command line editing support
- `libffi-dev`: Foreign function interface for Python

---

### **Step 2: Install Java 8 (Required for Minecraft/Malmo)**

```bash
# Download Azul Zulu JDK 8 (OpenJDK 1.8.0_152)
wget https://cdn.azul.com/zulu/bin/zulu8.25.0.1-jdk8.0.152-linux_x64.tar.gz

# Extract to /opt directory
sudo tar -xzf zulu8.25.0.1-jdk8.0.152-linux_x64.tar.gz -C /opt/

# Set environment variables
export JAVA_HOME=/opt/zulu8.25.0.1-jdk8.0.152-linux_x64
export PATH=$JAVA_HOME/bin:$PATH

# Add to .bashrc for persistence
echo 'export JAVA_HOME=/opt/zulu8.25.0.1-jdk8.0.152-linux_x64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc

# Verify installation
java -version
```

**Why Java 8?**
- Minecraft/Malmo specifically requires Java 8
- Azul Zulu provides stable OpenJDK 8
- Environment variables ensure Java is accessible everywhere

---

### **Step 3: Install Python 3.10 (Required Version)**

```bash
# Install pyenv for Python version management
curl https://pyenv.run | bash

# Setup pyenv in shell
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv init --path)"

# Add to .bashrc
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
echo 'eval "$(pyenv init --path)"' >> ~/.bashrc

# Reload shell or source .bashrc
source ~/.bashrc

# Install Python 3.10
pyenv install 3.10.0
pyenv global 3.10.0

# Verify installation
python --version
pip --version
```

**Why Python 3.10?**
- BlockAssist requires Python >=3.10,<4.0
- pyenv allows multiple Python versions
- Global setting makes Python 3.10 the default

---

### **Step 4: Install GUI Environment (For Minecraft Visual)**

```bash
# Install Xfce4 desktop environment (lightweight)
sudo apt install -y xfce4 xfce4-goodies x11-apps

# Install X11 server and virtual framebuffer
sudo apt install -y x11vnc xvfb x11-utils

# Install display manager
sudo apt install -y lightdm

# Install additional X11 tools
sudo apt install -y mesa-utils glxgears
```

**Why GUI is needed?**
- Minecraft requires a display for rendering
- Xfce4 is a lightweight desktop environment for WSL
- Xvfb provides virtual framebuffer for headless environments
- X11 protocol enables graphical applications

---

### **Step 5: Setup Virtual Environment & Dependencies**

```bash
# Create virtual environment
python -m venv blockassist-venv

# Activate virtual environment
source blockassist-venv/bin/activate

# Upgrade pip
pip install --upgrade pip setuptools wheel

# Install Python dependencies
pip install aiofiles boto3 datasets eth-account hydra-core \
    mbag-gensyn[rllib,malmo] numpy==1.21.6 readchar \
    requests six pydantic web3 python-dotenv psutil
```

**What this does:**
- Virtual environment isolates dependencies from system Python
- Dependencies include all packages needed by BlockAssist
- Version pinning ensures compatibility (numpy==1.21.6)

---

### **Step 6: Install Node.js & Yarn (For Web Interface)**

```bash
# Install nvm (Node Version Manager)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.1/install.sh | bash

# Setup nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

# Add to .bashrc
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"' >> ~/.bashrc
echo '[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"' >> ~/.bashrc

# Reload shell
source ~/.bashrc

# Install Node.js version specified in .nvmrc
nvm install $(cat .nvmrc)
nvm use $(cat .nvmrc)

# Enable Yarn
corepack enable

# Install Node.js dependencies
yarn install --frozen-lockfile
```

**What this does:**
- nvm manages multiple Node.js versions
- Yarn is a faster package manager than npm
- Frozen lockfile installs exact versions

---

### **Step 7: Clone BlockAssist Repository**

```bash
# Clone repository
git clone https://github.com/gensyn-ai/blockassist.git
cd blockassist

# Ensure all dependencies are installed
source blockassist-venv/bin/activate
pip install -e .
```

**What this does:**
- Git clone downloads the latest source code
- pip install -e . installs the project in editable mode

---

## 🚀 GPU Setup & Optimization

### **Check GPU Availability**

```bash
# Check if GPU is available
nvidia-smi

# Check CUDA installation
nvcc --version
python -c "import torch; print(torch.cuda.is_available())"
```

### **Set GPU Environment Variables**

```bash
# Set GPU environment variables
export CUDA_VISIBLE_DEVICES=0
export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128

# Add to .bashrc
echo 'export CUDA_VISIBLE_DEVICES=0' >> ~/.bashrc
echo 'export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128' >> ~/.bashrc
```

**Why set GPU?**
- Faster model training with GPU acceleration
- CUDA_VISIBLE_DEVICES=0 uses the first GPU
- PYTORCH_CUDA_ALLOC_CONF optimizes memory allocation

---

## 🎮 Running BlockAssist

### **Step 8: Setup Display & Launch**

```bash
# Set display environment
export DISPLAY=:0

# Start virtual framebuffer
Xvfb :0 -screen 0 1024x768x24 &

# Start desktop environment (optional, for debugging)
startxfce4 &

# Ensure virtual environment is activated
source blockassist-venv/bin/activate

# Set GPU (if available)
export CUDA_VISIBLE_DEVICES=0

# Run the application
python run.py
```

**What happens:**
- DISPLAY=:0 sets the display for X11 applications
- Xvfb provides virtual framebuffer for headless environments
- Virtual environment ensures dependencies are available
- GPU setup enables acceleration if available

---

## 🔍 Verification & Monitoring

### **Check System Status**

```bash
# Check Java
java -version

# Check Python
python --version

# Check GPU
nvidia-smi

# Check display
echo $DISPLAY

# Check processes
ps aux | grep Xvfb
ps aux | grep java
```

---

## 🛠️ Troubleshooting Common Issues

### **1. Minecraft Crash**

```bash
# Ensure display is set
export DISPLAY=:0
Xvfb :0 -screen 0 1024x768x24 &

# Run with display
DISPLAY=:0 python run.py
```

**Common causes:**
- Display not set properly
- Xvfb not running
- Java version mismatch

### **2. Permission Denied**

```bash
# Fix permissions
sudo chown -R $USER:$USER ~/blockassist
chmod +x setup.sh
```

### **3. Python Not Found**

```bash
# Ensure pyenv is in PATH
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
pyenv global 3.10.0
```

### **4. GPU Not Working**

```bash
# Check CUDA installation
nvcc --version
python -c "import torch; print(torch.cuda.is_available())"

# Verify GPU drivers
nvidia-smi
```

### **5. Display Issues**

```bash
# Check X11 processes
ps aux | grep Xvfb

# Restart Xvfb
pkill Xvfb
Xvfb :0 -screen 0 1024x768x24 &

# Check display variable
echo $DISPLAY
```

---

## 🌍 Environment Variables Complete

```bash
# Add all to .bashrc for persistence
echo 'export JAVA_HOME=/opt/zulu8.25.0.1-jdk8.0.152-linux_x64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
echo 'export PATH="$HOME/.pyenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
echo 'export CUDA_VISIBLE_DEVICES=0' >> ~/.bashrc
echo 'export PYTORCH_CUDA_ALLOC_CONF=max_split_size_mb:128' >> ~/.bashrc
echo 'export DISPLAY=:0' >> ~/.bashrc
```

---

## 🎯 Final Commands to Run

```bash
# Reload environment
source ~/.bashrc

# Setup display
Xvfb :0 -screen 0 1024x768x24 &

# Activate environment and run
source blockassist-venv/bin/activate
export CUDA_VISIBLE_DEVICES=0
python run.py
```

---

## 📚 Additional Resources

**Official Documentation:**
- [BlockAssist GitHub](https://github.com/gensyn-ai/blockassist)
- [WSL Documentation](https://docs.microsoft.com/en-us/windows/wsl/)
- [Minecraft Malmo](https://github.com/microsoft/malmo)

**Community Support:**
- [BlockAssist Discord](https://discord.gg/blockassist)
- [WSL Community](https://github.com/microsoft/WSL)

---

## 🎉 Success Checklist

- ✅ WSL2 installed and updated
- ✅ System dependencies installed
- ✅ Java 8 (OpenJDK 1.8.0_152) installed
- ✅ Python 3.10 with pyenv installed
- ✅ GUI environment (Xfce4 + Xvfb) installed
- ✅ Virtual environment created and activated
- ✅ Node.js and Yarn installed
- ✅ BlockAssist repository cloned
- ✅ GPU environment variables set (if applicable)
- ✅ Display environment configured
- ✅ BlockAssist running successfully

---

## 🚨 Important Notes

**Performance Tips:**
- Use GPU acceleration when available
- Ensure sufficient RAM (16GB+ recommended)
- Close unnecessary applications in WSL

**Security Considerations:**
- Keep WSL updated
- Use virtual environment for Python packages
- Don't run as root unless necessary

**Maintenance:**
- Regularly update packages: `sudo apt update && sudo apt upgrade`
- Clean up old packages: `sudo apt autoremove`
- Monitor disk space: `df -h`

---

## 🤝 Contributing

If you encounter issues or have improvements:
1. Check existing issues on GitHub
2. Create detailed bug reports
3. Submit pull requests with fixes
4. Share your experience with the community

---

**🎮 Happy Gaming with BlockAssist! 🚀**

*This guide is maintained by the BlockAssist community. Last updated: August 2024* 