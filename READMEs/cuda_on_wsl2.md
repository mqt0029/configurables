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
    ```

    In Vim, or your text editor, add

    ```
    [boot]
    command = service docker start
    ```

    Save the file (require sudo), and double check content of file.

    ```console
    user@host:~$ cat /etc/wsl.conf
    [boot]
    command = service docker start
    ```

    Again, kill WSL2 by running `wsl --shutdown` and open up another one.

### Installing NVIDIA Container Toolkit (nvidia-docker2)

Official instructions available at [**Install NVIDIA Container Toolkit**](https://docs.nvidia.com/cuda/wsl-user-guide/index.html#ch04-sub02-install-nvidia-docker).

```console
user@host:~$ distribution=$(. /etc/os-release;echo $ID$VERSION_ID)
user@host:~$ curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo apt-key add -
user@host:~$ sudo apt-get update
user@host:~$ sudo apt-get install -y nvidia-docker2
user@host:~$ sudo service docker stop
user@host:~$ sudo service docker start
user@host:~$ docker run --gpus all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```

You should now see output relating to your GPU running the benchmark. Mine looks like this

```
user@host:~$ docker run --gpus all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
Run "nbody -benchmark [-numbodies=<numBodies>]" to measure performance.
        -fullscreen       (run n-body simulation in fullscreen mode)
        -fp64             (use double precision floating point values for simulation)
        -hostmem          (stores simulation data in host memory)
        -benchmark        (run benchmark to measure performance)
        -numbodies=<N>    (number of bodies (>= 1) to run in simulation)
        -device=<d>       (where d=0,1,2.... for the CUDA device to use)
        -numdevices=<i>   (where i=(number of CUDA devices > 0) to use for simulation)
        -compare          (compares simulation results running once on the default GPU and once on the CPU)
        -cpu              (run n-body simulation on the CPU)
        -tipsy=<file.bin> (load a tipsy model file for simulation)

NOTE: The CUDA Samples are not meant for performance measurements. Results may vary when GPU Boost is enabled.

> Windowed mode
> Simulation data stored in video memory
> Single precision floating point simulation
> 1 Devices used for simulation
GPU Device 0: "Pascal" with compute capability 6.1

> Compute 6.1 CUDA device: [Quadro P4000]
14336 bodies, total time for 10 iterations: 15.540 ms
= 132.251 billion interactions per second
= 2645.018 single-precision GFLOP/s at 20 flops per interaction
```

## Building and Running images with access to WSL vGPU

Official instructions available at [**Containerized applications access to the vGPU**](https://github.com/microsoft/wslg/blob/main/samples/container/Containers.md#containerized-applications-access-to-the-vgpu=).

In my case, I tested my setup by running Gazebo, a simulation software commonly used with ROS. Framerates went from ~8-10 FPS to ~52 FPS. An extended set of instructions on [**Docker Hardware Acceleration**](http://wiki.ros.org/docker/Tutorials/Hardware%20Acceleration#nvidia-docker2) is also provided by ROS contributors.

1. Make a `Dockerfile` for your image. It is important that you have the `ENV LD_LIBRARY_PATH=/usr/lib/wsl/lib` line.

    ```Dockerfile
    FROM osrf/ros:noetic-desktop-full-focal

    ARG DEBIAN_FRONTEND=noninteractive

    # You may want to install mesa-utils to display and test OpenGL configs and performance
    RUN apt-get update && apt-get install -y mesa-utils

    ENV LD_LIBRARY_PATH=/usr/lib/wsl/lib

    ENV NVIDIA_VISIBLE_DEVICES \
        ${NVIDIA_VISIBLE_DEVICES:-all}
    ENV NVIDIA_DRIVER_CAPABILITIES \
        ${NVIDIA_DRIVER_CAPABILITIES:+$NVIDIA_DRIVER_CAPABILITIES,}graphics
    ```

2. Build the image

    ```console
    user@host:~$ docker build -t test_gpu -f Dockerfile .
    [...]
    ```

3. Create a convenient launch script to test containers generated from the image. Call it `test_cuda_container.sh`.

    ```bash
    XAUTH=/tmp/.docker.xauth
    if [ ! -f $XAUTH ]
    then
        xauth_list=$(xauth nlist :0 | sed -e 's/^..../ffff/')
        if [ ! -z "$xauth_list" ]
        then
            echo $xauth_list | xauth -f $XAUTH nmerge -
        else
            touch $XAUTH
        fi
        chmod a+r $XAUTH
    fi

    # you may also want the --privileged flag to access more devices
    docker run -i -t --rm \
        --device=/dev/dxg \
        --runtime=nvidia \
        --gpus=all \
        -e "WAYLAND_DISPLAY=$WAYLAND_DISPLAY" \
        -e "QT_X11_NO_MITSHM=1" \
        -e "XDG_RUNTIME_DIR=$XDG_RUNTIME_DIR" \
        -e "PULSE_SERVER=$PULSE_SERVER" \
        -e "DISPLAY=$DISPLAY" \
        -e "XAUTHORITY=$XAUTH" \
        -v "/tmp/.X11-unix:/tmp/.X11-unix" \
        -v "/mnt/wslg:/mnt/wslg" \
        -v "/usr/lib/wsl:/usr/lib/wsl" \
        -v "$XAUTH:$XAUTH" \
        test_gpu \
        bash
    ```

4. Make the script executable and run the container. You should see the container prompt with user `root`.

    ```console
    user@host:~$ chmod +x test_cuda_container.sh
    user@host:~$ ./test_cuda_container.sh
    root@7116fd6c4f81:/#
    ```

5. Test your graphics application. In my example, I just run `gazebo` and check the framerate.
6. (Optional) If you have `mesa-utils` installed, you can also check if OpenGL is rendering with your GPU.

    ```console
    root@89f97b8b6ca9:/# glxinfo | grep "OpenGL"
    OpenGL vendor string: Microsoft Corporation
    OpenGL renderer string: D3D12 (NVIDIA Quadro P4000)     # This is the important line!
    OpenGL core profile version string: 3.3 (Core Profile) Mesa 21.2.6
    OpenGL core profile shading language version string: 3.30
    OpenGL core profile context flags: (none)
    OpenGL core profile profile mask: core profile
    OpenGL core profile extensions:
    OpenGL version string: 3.1 Mesa 21.2.6
    OpenGL shading language version string: 1.40
    OpenGL context flags: (none)
    OpenGL extensions:
    OpenGL ES profile version string: OpenGL ES 3.0 Mesa 21.2.6
    OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.00
    OpenGL ES profile extensions:
    root@89f97b8b6ca9:/#
    ```

    Your `OpenGL renderer string` should say NVIDIA GPU like it did on mine.