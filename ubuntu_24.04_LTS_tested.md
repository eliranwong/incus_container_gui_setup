# Ubuntu 24.04 LTS

Tested Ubuntu 24.04 LTS, running on a [mobile tablet](https://starlabs.systems/pages/starlite).

# Install incus

> sudo apt install -y incus

> sudo adduser $LOGNAME incus-admin

> newgrp incus-admin

> incus admin init

# Check Host Devices

> incus info --resources | grep "PCI address: " -B 4

Output:

```
GPU:
  NUMA node: 0
  Vendor: Intel Corporation (8086)
  Product: Alder Lake-N [UHD Graphics] (46d0)
  PCI address: 0000:00:02.0
```

> ls /dev/dri

Output:

```
by-path  card1  renderD128
```

> incus info --resources | grep [Cc]am -B 3 -A 4

Output:

```
  Device 3:
    Vendor: Realtek Semiconductor Corp.
    Vendor ID: 0bda
    Product: USB2.0 camera
    Product ID: 5830
    Bus Address: 1
    Device Address: 7
  Device 4:
    Vendor: Sunplus Innovation Technology Inc.
    Vendor ID: 1bcf
    Product: USB 2.0 Camera
    Product ID: 284d
    Bus Address: 1
    Device Address: 8
```

> ls /dev/snd/control*

Output:

```
/dev/snd/controlC0
```

> ls /dev/video*

Output:

```
/dev/video0  /dev/video1  /dev/video2  /dev/video3
```

# Run in Host

> incus launch images:ubuntu/noble/cloud ub

> echo "alias ub="incus exec ub -- sudo --login --user ubuntu" >> ~/.bashrc

> source .bashrc

> xhost +local:

> echo "xhost +local:" >> ~/.profile

> incus config edit ub

```
description: GUI profile for running Ubuntu 24.04 LTS on Star Lite
config:
  security.nesting: "true"
devices:
  camera1:
    busnum: "1"
    devnum: "7"
    gid: "1000"
    productid: "5830"
    type: usb
    uid: "1000"
    vendorid: 0bda
  camera2:
    busnum: "1"
    devnum: "8"
    gid: "1000"
    productid: 284d
    type: usb
    uid: "1000"
    vendorid: 1bcf
  controlC0:
    source: /dev/snd/controlC0
    type: unix-char
  dri_card0:
    gid: "44"
    source: /dev/dri/card1
    type: unix-char
  dri_renderD128:
    gid: "110"
    source: /dev/dri/renderD128
    type: unix-char
  gpu:
    gid: "44"
    pci: "0000:00:02.0"
    type: gpu
  pulseovernetwork:
    bind: container
    connect: tcp:127.0.0.1:4713
    listen: tcp:127.0.0.1:4713
    type: proxy
  video0:
    gid: "44"
    source: /dev/video0
    type: unix-char
  video1:
    gid: "44"
    source: /dev/video1
    type: unix-char
  video2:
    gid: "44"
    source: /dev/video2
    type: unix-char
  video3:
    gid: "44"
    source: /dev/video3
    type: unix-char
  x11_server:
    bind: container
    connect: unix:@/tmp/.X11-unix/X1
    listen: unix:@/tmp/.X11-unix/X1
    type: proxy
```

To grant network access for audio:

> mkdir -p ~/.config/pipewire/pipewire-pulse.conf.d

> nano ~/.config/pipewire/pipewire-pulse.conf.d/pulse-tcp.conf

Add the following content:

```
context.exec = [
 { path = "pactl"  args = "load-module module-native-protocol-tcp auth-ip-acl=127.0.0.1" }
]
```

> incus restart ub

> ub

# In CONTAINER

> sudo usermod -a -G render,video $LOGNAME

> sudo apt install linux-headers-\`uname -r\`

> sudo reboot

> sudo apt install -y pulseaudio-utils dbus-user-session dbus-x11 ibus im-config nano micro espeak x11-apps build-essential guvcview alsa-base alsa-utils pavucontrol gedit falkon

> nano ~/.bashrc

Add the following content at the end of the file:

```
export DISPLAY=:1
export PULSE_SERVER=tcp:127.0.0.1:4713
```

> source ~/.bashrc

> ibus-daemon -drx

> echo "ibus-daemon -drx" >> ~/.profile

# Testings

GUI X11 App:

> xclock

GTK Apps:

> gedit

Fix slow startup:

> sudo apt purge xdg-desktop-portal-gnome

> sudo apt purge xdg-desktop-portal-gtk

Qt Apps:

> sudo apt install -y libxcb-cursor0

> falkon

Snap Apps:

> sudo apt install snapd

> sudo anap install firefox

System-tray Apps:

> ibus-daemon -drx

Sound:

> espeak testing

Camera:

> guvcview

Microphone

> pavucontrol

Input Method:

> ibus-setup
