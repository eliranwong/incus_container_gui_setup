# Loading GTK Applications in a GUI-enabled Incus Container

Recently, I have been working on running everything, including GUI applications, with incus containers.

For examples, I managed to work with all the following features with incus containers:

Hardware: GPU, Sound, Microphone, Camera

GUI Applications: X11 apps, Qt-based GUI apps, GTK-based apps, Snap apps, System-tray apps

Input method: ibus

Nested container: docker

AI Tools: Llama.cpp, Ollama, Perplexica

If you are interested about the container setup, you may read:

[Running GUI Apps via Incus on Ubuntu 22.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_22.04_LTS_tested.md)

[Running GUI Apps via Incus on Ubuntu 24.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_24.04_LTS_tested.md)

During the setup and testings, two issues had troubled me for a while.  The first one is how to work out display for sanp applications, which was finally resolved, as integrated with the setup just mentioned.  The second one concerns GTK-based applications, with which I encountered the following two issues.  I would like to share my solutions below.  Anyone is welcome to suggest better alternatives.

# Testing GTK-based applications

For testing purpose, install and run `gnome-text-editor`

> sudo apt install gnome-text-editor

> gnome-text-editor

## Issue 1 - Failed to load module "canberra-gtk-module"

This issue is minor, and easy to fix.

Solution:

> sudo apt install -y xdg-desktop-portal-gnome

## Issue 2 - Extremely slow to start

GTK gui applications are extremely slow to be loaded for display.

Solution:

> sudo apt purge xdg-desktop-portal-gnome

> sudo apt purge xdg-desktop-portal-gtk

# Final Tips

I believe you are smart enought to spot that the first solution is `sudo apt install -y xdg-desktop-portal-gnome` whereas the second solution is `sudo apt purge xdg-desktop-portal-gnome`. To solve both issues, implement the first solution once, before running the second solution.

Hope this may help those who had the same puzzles.
