# incus_container_gui_setup

Setup of running GUI applications in a [incus container](https://linuxcontainers.org/incus/docs/main/)

Tested on Ubuntu 22.04 LTS

# Install Incus

> sudo mkdir -p /etc/apt/keyrings

> wget -qO - https://pkgs.zabbly.com/key.asc | sudo tee /etc/apt/keyrings/zabbly.asc

> sudo nano /etc/apt/sources.list.d/zabbly-incus-stable.sources

Add the following content:

```
Enabled: yes
Types: deb
URIs: https://pkgs.zabbly.com/incus/stable
Suites: jammy
Components: main
Architectures: amd64
Signed-By: /etc/apt/keyrings/zabbly.asc
```

> sudo apt update

> sudo apt install -y incus

> sudo adduser $LOGNAME incus-admin

> newgrp incus-admin

> incus admin init

# View available Images

To list all available images:

> incus image list images:

Filter available images, e.g.:

> incus image list images:ubuntu

> incus image list images:22.04

> incus image list images:debian

To filter images that work with ```cloud-init```:

> incus image list images:cloud

# Prepare a profile

Choose one of the GUI profiles below or use your own profile:

[x11_pipewire_shift](https://github.com/eliranwong/incus_container_gui_setup/blob/main/incus_profiles/x11_pipewire_shift)

[x11_pipewire_snap_shift](https://github.com/eliranwong/incus_container_gui_setup/blob/main/incus_profiles/x11_pipewire_snap_shift)

[x11_pulseaudio_shift](https://github.com/eliranwong/incus_container_gui_setup/blob/main/incus_profiles/x11_pulseaudio_shift)

[x11_pulseaudio_snap_shift](https://github.com/eliranwong/incus_container_gui_setup/blob/main/incus_profiles/x11_pulseaudio_snap_shift)

Remarks: Use one of the following commands to check your display settings and replace 'X1' with your display settings:

> printenv | grep -i display

> echo $DISPLAY

> ls /tmp/.X11-unix/

# Create a Profile

To create a new profile:

> incus profile create x11_pipewire_shift < x11_pipewire_shift

To edit an existing profile:

> incus profile edit x11_pipewire_shift < x11_pipewire_shift

Configure X11's access control, to allow non-network local connections:

> xhost +local:

> echo "xhost +local:" >> ~/.profile

# Launch

To launch a container:

> incus launch images:ubuntu/jammy/cloud -p default -p x11_pipewire_shift ub

To launch a container with limits, e.g.:

> incus launch images:ubuntu/jammy/cloud -p default -p x11_pipewire_shift ub -c limits.cpu=4 -c limits.memory=32GiB

To edit size

> incus config device override ub root size=128GiB

# GPU Support

Check available GPUs:

> incus info --resources | grep "PCI address: " -B 4

iGPU ONLY:

```
devices:
  gpu:
    type: gpu
    gid: 44
    pci: "xxxx:xx:xx.x"
```

Multiple GPUs:

```
devices:
  gpu1:
    type: gpu
    gid: 44
    pci: "xxxx:xx:xx.x"
  gpu2:
    type: gpu
    gid: 44
    pci: "xxxx:xx:xx.x"
```

Remarks:

1. Replace "xxxx:xx:xx.x" with your GPU pci addresses.
2. To work with ROCM, add either iGPU or discrete GPUs.

# Direct Rendering Infrastructure

> ls /dev/dri

Output, e.g.:

```
by-path  card0  renderD128
```

> incus config edit ub

Add `card0` and `renderD128` under section `devices`:

```
devices:
  dri_card0:
    gid: "44"
    source: /dev/dri/card0
    type: unix-char
  dri_renderD128:
    gid: "110"
    source: /dev/dri/renderD128
    type: unix-char
```

# Bind Home Folder

> incus exec ub -- mkdir /home/ubuntu/eliran

> incus exec ub -- chown ubuntu:ubuntu /home/ubuntu/eliran

> incus config device add ub home disk source=/home/${USER} path=/home/ubuntu/eliran shift=true

Remarks: It is not good to bind directly to /home/ubuntu/, as it overrides .bashrc or .profile.

# Login

> incus exec ub -- sudo --login --user ubuntu

# Test Sound

> sudo apt install -y espeak

> espeak test

# Test Display

> sudo apt install -y x11-apps

> xclock

# Webcam

Run in HOST:

> incus info --resources | grep Webcam -B 3 -A 3

Output, e.g.:

```
  Device 1:
    Vendor: Logitech, Inc.
    Vendor ID: 046d
    Product: Logitech Webcam C930e
    Product ID: 0843
    Bus Address: 3
    Device Address: 2
```

> ls -l /dev/snd/by-id

Output, e.g.:

```
usb-046d_Logitech_Webcam_C930e_095C816E-02 -> ../controlC2
```

> incus config edit ub

```
devices:
  Webcam:
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
  video0:
    source: /dev/video0
    gid: 44
    type: unix-char
  video1:
    source: /dev/video1
    gid: 44
    type: unix-char
```

# Test Camera

> sudo usermod -a -G render,video $LOGNAME

> sudo apt install build-essential linux-headers-\`uname -r\` guvcview

> sudo reboot

> guvcview

# Test Microphone

Run in CONTAINER:

> sudo apt install -y alsa-base alsa-utils pavucontrol

Run in HOST:

> incus restart ub

Run in CONTAINER:

> pavucontrol

Go to "Input Devices" and say something ...

# Install python and related tools

> sudo apt install -y make build-essential python3 python-setuptools python3-pip python3-dev python3-venv libssl-dev libffi-dev libnss3 zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

# Test Speech Recognition

> sudo apt install -y portaudio19-dev

> python3 -m venv test

> source test/bin/activate

> pip install sounddevice SpeechRecognition PyAudio

> nano test.py

```
import sounddevice
import speech_recognition as sr
with sr.Microphone() as source:
  r = sr.Recognizer()
  audio = r.listen(source)
  print(r.recognize_google(audio, language="en-US"))
```

> python3 test.py

Say something ...

# Install ROCm [For AMD GPU]

Run in HOST:

> incus config edit ub

Add kdf to under `devices`:

```
devices:
  kfd:
    gid: "110"
    source: /dev/kfd
    type: unix-char
```

> incus restart ub

Run in CONTAINER:

> wget https://repo.radeon.com/amdgpu-install/6.0.2/ubuntu/jammy/amdgpu-install_6.0.60002-1_all.deb

> sudo apt install ./amdgpu-install_6.0.60002-1_all.deb

> sudo amdgpu-install --rocmrelease=6.0.2 --usecase=graphics,opencl,openclsdk,hip,hiplibsdk,rocm,rocmdev,rocmdevtools,lrt,mllib,mlsdk --vulkan=amdvlk,pro --no-dkms -y --accept-eula

For multiple GPUs setup, read https://github.com/eliranwong/MultiAMDGPU_AIDev_Ubuntu

# Troubleshooting - Snap Apps

To test:

> sudo apt install snapd

> sudo snap install firefox

> firefox

Output:

```
Error: cannot open display: :1
```

Solution:

> incus config edit ub

Add the following content under section `devices`

```
devices:
  X11Display:
    bind: container
    connect: unix:@/tmp/.X11-unix/X1
    listen: unix:@/tmp/.X11-unix/X1
    type: proxy
```

Remarks: To work with snap apps, DO NOT USE disk-type device, as suggested in [another post](https://discuss.linuxcontainers.org/t/ai-tutorial-rocm-and-pytorch-on-amd-apu-or-gpu/19743):

```
  x11_socket:
    source: /tmp/.X11-unix/X1
    path: /mnt/.container_sockets/X1
    type: disk
```

# Troubleshooting - Loading GTK Apps 

To test:

> sudo apt install gnome-text-editor

> gedit

## Issue 1

Failed to load module "canberra-gtk-module"

Solution:

> sudo apt install -y xdg-desktop-portal-gnome

## Issue 2

GTK gui applications are very slow to loaded for display.

Solution:

> sudo apt purge xdg-desktop-portal-gnome

> sudo apt purge xdg-desktop-portal-gtk

# Install Missing Drivers [optional]

Mesa

> sudo add-apt-repository ppa:kisak/kisak-mesa

> sudo apt update && sudo apt full-upgrade

Vulkan

> sudo add-apt-repository ppa:oibaf/graphics-drivers

> sudo apt update

> sudo apt install -y libvulkan1 mesa-vulkan-drivers vulkan-tools

Autoinstall

> sudo apt install -y ubuntu-drivers-common

> sudo ubuntu-drivers autoinstall

# Add Input Method [optional]

1. Manage languages

Example - add Chinese package

Edit /etc/locale.gen:

> sudo nano /etc/locale.gen

Locate and uncomment "zh_CN.UTF8 UTF8"

To download the newly selected language package(s), run:

> sudo locale-gen

2. Add Chinese fonts

> sudo apt install -y xfonts-wqy ttf-wqy-zenhei ttf-wqy-microhei fonts-arphic-bkai00mp fonts-arphic-bsmi00lp fonts-arphic-gbsn00lp fonts-arphic-gkai00mp xfonts-intl-chinese xfonts-intl-chinese-big

3. Add input method

Example - add ibus-pinyin

> sudo apt install ibus-pinyin

> ibus restart

> ibus-setup

4. Configure ibus as default input:<br>

> sudo apt install im-config

> im-config<br>

select "OK"<br>
select "Yes"<br>
select "ibus"<br>
select "OK"<br>

5. Add environment variables

> nano ~/.bashrc

```
export LC_CTYPE=zh_CN.UTF-8
export XIM=ibus
export XIM_PROGRAM=/usr/bin/ibus
export QT_IM_MODULE=ibus
export GTK_IM_MODULE=ibus
export XMODIFIERS=@im=ibus
export DefaultIMModule=ibus
```

# Resources

incus info --resources

To add a device, read https://linuxcontainers.org/incus/docs/main/reference/devices/

# Log information

> incus info --show-log ub

# Delete

To delete a container:

> incus delete mycontainername

# Uninstall

> sudo apt remove --autoremove incus incus-base

> sudo rm /etc/apt/sources.list.d/zabbly-incus-stable.sources /etc/apt/keyrings/zabbly.asc

# References

https://linuxcontainers.org/incus/docs/main/

https://ubuntuhandbook.org/index.php/2024/02/use-incus-container-ubuntu/

https://discuss.linuxcontainers.org/t/incus-lxd-profile-for-gui-apps-wayland-x11-and-pulseaudio/18295

https://xahteiwi.eu/resources/hints-and-kinks/lxc-basics/

https://discourse.ubuntu.com/t/lxd-incus-profile-for-gui-apps-wayland-x11-and-pulseaudio/40255

https://discuss.linuxcontainers.org/t/how-to-add-sound-to-an-ubuntu-lxd-vm/14372

https://discuss.linuxcontainers.org/t/audio-via-pulseaudio-inside-container/8768

https://stackoverflow.com/questions/45700653/can-my-docker-container-app-access-the-hosts-microphone-and-speaker-mac-wind

https://askubuntu.com/questions/508221/sound-input-device-microphone-not-working

https://linuxcontainers.org/incus/docs/main/faq/

https://unix.stackexchange.com/questions/594541/gtk-apps-slow-if-not-root

https://discuss.linuxcontainers.org/t/ai-tutorial-rocm-and-pytorch-on-amd-apu-or-gpu/19743
