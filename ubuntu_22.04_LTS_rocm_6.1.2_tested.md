# Testing ROCm 6.1.2 with Incus Container

An incus container is set up, on a machine running Ubuntu 22.04 LTS, for testing ROCm 6.1.2.

Hardware: iGPU only; no discrete GPU

Tested device: Beelink GTR6 (Ryzen 9 6900HX CPU + integrated Radeon 680M GPU + 64GB RAM)

# Run in Host

Run in terminal:

```
incus launch images:ubuntu/jammy/cloud -p default -p x11_pipewire_snap_shift rocmlatest
incus config device add rocmlatest home disk source=/home/${USER} path=/home/ubuntu/eliran shift=true
incus config edit rocmlatest
```

Add the following content under section `devices`:

```
devices:
  camera:
    busnum: "3"
    devnum: "2"
    gid: "1000"
    productid: "0843"
    type: usb
    uid: "1000"
    vendorid: 046d
  controlC2:
    source: /dev/snd/controlC2
    type: unix-char
  dri_card0:
    gid: "44"
    source: /dev/dri/card0
    type: unix-char
  dri_renderD128:
    gid: "110"
    source: /dev/dri/renderD128
    type: unix-char
  gpu:
    gid: "44"
    pci: "0000:05:00.0"
    type: gpu
  kfd:
    gid: "110"
    source: /dev/kfd
    type: unix-char
  video0:
    gid: "44"
    source: /dev/video0
    type: unix-char
  video1:
    gid: "44"
    source: /dev/video1
    type: unix-char
```

Run in terminal:

```
incus restart rocmlatest
alias rocmlatest="incus exec rocmlatest -- sudo --login --user ubuntu"
rocmlatest
```

# Run

Run in terminal:

```
sudo apt install -y pulseaudio-utils dbus-user-session dbus-x11 ibus software-properties-common dirmngr apt-transport-https lsb-release ca-certificates apt-utils build-essential make cmake tree wget curl git zip unzip xz-utils nano micro
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo usermod -a -G render,video $LOGNAME
sudo reboot
```

Run in terminal after reboot:

```
sudo apt update
wget https://repo.radeon.com/amdgpu-install/6.1.2/ubuntu/jammy/amdgpu-install_6.1.60102-1_all.deb
sudo apt install ./amdgpu-install_6.1.60102-1_all.deb
sudo amdgpu-install --usecase=graphics,opencl,openclsdk,hip,hiplibsdk,rocm,rocmdev,rocmdevtools,lrt,mllib,mlsdk,openmpsdk,multimedia,multimediasdk --no-dkms -y
nano ~/.bashrc
```

Add the following content at the end of `~/.bashrc`:

```
export GFX_ARCH=gfx1030
export ROCM_VERSION=6.1
export ROCM_HOME=/opt/rocm
export LD_LIBRARY_PATH=/opt/rocm/include:/opt/rocm/lib:$LD_LIBRARY_PATH
export PATH=$HOME/.local/bin:/opt/amdgpu/bin:/opt/rocm/bin:/opt/rocm/llvm/bin:/opt/rocm/libexec/amdsmi_cli:$PATH
export HSA_OVERRIDE_GFX_VERSION=10.3.0
```

Reboot

```
sudo reboot
```

# Test with Llama.cpp

```
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make LLAMA_HIPBLAS=1 LLAMA_HIP_UMA=1 AMDGPU_TARGETS=gfx1030 -j$(lscpu | grep '^Core(s)' | awk '{print $NF}')
./llama-cli -t $(lscpu | grep '^Core(s)' | awk '{print $NF}') --temp 0 -p "What is machine learning?" -ngl 33 -m ../eliran/freegenius/LLMs/gguf/mistral.gguf
```

```
llama_print_timings:        load time =    4449.41 ms
llama_print_timings:      sample time =      11.86 ms /   343 runs   (    0.03 ms per token, 28918.30 tokens per second)
llama_print_timings: prompt eval time =     130.74 ms /     6 tokens (   21.79 ms per token,    45.89 tokens per second)
llama_print_timings:        eval time =   23766.91 ms /   342 runs   (   69.49 ms per token,    14.39 tokens per second)
llama_print_timings:       total time =   24146.75 ms /   348 tokens
```

Faster than ROCm 6.0.2: Running ROCM 6.1.2 inside an incus container is faster than both running ROCm 6.0.2 inside an incus container and running ROCm 6.0.2 directly on the host.

Read https://github.com/eliranwong/MultiAMDGPU_AIDev_Ubuntu/blob/main/igpu_only.md#further-comparison-with-running-inside-an-incus-container

# Set up Python

```
sudo apt update
sudo apt install -y make build-essential python3 python-setuptools libjpeg-dev python3-pip python3-dev python3-venv libssl-dev libffi-dev libnss3 zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev python3-wheel python3-wheel-whl twine
```

# Install latest PyTorch

```
python3 -m venv pytorchnightly
source pytorchnightly/bin/activate
pip3 install --pre torch torchvision torchaudio --index-url https://download.pytorch.org/whl/nightly/rocm6.1/
```
