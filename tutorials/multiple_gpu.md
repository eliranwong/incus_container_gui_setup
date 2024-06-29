# Tutorial: Setup of Multiple Discrete AMD GPUs in an Incus Container

In [my last tutorial](https://discuss.linuxcontainers.org/t/run-offline-tts-with-amd-gpu-acceleration-in-an-incus-container/20273), I demonstrated the setup of ROCm with integrated GPU. In this tutorial, I am going to demonstrat how to set up multiple discrete GPUs in an incus container.

# Launch an Incus Container

Check your current audio server with following command:

```
pactl info | grep PipeWire
```

If there is an output, run:

```
wget https://github.com/eliranwong/incus_container_gui_setup/raw/main/incus_profiles/x11_pipewire_snap_shift
incus profile create x11_pipewire_snap_shift < x11_pipewire_snap_shift
incus launch images:ubuntu/jammy/cloud -p default -p x11_pipewire_snap_shift rocm613
```

If there is no output, run:

```
wget https://github.com/eliranwong/incus_container_gui_setup/raw/main/incus_profiles/x11_pulseaudio_snap_shift
incus profile create x11_pulseaudio_snap_shift < x11_pulseaudio_snap_shift
incus launch images:ubuntu/jammy/cloud -p default -p x11_pulseaudio_snap_shift rocm613
```

# Preparation: Gather Device Information

Check Available GPUs

> incus info --resources | grep "PCI address: " -B 4 -A 6

Output:

```
GPUs:
  Card 0:
    NUMA node: 0
    Vendor: Advanced Micro Devices, Inc. [AMD/ATI] (1002)
    PCI address: 0000:83:00.0
    Driver: amdgpu (6.5.0-41-generic)
    DRM:
      ID: 0
      Card: card0 (226:0)
      Control: controlD64 (226:0)
      Render: renderD128 (226:128)
  Card 1:
    NUMA node: 0
    Vendor: Advanced Micro Devices, Inc. [AMD/ATI] (1002)
    PCI address: 0000:43:00.0
    Driver: amdgpu (6.5.0-41-generic)
    DRM:
      ID: 1
      Card: card1 (226:1)
      Control: controlD65 (226:1)
      Render: renderD129 (226:129)
```

Check Direct Rendering Infrastructure:

> ls /dev/dri

Ouput:

```
by-path  card0  card1  renderD128  renderD129
```

# Add Devices

Add devices according to the information checked above:

> incus config edit ai

```
devices:
  dri_card0:
    gid: "44"
    source: /dev/dri/card0
    type: unix-char
  dri_card1:
    gid: "44"
    source: /dev/dri/card1
    type: unix-char
  dri_renderD128:
    gid: "110"
    source: /dev/dri/renderD128
    type: unix-char
  dri_renderD129:
    gid: "110"
    source: /dev/dri/renderD129
    type: unix-char
  gpu0:
    gid: "44"
    pci: 0000:83:00.0
    type: gpu
  gpu1:
    gid: "44"
    pci: "0000:43:00.0"
    type: gpu
  kfd:
    gid: "110"
    source: /dev/kfd
    type: unix-char
```

Remarks: Don't forget to add the device `/dev/kfd`.

# Restart Container

> incus restart ai

# Log in Container

```
incus exec ai -- sudo --login --user ubuntu
```

# Install Basic Tools

Run in CONTAINER:

```
sudo apt install -y pulseaudio-utils alsa-base alsa-utils pavucontrol dbus-user-session dbus-x11 ibus im-config software-properties-common dirmngr apt-transport-https lsb-release ca-certificates apt-utils build-essential make cmake tree wget curl git zip unzip xz-utils nano micro vlc
sudo apt install -y "linux-headers-$(uname -r)" "linux-modules-extra-$(uname -r)"
sudo usermod -a -G render,video $LOGNAME
sudo reboot
```

# Install ROCm 6.1.3

Login agin and run:

```
sudo apt update
sudo apt install -y libstdc++-12-dev
wget https://repo.radeon.com/amdgpu-install/6.1.3/ubuntu/jammy/amdgpu-install_6.1.60103-1_all.deb
sudo apt install ./amdgpu-install_6.1.60103-1_all.deb
sudo amdgpu-install --usecase=graphics,multimedia,multimediasdk,rocm,rocmdev,rocmdevtools,lrt,opencl,openclsdk,hip,hiplibsdk,openmpsdk,mllib,mlsdk --no-dkms -y
reboot
```

# Check ROCm Information

Run:

> rocminfo

Look for information like:

```
*******                  
Agent 2                  
*******                  
  Name:                    gfx1100                            
  Uuid:                    GPU-b54ca445df90862b               
  Marketing Name:          Radeon RX 7900 XTX                 
  Vendor Name:             AMD 

*******                  
Agent 3                  
*******                  
  Name:                    gfx1100                            
  Uuid:                    GPU-2ff163adb661d5fb               
  Marketing Name:          Radeon RX 7900 XTX                 
  Vendor Name:             AMD        
```

# Set Environment Variables

Append the file `~/.bashrc`:

> nano ~/.bashrc

Add the following content at the end of the file:

```
# ROCM 6.1.3
export GFX_ARCH=gfx1100
export HCC_AMDGPU_TARGET=gfx1100
export CUPY_INSTALL_USE_HIP=1
export ROCM_VERSION=6.1
export ROCM_HOME=/opt/rocm
export LD_LIBRARY_PATH=/opt/rocm/include:/opt/rocm/lib:$LD_LIBRARY_PATH
export PATH=/home/eliran/.local/bin:/opt/rocm/bin:/opt/rocm/llvm/bin:$PATH
export HSA_OVERRIDE_GFX_VERSION=11.0.0
export ROCR_VISIBLE_DEVICES=GPU-b54ca445df90862b,GPU-2ff163adb661d5fb
export GPU_DEVICE_ORDINAL=0,1
export HIP_VISIBLE_DEVICES=0,1
export CUDA_VISIBLE_DEVICES=0,1
export LLAMA_HIPLAS=0,1
export GGML_VULKAN_DEVICE=0,1
export GGML_VK_VISIBLE_DEVICES=0,1
export DRI_PRIME=0
export OMP_DEFAULT_DEVICE=1
```

Remarks: Modify `gfx1100` and the value of `ROCR_VISIBLE_DEVICES`, according to your `rocminfo` output.

# Speed Test: CPU vs CPU+GPUx2

To test, I ran the same prompt `What is machine learning?` with the same model file `mistral.gguf`, using [llama.cpp](https://github.com/ggerganov/llama.cpp).

## CPU only

Build from source:

```
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make -j$(lscpu | grep '^Core(s)' | awk '{print $NF}')
```

To run:

> ./llama-cli -t $(lscpu | grep '^Core(s)' | awk '{print $NF}') --temp 0 -m ../mistral.gguf -p "What is machine learning?"

Output:

```
llm_load_tensors: ggml ctx size =    0.14 MiB
llm_load_tensors:        CPU buffer size =  3917.87 MiB
...
...
...
llama_print_timings:        load time =    1602.50 ms
llama_print_timings:      sample time =      11.38 ms /   571 runs   (    0.02 ms per token, 50193.39 tokens per second)
llama_print_timings: prompt eval time =      81.22 ms /     6 tokens (   13.54 ms per token,    73.88 tokens per second)
llama_print_timings:        eval time =   24270.78 ms /   570 runs   (   42.58 ms per token,    23.49 tokens per second)
llama_print_timings:       total time =   24522.14 ms /   576 tokens
```

## CPU + GPU x 2

Build from source:

```
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp
make GGML_HIPBLAS=1 AMDGPU_TARGETS=gfx1100 -j$(lscpu | grep '^Core(s)' | awk '{print $NF}')
```

Remarks: Use `GGML_HIPBLAS` instead of `LLAMA_HIPBLAS`

To run:

> ./llama-cli -t $(lscpu | grep '^Core(s)' | awk '{print $NF}') --temp 0 -m ../mistral.gguf -p "What is machine learning?" -ngl 33

Output:

```
ggml_cuda_init: found 2 ROCm devices:
  Device 0: Radeon RX 7900 XTX, compute capability 11.0, VMM: no
  Device 1: Radeon RX 7900 XTX, compute capability 11.0, VMM: no
llm_load_tensors: ggml ctx size =    0.41 MiB
llm_load_tensors: offloading 32 repeating layers to GPU
llm_load_tensors: offloading non-repeating layers to GPU
llm_load_tensors: offloaded 33/33 layers to GPU
llm_load_tensors:      ROCm0 buffer size =  1989.53 MiB
llm_load_tensors:      ROCm1 buffer size =  1858.02 MiB
llm_load_tensors:        CPU buffer size =    70.31 MiB
...
...
...
llama_print_timings:        load time =    3440.52 ms
llama_print_timings:      sample time =      17.78 ms /   952 runs   (    0.02 ms per token, 53540.30 tokens per second)
llama_print_timings: prompt eval time =      12.38 ms /     6 tokens (    2.06 ms per token,   484.46 tokens per second)
llama_print_timings:        eval time =    8928.70 ms /   951 runs   (    9.39 ms per token,   106.51 tokens per second)
llama_print_timings:       total time =    9119.94 ms /   957 tokens
```

## Result

The difference is more than obvious.

# Read More: 

More about Environment variables:

https://github.com/eliranwong/MultiAMDGPU_AIDev_Ubuntu/tree/main?tab=readme-ov-file#environment-variables

Install Common Python Packages for AI Development:

Read https://github.com/eliranwong/MultiAMDGPU_AIDev_Ubuntu/tree/main?tab=readme-ov-file#set-up-python