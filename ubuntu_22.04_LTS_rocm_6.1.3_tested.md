# Testing ROCm 6.1.3 with Incus Container

An incus container is set up, on a machine running Ubuntu 22.04 LTS, for testing ROCm 6.1.3.

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

# Install Basic Tools

Run in CONTAINER:

```
sudo apt install -y pulseaudio-utils dbus-user-session dbus-x11 ibus software-properties-common dirmngr apt-transport-https lsb-release ca-certificates apt-utils build-essential make cmake tree wget curl git zip unzip xz-utils nano micro
sudo apt install "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo usermod -a -G render,video $LOGNAME
sudo reboot
```

# Install ROCm 6.1.3

Run in CONTAINER:

```
sudo apt update
wget https://repo.radeon.com/amdgpu-install/6.1.3/ubuntu/jammy/amdgpu-install_6.1.60103-1_all.deb
sudo apt install ./amdgpu-install_6.1.60103-1_all.deb
sudo amdgpu-install --usecase=graphics,multimedia,multimediasdk,rocm,rocmdev,rocmdevtools,lrt,opencl,openclsdk,hip,hiplibsdk,openmpsdk,mllib,mlsdk --no-dkms -y
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

# Install MIGraphX

Run in CONTAINER:

```
sudo apt install -y migraphx
```

# Set up Python

```
sudo apt update
sudo apt install -y make build-essential python3 python-setuptools libjpeg-dev python3-pip python3-dev python3-venv libssl-dev libffi-dev libnss3 zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev python3-wheel python3-wheel-whl twine
```

# Set up a python virtual environment

```
python3 -m venv ai
source ai/bin/activate
pip3 install --upgrade pip wheel
```

# Install Compatible Versions of numpy and protobuf

```
pip install numpy==1.26.4 protobuf==4.25.3
```

# Install PyTorch

```
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.1.3/torch-2.1.2%2Brocm6.1.3-cp310-cp310-linux_x86_64.whl
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.1.3/torchvision-0.16.1%2Brocm6.1.3-cp310-cp310-linux_x86_64.whl
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.1.3/pytorch_triton_rocm-2.1.0%2Brocm6.1.3.4d510c3a44-cp310-cp310-linux_x86_64.whl
pip3 install torch-2.1.2+rocm6.1.3-cp310-cp310-linux_x86_64.whl torchvision-0.16.1+rocm6.1.3-cp310-cp310-linux_x86_64.whl pytorch_triton_rocm-2.1.0+rocm6.1.3.4d510c3a44-cp310-cp310-linux_x86_64.whl
```

To verify:

```
python3 -c 'import torch' 2> /dev/null && echo 'Success' || echo 'Failure'
python3 -c 'import torch; print(torch.cuda.is_available())'
python3 -c "import torch; print(f'device name [0]:', torch.cuda.get_device_name(0))"
python3 -m torch.utils.collect_env
```

# Install ONNX Runtime

Install `migraphx` FIRST!

```
wget https://repo.radeon.com/rocm/manylinux/rocm-rel-6.1.3/onnxruntime_rocm-inference-1.17.0-cp310-cp310-linux_x86_64.whl
mv onnxruntime_rocm-inference-1.17.0-cp310-cp310-linux_x86_64.whl onnxruntime_rocm-1.17.0-cp310-cp310-linux_x86_64.whl
pip install onnxruntime_rocm-1.17.0-cp310-cp310-linux_x86_64.whl
```

Remarks: The 2nd line above renames the wheel file before installation, to avoid errors resulted from running [official instructions](https://rocm.docs.amd.com/projects/radeon/en/latest/docs/install/native_linux/install-onnx.html#).

To verify:

```
python3 -c "import onnxruntime; print(onnxruntime.get_available_providers())"
```

# Install Tensorflow

```
pip3 install https://repo.radeon.com/rocm/manylinux/rocm-rel-6.1.3/tensorflow_rocm-2.15.1-cp310-cp310-manylinux_2_28_x86_64.whl
```

To verify:

```
python3 -c 'import tensorflow' 2> /dev/null && echo 'Success' || echo 'Failure'
```

# References

https://rocm.docs.amd.com/projects/radeon/en/latest/index.html