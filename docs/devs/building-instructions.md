---
title: Building instructions
---

The installation script will properly install felix86 and a rootfs for you. These instructions are for developers and users who want to compile felix86 manually.

## Building

On a RISC-V machine, you can build same as any other CMake project:
```sh
cmake -B build
cmake --build build -j$(nproc)
```

On an x86 machine, install the `riscv64-linux-gnu-gcc` cross-compiler and set the toolchain file:
```sh
cmake -B build -DCMAKE_TOOLCHAIN_FILE=./riscv.cmake
cmake --build build -j$(nproc)
```

By default, this build will be `RelWithDebInfo` to assist with development.

### binfmt_misc

felix86 can work without being registered to binfmt_misc, but AppImage files and setuid applications like `sudo` won't work.

To register to binfmt_misc run the following command:
```sh
sudo -E ./build/felix86 -b
```

!!! warning

    Since we use the `F` flag in binfmt_misc, each new compilation needs a new binfmt_misc registration. As such, always run `sudo -E ./build/felix86 -b` after compiling to make sure the binfmt_misc version gets updated.

## Tests

You can build the tests by setting the `BUILD_TESTS` flag during configuration:
```bash
cmake -B build -DBUILD_TESTS=ON
```

These include the single instruction tests from [FEX-Emu](https://github.com/FEX-Emu/FEX/tree/main/unittests/ASM), some unit tests for syscalls and some binary tests.

The felix86 CI runs more binary tests than that. If you want to run the binary tests, clone the binary_tests repo:
```bash
git clone https://github.com/felix86-emu/binary_tests
cd binary_tests
```

You can use the `install_tests.sh` script to download the binary tests:
```sh
./install_tests.sh $(pwd)/binaries
```

Then to run the tests, temporarily disable binfmt_misc usage and use the `run_tests.sh` script:
```sh
export FELIX86_BINFMT_MISC_INSTALLED=0
./run_tests.sh /path/to/felix86 $(pwd)/binaries
```

## Rootfs

If you don't want to install a rootfs [using the installation script](../users/installation-guide.md), you can build one using Docker.

First, you need a Dockerfile:
```dockerfile title="Dockerfile"
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

COPY Run.sh /tmp/Run.sh

RUN bash /tmp/Run.sh

WORKDIR /root

CMD ["/bin/bash"]
```

You also need a script to run inside the container:
```bash title="Run.sh"
#!/bin/bash
apt update
apt upgrade -y

# Install whatever packages you want your rootfs to have
# apt install hello

# Create a .tar.gz inside the container while excluding the tempfs directories
touch archive.tar.gz
tar  --exclude=archive.tar.gz --exclude=./media --exclude=./mnt --exclude=./root --exclude=./srv --exclude=./boot --exclude=./home --exclude=./run --exclude=./proc --exclude=./sys --exclude=./dev --exclude=./tmp --exclude=./.dockerenv -czf archive.tar.gz .
```

Now create your rootfs:
```bash
sudo docker build -t rootfs .
sudo docker create --name rootfs rootfs
sudo docker cp rootfs:/archive.tar.gz ./rootfs.tar.gz
sudo docker rm rootfs
sudo docker rmi rootfs
```

This will create an archived and minimal x86-64 Ubuntu rootfs.

Extract it using `tar -xzf rootfs.tar.gz -C /path/to/rootfs` and set it using `felix86 -s /absolute/path/to/rootfs`.

You can install packages inside the rootfs through felix86, or during rootfs creation by changing the `Run.sh` script.

For an idea of what sort of packages you might need, check out [the rootfs generator repository](https://github.com/felix86-emu/rootfs).

#### Important files

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

Copy these to the rootfs while retaining the host permissions like so:
```bash
sudo cp -rp /etc/mtab $ROOTFS/etc/mtab
```

