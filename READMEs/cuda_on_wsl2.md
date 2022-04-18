# WSL2 + Docker + NVIDIA Container Toolkit

Installation instruction to get Docker container to use NVIDIA Container Toolkit for GPU acceleration, within WSL2 distribution.

Officially, the original instructions are available at [**CUDA on WSL User Guide**](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#ch02-getting-started).

This is a step-by-step instruction including all the little details of the process.

## System Configuration

- Windows 11 Version 10.0.22000 Build 22000
- Ubuntu 20.04 in WSL2
- Kernel version `Linux version 5.10.102.1-microsoft-standard-WSL2`
- GPU is NVIDIA Quadro P4000

## Procedure

### Installing WSL2 with Ubuntu

Assuming using Windows 11, Windows Terminal will be available by default and should open up a PowerShell prompt. Official instructions are also available at [**Install WSL**](https://docs.microsoft.com/en-us/windows/wsl/install).

1. In an elevated (Run as Administrator) Windows Terminal, run

    ```powershell
    PS C:\Users\username>wsl --install -d Ubuntu-20.04
    ```

2. Restart the machine as requested.
3. Setup a user for Ubuntu (should automatically pops up).
4. Open another elevated Windows Terminal or PowerShell windows, run

    ```powershell
    PS C:\Users\username>wsl --shutdown
    PS C:\Users\username>wsl --update
    ```

### Installing Docker

Official instructions available at [**Install Docker Engine on Ubuntu**](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository). Alternatively, just run `curl https://get.docker.com | sh`.

1. Start a WSL2 windows (Windows Terminal > New Tab dropdown > Ubuntu-20.04).
2. Update sources and (optionally) update all packages.

    ```console
    user@host:~$ sudo apt-get update
    user@host:~$ sudo apt-get dist-upgrade    # optional
    ```

3. Setup Docker repository and install Docker.

    ```console
    user@host:~$ sudo apt-get install ca-certificates curl gnupg lsb-release
    user@host:~$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    user@host:~$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    user@host:~$ sudo apt-get update
    user@host:~$ sudo apt-get install docker-ce docker-ce-cli containerd.io
    ```

4. Follow [**Post-installation steps for Linux**](https://docs.docker.com/engine/install/linux-postinstall/) to run `docker` without `sudo`.

    ```console
    user@host:~$ sudo groupadd docker       # probably will say group `docker` already exists...
    user@host:~$ sudo usermod -aG docker $USER
    ```

    At this point, kill WSL2 by running `wsl --shutdown` in PowerShell, and reopen WSL windows (effectively "restart"?).

    ```console
    user@host:~$ newgrp docker
    user@host:~$ docker run --rm hello-world

    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    [...]
    ```

    You should see the `hello-world` image output as above. If it complains about something something `socket`, complete step 5, then retry this step.

5. Make sure `docker` service is always running when using WSL.

    ```console
    user@host:~$ sudo vim /etc/wsl.conf     # or a text editor of your choice

    # in Vim
    [boot]
    command = service docker start
    ~
    ~
    ~
    [...]

    # save file, double check
    user@host:~$ cat /etc/wsl.conf
    [boot]
    command = service docker start
    ```

    kill WSL2 by running `wsl --shutdown` and open up another one.

### Installing NVIDIA Container Toolkit (nvidia-docker2)

Official instructions available at [**Install NVIDIA Container Toolkit**](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#ch04-sub02-install-nvidia-docker).

