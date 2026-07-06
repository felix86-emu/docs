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
        sudo pacman -S --needed base-devel cmake pkgconf libx11 mesa vulkan-devel wayland nasm
        ```

    === "Ubuntu/Debian"
        ```
        sudo apt install build-essential cmake pkg-config libx11-dev libgl1-mesa-dev libglx-dev libvulkan-dev libwayland-dev nasm
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

    If compiling on RISC-V hardware, you'll need x86-64 binutils to build the x86-64 VDSO.

    If your package manager provides x86-64 binutils, simply install it. Otherwise, you can follow the steps below. This is a one-time installation:
    ```sh
    sudo apt update
    sudo apt install -y build-essential bison flex texinfo libgmp-dev libmpfr-dev libmpc-dev libisl-dev wget

    mkdir -p /tmp/binutils
    cd /tmp/binutils
    wget https://ftp.gnu.org/gnu/binutils/binutils-2.42.tar.xz
    tar xf binutils-2.42.tar.xz

    mkdir -p /tmp/binutils/build
    cd /tmp/binutils/build

    ../binutils-2.42/configure \
        --target=x86_64-linux-gnu \
        --prefix=/opt/x86_64-linux-gnu \
        --with-sysroot \
        --disable-nls \
        --disable-werror

    make -j$(nproc)
    sudo make install
    ```

    If you want to manually build the thunk libraries you'll need an x86 compiler too. Check out the build.yml workflow to get an idea on how to build them, or install them once using the installation script.


    ## Configuring
    If you're building on a RISC-V machine, make sure to set the path to the x86-64 binutils:
    ```
    cmake -B build -DFELIX86_X86_CROSS="/opt/x86_64-linux-gnu/bin/x86_64-linux-gnu-"
    ```
    If you're cross-compiling on x86 hardware, you just need to use the RISC-V CMake toolchain and host binutils can be used.
    ```
    cmake -B build -DCMAKE_TOOLCHAIN_FILE=riscv.cmake
    ```

    ## Building
    Just like any other CMake project.
    ```
    cmake --build build -j$(nproc)
    ```

    ## binfmt_misc installation
    You should also install in binfmt_misc with `--binfmt-misc`:
    ```
    sudo ./build/felix86 --binfmt-misc
    ```
    Each new compilation needs a reinstallation in binfmt_misc.

    Installing with binfmt-misc-setuid allows running privileged applications, such as `sudo`, through the emulator.

    This has some security implications, see [this troubleshooting entry](./troubleshooting.md#privileged-executables-dont-work) to learn more.
    ```
    sudo ./build/felix86 --binfmt-misc-setuid
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

    Create the directories that will get mounted inside the rootfs:
    ```
    sudo mkdir -p $ROOTFS/dev
    sudo mkdir -p $ROOTFS/proc
    sudo mkdir -p $ROOTFS/sys
    sudo mkdir -p $ROOTFS/run
    sudo mkdir -p $ROOTFS/tmp
    sudo mkdir -p $ROOTFS/home
    ```

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

!!! tip
    Install the rootfs in a path accessible by root, such as the default `/opt/felix86/rootfs`. Installing the rootfs in the home directory may lead to problems.

Consult the [usage guide](./usage-guide.md) for further info.
