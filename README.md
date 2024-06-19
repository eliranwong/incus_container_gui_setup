# incus_container_gui_setup

Setup of running GUI applications in a [incus container](https://linuxcontainers.org/incus/docs/main/)

Tested on [Ubuntu 22.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_22.04_LTS_tested.md)

Tested on [Ubuntu 24.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_24.04_LTS_tested.md)

# Install Incus on Ubuntu 24.04 LTS

> sudo apt install -y incus

> sudo adduser $LOGNAME incus-admin

> newgrp incus-admin

> incus admin init

# Install Incus on Ubuntu 22.04 LTS

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

# Run GUI Apps

Tested on [Ubuntu 22.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_22.04_LTS_tested.md)

Tested on [Ubuntu 24.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_24.04_LTS_tested.md)

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

On Ubuntu 22.04, also run:

> sudo rm /etc/apt/sources.list.d/zabbly-incus-stable.sources /etc/apt/keyrings/zabbly.asc
