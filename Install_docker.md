# Docker
Created Sunday 10 June 2018

### For Ubuntu
1. [Ubuntu Version - 18.04 LTS Bionic Beaver](https://www.ubuntu.com/download/desktop)
1. [Official Docker installation guide for Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
1. Apt in Ubuntu has alternate **docker.io**. The docker mentioned here referes to https://www.docker.com.

### For Raspbian
1. [Raspbian Stretch](https://www.raspberrypi.org/downloads/raspbian/)
1. [Official Docker installtion guide for Raspbian](https://docs.docker.com/install/linux/docker-ce/debian/)

#### Uninstall older versions
```
sudo apt-get remove docker docker-engine docker.io
```
**List installed packages to check**
```
sudo apt list --installed | grep docker
```
After installation the output in my system for this command was -
| Distro | Output |
| --- | --- |
| Ubuntu | `docker-ce/bionic,now 18.05.0~ce~3-0~ubuntu amd64 [installed]` |

Evan after removing docker using the above mentioned steps the contents of **/var/lib/docker/**, including images, containers, volumes, and networks, are preserved.

## **Main installtion steps**
---

### Allow apt to use repository over https
| Distro | Command |
| --- | --- |
| Ubuntu | `sudo apt-get -yf install apt-transport-https ca-certificates curl software-properties-common`|
| Raspbian | `sudo apt-get -yf install apt-transport-https ca-certificates curl gnupg2 software-properties-common` | 

### Add Docker's official GPG key
| Distro | Command |
| --- | --- |
| Ubuntu | `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -` |
| Raspbian | `curl -fsSL https://download.docker.com/linux/raspbian/gpg | sudo apt-key add -` |

Verify the key 9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88
```
sudo apt-key fingerprint 0EBFCD88
```

You can serach [here](https://download.docker.com/linux/) for the relevant version for your system.

## Setup repository
There are three types of repositories - `stable`, `edge` and `test`. `stable` reposioty is needed even if `edge` or `test` are used. 
Use the following command to set up the `stable` repository. To add the edge or test repository, add the word `edge` or `test` (or both) after the word `stable` in the commands below.

first determine hardware architecture- 
```
dpkg --print-architecture
```

`egde` releases are monthly compared to quarterly `stable`. For more info about this refer [here](https://docs.docker.com/install/).

| Distro | Command |
| --- | --- |
| Ubuntu **x86_64** | `sudo add-apt-repository "deb [arch=amd64] <https://download.docker.com/linux/ubuntu> $(lsb_release -cs) stable"`
| Raspbian **armhf** | `echo "deb [arch=armhf] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list` |

In my case there were some issue with the Ubuntu installation while using only `stable`. As it is quite early in Ubuntu18. After some searching I had to add the `edge` as well. 
```
sudo add-apt-repository "deb [arch=amd64] <https://download.docker.com/linux/ubuntu> $(lsb_release -cs) stable edge"
```

## Install Docker-CE
```
sudo apt-get update
sudo apt-get -yf install docker-ce
```

>To install specific version use
>`apt-cache madison docker-ce
<<docker-ce | **18.03.0~ce-0~ubuntu** | <https://download.docker.com/linux/ubuntu> xenial/stable amd64 Packages`.
>Install a specific version by its fully qualified package name, which is package name `(docker-ce) “=”` version string (2nd column), for example,`docker-ce=18.03.0~ce-0~ubuntu`. Command is `sudo apt-get install docker-ce=<VERSION>`

After the package installtion is complete you would like to check if you can say _hurrah_.
### Check Docker running staus
Either of the below commands will work
```
sudo systemctl is-active docker

sudo status docker

sudo service docker status
```

### Verify installation
The Docker daemon starts automatically.
| Distro | Command |
| --- | --- |
| Ubuntu | `sudo docker run hello-world` |
| Raspbian | `sudo docker run armhf/hello-world` |

This command will automatically pull the hello-world container image if mot present locally and run it.

## Docker help
```
dockerd --help
```

To stop starting of dockerd at boottime
---------------------------------------
```
sudo systemctl disable docker
```
### To enable docker to start at boottime
```
sudo systemctl enable docker
```


### Start the Docker daemon maually

```
dockerd
```
`Ctrl+C` to stop.

### To start as a service
systemctl: 
```
sudo systemctl start docker
```
service:
```
sudo service docker start
```

### To stop docker
```
sudo systemctl stop docker
```

## Uninstall Docker package
```
sudo apt-get purge docker-ce
sudo rm -rf /var/lib/docker
```


