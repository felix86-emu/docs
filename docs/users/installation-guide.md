---
title: Installation guide
---

felix86 can be installed automatically through an installation script, or manually.

Make sure [your RISC-V device is supported](./supported-devices.md) before installing.

=== "Automatic"
    You can install felix86 using the installation script:
    ```sh
    bash <(curl -s https://install.felix86.com)
    ```

    This script will guide you through installing felix86 and a rootfs.

    !!! example "Reading the script"
        This script is hosted via GitHub Pages and its source code is available at [https://github.com/felix86-emu/install](https://github.com/felix86-emu/install)

        You can also read the script before running it: `curl -s https://install.felix86.com | less`

        Or run it directly from the repository: `bash <(curl -s https://raw.githubusercontent.com/felix86-emu/install/refs/heads/main/index.html)`


=== "Manual"
    ## Prerequisites
    You need a C and C++ compiler like GCC or Clang, and CMake. If you want to build with thunking, and you should, you may need X11/GLX/Vulkan/Wayland header files. Linking is done at runtime using `dlopen`, so for thunking to properly work you also need the libraries themselves.
    === "Archlinux"
        ```
        sudo pacman -S --needed base-devel cmake libx11 mesa vulkan-devel vulkan-tools wayland
        ```

    === "Ubuntu/Debian"
        ```
        sudo apt install build-essential cmake libx11-dev libgl1-mesa-dev libglx-dev libvulkan-dev vulkan-tools libwayland-dev
        ```

    If you're cross-compiling, you need to install the RISC-V build tools.
    === "Archlinux"
        ```
        sudo pacman -S --needed riscv64-linux-gnu-gcc riscv64-linux-gnu-binutils
        ```

    === "Ubuntu/Debian"
        ```
        sudo apt install gcc-riscv64-linux-gnu g++-riscv64-linux-gnu
        ```


    ## Configuring
    If you're building on a RISC-V machine, the default configuration is good enough.
    ```
    cmake -B build
    ```
    If you're cross-compiling, you need to use the RISC-V CMake toolchain.
    ```
    cmake -B build -DCMAKE_TOOLCHAIN_FILE=riscv.cmake
    ```

    ## Building
    Just like any other CMake project.
    ```
    cmake --build build -j$(nproc)
    ```

    ## binfmt_misc installation
    Installing in binfmt_misc allows running privileged applications, such as `sudo`, through the emulator, so it is recommended.
    ```
    sudo ./build/felix86 -b
    ```

    ## Rootfs
    You can download a ready-made rootfs using the installation script.
    ```
    bash <(curl -s https://install.felix86.com/rootfs.sh)
    ```
    There's many ways to create an x86 rootfs. The felix86 project uses Docker, but you can also use [Debootstrap](https://wiki.debian.org/Debootstrap) or other similar tools.

    For Docker, you can start by using the example files below.
    === "Dockerfile"
        ```Dockerfile
        FROM ubuntu:24.04

        ENV DEBIAN_FRONTEND=noninteractive

        COPY Ubuntu.sh /tmp/Ubuntu.sh

        RUN bash /tmp/Ubuntu.sh

        WORKDIR /root

        CMD ["/bin/bash"]
        ```
    
    === "Ubuntu.sh"
        ```sh
        #!/bin/bash
        set -e
        dpkg --add-architecture i386
        apt update
        apt upgrade -y
        # Install packages you want inside your rootfs
        apt install -y sudo
        ```

    Then, run the following commands to extract your rootfs into an archive.
    ```sh
    sudo docker build -t "felix86-rootfs" "."
    sudo docker create --name "felix86-rootfs" "felix86-rootfs"
    sudo docker export "felix86-rootfs" | bsdtar -czf - \
        --exclude=media --exclude=mnt --exclude=root --exclude=srv \
        --exclude=boot --exclude=home --exclude=run --exclude=proc \
        --exclude=sys --exclude=dev --exclude=tmp --exclude=.dockerenv \
        @- > "felix86-rootfs.tar.gz"
    sudo docker rm "felix86-rootfs"
    sudo docker rmi "felix86-rootfs"
    ```

    Unzip this rootfs to your target directory on your RISC-V machine, and set it using `felix86 --set-rootfs /path/to/rootfs`

    ### Important files

    During rootfs installation, some important files are copied to the rootfs. Currently, these are the following:
    ```
    /etc/mtab
    /etc/passwd
    /etc/passwd-
    /etc/group
    /etc/group-
    /etc/shadow
    /etc/shadow-
    /etc/gshadow
    /etc/gshadow-
    /etc/hosts
    /etc/hostname
    /etc/timezone
    /etc/localtime
    /etc/fstab
    /etc/subuid
    /etc/subgid
    /etc/machine-id
    /etc/resolv.conf
    /etc/sudoers
    ```

    Copy these to the rootfs while retaining the host permissions.
    ```
    sudo cp -rp /etc/mtab $ROOTFS/etc/mtab
    ```

    It is recommended you also create a home directory for your user: `sudo mkdir "$ROOTFS/home/$USER" && sudo chown $USER:$USER "$ROOTFS/home/$USER"`

!!! tip
    Install the rootfs in a path accessible by root, such as the default `/opt/felix86/rootfs`. Installing the rootfs in the home directory may lead to problems.

After installation you can run 32-bit and 64-bit x86 apps inside the rootfs, or outside the rootfs if the directory is trusted.

Consult the [usage guide](./usage-guide.md) for further info.
