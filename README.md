# WSLg+Docker Setup on Windows 11

## On Windows 11
- ~Install [**Windows Subsystem for Linux Preview**](https://www.microsoft.com/store/productId/9P9TQF7MRM4R) (*Microsoft Store App)~
- Download and install
  - ~[**CUDA on Windows Subsystem for Linux (WSL)**](https://developer.nvidia.com/cuda/wsl/download)~
  - [**Visual Studio Code**](https://code.visualstudio.com/download)
- Open **Windows Terminal (Admin)**
  - Displays a list of available distributions for install
  - Install **Ubuntu 18.04 LTS (or any other ubuntu distribution like Ubuntu-20.04)**

```bash
> wsl --list --online
The following is a list of valid distributions that can be installed.
Install using 'wsl.exe --install <Distro>'.

NAME            FRIENDLY NAME
Ubuntu          Ubuntu
Debian          Debian GNU/Linux
kali-linux      Kali Linux Rolling
openSUSE-42     openSUSE Leap 42
SLES-12         SUSE Linux Enterprise Server v12
SLES-15         SUSE Linux Enterprise Server v15
Ubuntu-18.04    Ubuntu 18.04 LTS
Ubuntu-20.04    Ubuntu 20.04 LTS

> wsl --install -d Ubuntu-18.04

> wsl -l -v
  NAME            STATE           VERSION
* Ubuntu-18.04    Running         2

> wsl --update

> wsl --shutdown

> wsl --status
Default Distribution: Ubuntu-18.04
Default Version: 2
WSL version: 0.50.2.0
Kernel version: 5.10.74.3
WSLg version: 1.0.29
Windows version: 10.0.22000.376
```

- Launch **Visual Studio Code**
  - Open a remote WSL window using **disto Ubuntu-18.04**

## On Ubuntu 18.04
- Check kernel version and see if it is up-to-date

```bash
$ uname -r
5.10.74.3-microsoft-standard-WSL2
```

- Now, see if GUI apps work and GPU is recognized
```bash
$ sudo apt-get update
$ sudo apt-get install -y x11-apps
$ xeyes
$ nvidia-smi
Mon Jan  3 11:58:45 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.00       Driver Version: 510.06       CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:07:00.0  On |                  N/A |
|  0%   38C    P8    20W / 215W |    555MiB /  8192MiB |     N/A      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

- Install **Docker Engine on Ubuntu** following the instruction (https://docs.docker.com/engine/install/ubuntu/)
```bash
$ sudo apt-get update
$ sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
$ sudo apt-get update
$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io
```

- Add user to docker group and restart Ubuntu (**wsl shutdown** command on Windows Terminal)
```bash
$ sudo usermod -aG docker $USER
```

- Docker service does NOT automatically start on WSL by default
```bash
$ docker run hello-world
docker: Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?.
See 'docker run --help'.

$ sudo service docker start
 * Starting Docker: docker 

$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

- However, by setting the following command in [**/etc/wsl.conf**](https://docs.microsoft.com/en-us/windows/wsl/wsl-config#wslconf), docker service starts automatically when a WSL instance launches
```bash
# Set a command to run when a new WSL instance launches. This example starts the Docker container service.
[boot]
command = service docker start
```

- Install **NVIDIA Container Toolkit** following the instruction (https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
```bash
$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
   && curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add - \
   && curl -s -L https://nvidia.github.io/nvidia-docker/$distribution/nvidia-docker.list | sudo tee /etc/apt/sources.list.d/nvidia-docker.list
$ sudo apt-get update
$ sudo apt-get install -y nvidia-docker2
$ sudo service docker restart
$ docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
Mon Jan  3 04:34:34 2022       
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 510.00       Driver Version: 510.06       CUDA Version: 11.6     |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|                               |                      |               MIG M. |
|===============================+======================+======================|
|   0  NVIDIA GeForce ...  On   | 00000000:07:00.0  On |                  N/A |
|  0%   48C    P8    16W / 215W |    596MiB /  8192MiB |     N/A      Default |
|                               |                      |                  N/A |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                                  |
|  GPU   GI   CI        PID   Type   Process name                  GPU Memory |
|        ID   ID                                                   Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```
## Running ORB-SLAM3 Example

- Build docker image using Dockerfile
```bash
$ mkdir Downloads
$ cd Downloads/
$ git clone https://github.com/obikata/Dockerfiles.git
$ git clone https://github.com/obikata/orb-slam3.git
$ wget http://robotics.ethz.ch/~asl-datasets/ijrr_euroc_mav_dataset/machine_hall/MH_01_easy/MH_01_easy.zip
$ unzip MH_01_easy.zip -d orb-slam3/Examples/MH01
$ cd Dockerfiles/orb-slam3-wslg
$ docker build -t orb-slam3-wslg .
```

- Run a new container using the built image
```bash
docker run --gpus all --env="DISPLAY" --volume="/tmp/.X11-unix:/tmp/.X11-unix:rw" --name orb-slam3-wslg -d -it --mount type=bind,src=/home/obikata/Downloads,dst=/home/obikata/Downloads orb-slam3-wslg
```

- Start and attach to the new container
```bash
$ docker start orb-slam3-wslg
$ docker attach orb-slam3-wslg
```

- Run ORB-SLAM3 (e.g. EuRoC MH01)
```bash
$ cd Downloads/orb-slam3/
$ ./build.sh
$ ./Examples/Monocular/mono_euroc Vocabulary/ORBvoc.txt Examples/Monocular/EuRoC.yaml Examples/MH01/ Examples/Monocular/EuRoC_TimeStamps/MH01.txt
```
