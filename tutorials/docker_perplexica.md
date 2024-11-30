# Tutorial: Docker + Perplexica + Snap + Firefox in a GUI-enabled Incus Container (AI)

"Perplexica is an open-source AI-powered searching tool or an AI-powered search engine that goes deep into the internet to find answers."

In this tutorial, we will learn:

1. How to set up a GUI-enabled incus container
2. Configure a docker-enabled device
3. Install Docker in an incus container
4. Install Perplexica in an incus container
5. Install snap firfox to launch Perplexica

# Set up a GUI-enabled Incus Container

This tutorial assume we work with a container named 'ub' that supports GUI applications. you may change the container name 'ub' to your own ones.

To set up a GUI-enabled Incus Container, read one of the following examples:

[Running GUI Apps via Incus on Ubuntu 22.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_22.04_LTS_tested.md)

[Running GUI Apps via Incus on Ubuntu 24.04 LTS](https://github.com/eliranwong/incus_container_gui_setup/blob/main/ubuntu_24.04_LTS_tested.md)

# Configure a Docker Device

1. Create a btrfs storage pool
2. Create a volume to work with docker
3. Attach the volume to an incus container
4. Restart container to make new configurations effective.

Run in HOST:

```
# Create a btrfs storage pool, as docker does not run well with the default zfs file system
sudo apt install btrfs-progs
incus storage create btrfspool btrfs
# Create a volume to work with docker
incus storage volume create btrfspool docker
# Attach volume 'docker' to container 'ub'
incus config device add ub docker disk pool=btrfspool source=docker path=/var/lib/docker
# Configure security options to work with docker
incus config set ub security.nesting=true security.syscalls.intercept.mknod=true security.syscalls.intercept.setxattr=true
# Restart container 'ub'
incus restart ub
```

# Log in a GUI-enabled Incus Container

Run in HOST:

```
incus exec ub -- sudo --login --user ubuntu
```

# Install Docker

Run in CONTAINER:

1. Set up Docker's apt repository:

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Install the latest version

```
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Add user to docker group

```
sudo usermod -aG docker $LOGNAME
newgrp docker
```

# Install Git and Perplexica

Run in CONTAINER:

```
sudo apt install -y git
git clone https://github.com/ItzCrazyKns/Perplexica.git
cd Perplexica
cp sample.config.toml config.toml
docker compose up -d
```

# Install Snap and Firfox

Run in CONTAINER:

```
sudo apt install -y snapd
sudo snap install firefox
```

# Launch Perplexica

Run in CONTAINER:

```
firefox localhost:3000
```

Configure Perplexica settings by first clicking the 'wheel' button at the left bottom corner.

# Launch from Host

Run in HOST:

1. Set aliases

> nano ~/.bashrc

Add the following content at the end of the file

```
alias ub="incus exec ub -- sudo --login --user ubuntu"
alias perplexica="incus exec ub -- sudo --login --user ubuntu firefox localhost:3000 &>/dev/null & disown"
```

> source ~/.bashrc

2. To launch Perplexica from host, run either:

> ub firefox localhost:3000

or

> perplexica

# Direct Access from the Host

Install socat for port forwarding:

```
sudo apt -y install socat
echo -e "# forward perplexica backend port\nif ! nc -z localhost 3001 &> /dev/null; then\n  nohup socat TCP-LISTEN:3001,fork TCP:$(incus list | grep '[0-9] (eth0)' | awk '{print $4}' | cut -d'(' -f1):3001&\nfi\nalias perplexica=\"open http://$(incus list | grep '[0-9] (eth0)' | awk '{print $4}' | cut -d'(' -f1):3000\"" | tee -a ~/.bashrc
source ~/.bashrc
```

Remarks: Forwarding port 3001 is necessary for access to Perplexica backend server.

# References

https://docs.docker.com/engine/install/ubuntu/

https://ubuntu.com/tutorials/how-to-run-docker-inside-lxd-containers#1-overview

https://github.com/ItzCrazyKns/Perplexica
