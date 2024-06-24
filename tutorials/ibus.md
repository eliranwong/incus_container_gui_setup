# Tutorial: Ibus Setup in a GUI-enabled Incus Container

"IBus (Intelligent Input Bus) is an input method framework, a type of application that allows for easily switching between different keyboard layouts."

In this tutorial, we will learn:

1. How to set up ibus in a GUI-enabled incus container
2. How to add a ibus input method
3. How to autostart ibus daemon

In this tutorial, we will set up two input methods as an example:
* English (UK)
* Chinese - Pinyin

# Set up a GUI-enabled Incus Container

This tutorial assume we work with a container named 'ub' that supports GUI applications. you may change the container name 'ub' to your own ones.

To set up a GUI-enabled Incus Container, read one of the following examples:

[Running GUI Apps via Incus on Ubuntu 22.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_22.04_LTS_tested.md)

[Running GUI Apps via Incus on Ubuntu 24.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_24.04_LTS_tested.md)

# Manage language packages

English package is installed by default.

In this example, we will add a Chinese package

Edit /etc/locale.gen:

> sudo nano /etc/locale.gen

Locate and uncomment "zh_CN.UTF8 UTF8"

To download the newly selected language package(s), run:

> sudo locale-gen

# Add Chinese fonts

> sudo apt install -y xfonts-wqy ttf-wqy-zenhei ttf-wqy-microhei fonts-arphic-bkai00mp fonts-arphic-bsmi00lp fonts-arphic-gbsn00lp fonts-arphic-gkai00mp xfonts-intl-chinese xfonts-intl-chinese-big

# Add input method

Example - add ibus-pinyin

> sudo apt install ibus-pinyin

> ibus restart

> ibus-setup

Go to tab "Input Method":

1. Select "Add" > "English" > "English (UK)" > "Add"
2. Select "Add" > "Chinese" > "Pinyin" > "Add"
3. Select "Close"

# Configure ibus as default input:

This step is important! Don't miss it.

> sudo apt install im-config

> im-config

Select options according to the following order as you walk through the im-config dialog:

```
select "OK"
select "Yes"
select "ibus"
select "OK"
```

# Add environment variables

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

> source ~/.bashrc

# Auto-start Ibus Daemon

Append the ~/.profile file to auto-start ibus daemon:

> echo "ibus-daemon -drx" >> ~/.profile

> sudo reboot

Now, when you login this container again, you can see the ibus interface will be seamlessly loaded on the "Top Panel" or "Header Bar", where you can change the ibus input methods for your container GUI applications.