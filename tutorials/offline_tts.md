# Tutorial: Run Offline TTS with AMD GPU Acceleration in an Incus Container

To me, running incus container is the best way to isolate testing for the latest graphic card driver or softwares, to minimize the risk of messing up the host.

I have AMD GPU devices. I managed to run offline text-to-speech engine `piper-tts` with discrete GPU acceleration via ROCm 6.0.2. However, with ROCm 6.0.2, I failed to do the same with iGPU.

With the release of the latest ROCm 6.1.3. I ran a test in an incus container before installing it on the host. I am happy that the test is positive and I would like to share with you here.

With this tutorial, we will learn, how to:

1. Set up the latest ROCm 6.1.3
2. Set up ONNX runtime with ROCM and MIGraphX
3. Set up offline [Piper TTS](https://github.com/rhasspy/piper), running with either ROCM or MIGraphX provider.

in an incus container.

Remarks: This tutorial assumes to work with iGPU-only devices.  I may write another tutorial to work multiple discrete GPUs.

# Launch an Audio-enabled Incus Container

We need to launch an audio-enabled incus container for our testing.

Check your current audio server with following command:

```
pactl info | grep PipeWire
```

If there is output, run:

```
wget https://github.com/eliranwong/incus_container_gui_setup/raw/main/incus_profiles/x11_pipewire_snap_onegpu_rocm_shift
incus launch images:ubuntu/jammy/cloud -p default -p x11_pipewire_snap_onegpu_rocm_shift rocm613
```

If there is no output, run:

```
wget https://github.com/eliranwong/incus_container_gui_setup/raw/main/incus_profiles/x11_pulseaudio_snap_onegpu_rocm__shift
incus launch images:ubuntu/jammy/cloud -p default -p x11_pulseaudio_snap_onegpu_rocm__shift rocm613
```

# Log in Container

```
incus exec rocm613 -- sudo --login --user ubuntu
```

REMARKS: Follow the following installation order.

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

(Remarks: Check your gpu gfx version by running `rocminfo | grep gfx` and modify the following content accordingly.)

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
cd ~
python3 -m venv offlinetts
source offlinetts/bin/activate
pip3 install --upgrade pip wheel
```

# Install Compatible Versions of numpy and protobuf

```
pip install numpy==1.26.4 protobuf==4.25.3
```

# Install piper-tts

```
pip install piper-tts
```

# Uninstall onnxruntime

```
pip uninstall onnxruntime
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

# Manually Edit ONNX Provider

At the time of writing, piper-tts is yet to support either `MIGraphXExecutionProvider` or `ROCMExecutionProvider`.  I submitted a PR, though.

Therefore, we need to manually edit the 'load' function in the file //site-packages/piper/voice.py:

Open the file with an editor:

> nano /home/ubuntu/offlinetts//lib/python3.10/site-packages/piper/voice.py

Change from:

```
providers=["CPUExecutionProvider"]
if not use_cuda
else ["CUDAExecutionProvider"],
```

To:

```
providers=["MIGraphXExecutionProvider"],
```

Save (ctrl+o) and exit (ctrl+x) the file.

# To Test

You should be able to hear the statement "Incus container is amazing!" with the following command:

```
echo "Incus container is amazing!" | piper --model en_US-lessac-medium --output-raw | aplay -r 22050 -f S16_LE -t raw -
```

# Final Note

This tutorial not only demonstrate the use of piper-tts with AMD GPU, but also the value of incus container for testing latest drivers or softwares.

# Reference

https://rocm.docs.amd.com/projects/radeon/en/latest/index.html