---
title: AMDGPU installation guide
---

Current RISC-V distros don't usually come with AMDGPU driver enabled and may need custom patches to work.

**Make sure to backup any important data that exists on your RISC-V boards.**

## SpacemiT K1

Bianbu wiki has [a guide for getting an old AMD HD 7350 GPU](https://bianbu.spacemit.com/en/development/r600) to work. This reply in Milk-V wiki has [a guide for getting an AMD RX 550](https://community.milkv.io/t/amd-radeon-rx550-gpu-not-working/2869/23) to work.

Both were confirmed to work.

## SpacemiT K3

You will need an M.2 M-Key riser. 

Starting with https://archive.spacemit.com/image/k3/version/bianbu/v4.0/ as a base, we need to install a custom kernel.

Make sure to finish the initial distro setup and connect to Wi-Fi, as we are going to use ssh.

Take note of the board local IP.

On your x86 system, run the following commands:
```bash
git clone https://github.com/felix86-emu/k3-amdgpu --depth 1 --branch k3-amdgpu k3-amdgpu
cd k3-amdgpu
export CROSS_COMPILE=riscv64-linux-gnu-
export ARCH=riscv
export KVER="6.18.3+"
make k3_bianbu_defconfig
scripts/config --enable CONFIG_DRM_AMDGPU
scripts/config --set-val CONFIG_DRM_AMDGPU_SI y
scripts/config --set-val CONFIG_DRM_AMDGPU_CIK y
make olddefconfig
make -j$(nproc) Image.gz modules dtbs
make -j$(nproc) INSTALL_MOD_PATH=./staging modules_install
make -j$(nproc) INSTALL_DTBS_PATH=./staging/dtbs dtbs_install
cp ./arch/riscv/boot/Image.gz ./staging/vmlinuz-${KVER} 
cp .config ./staging/config-${KVER}
```

Then copy over to your board. I used rsync for this:
```bash
rsync -avP "./staging/"  offtkp@192.168.1.62:~/staging
```

Now on your remote system run the following
```bash
export KVER="6.18.3+"
sudo mv ~/staging/vmlinuz-${KVER} /boot/vmlinuz-${KVER}
sudo chown root:root /boot/vmlinuz-${KVER}
sudo mv ~/staging/lib/modules/${KVER} /lib/modules/${KVER}
sudo chown -R root:root /lib/modules/${KVER}
sudo depmod -a ${KVER}
sudo cp ~/staging/config-${KVER} /boot/config-${KVER}
sudo chown -R root:root /boot/config-${KVER}
sudo mv ~/staging/dtbs/spacemit /boot/spacemit/${KVER}
sudo chown -R root:root /boot/spacemit/${KVER}
```

Edit `/boot/env_k3.txt` with a text editor to replace `6.18.3-generic` to `6.18.3+`

Here's how it should look like:
```
knl_name=vmlinuz-6.18.3+
ramdisk_name=initrd.img-6.18.3+
dtb_dir=spacemit/6.18.3+
ramdisk_addr=0x130000000
loglevel=8
commonargs=setenv bootargs plymouth.prefer-fbcon plymouth.ignore-serial-consoles splash
```

Add Ubuntu sources by creating a file `/etc/apt/sources.list.d/ubuntu.sources` with this content:
```
Types: deb
URIs: http://ports.ubuntu.com/ubuntu-ports
Suites: noble noble-updates noble-backports noble-security
Components: main restricted universe multiverse
Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg
```

Backup your Bianbu sources and install firmware, and comment out the Mesa override in `/etc/environment`
```bash
sudo mv /etc/apt/sources.list.d/bianbu.sources /etc/apt/sources.list.d/bianbu.bak
sudo apt update
sudo apt install linux-firmware
sudo apt upgrade
sudo sed -i 's/^MESA_LOADER/#MESA_LOADER/' /etc/environment
```

Finally, update the initramfs:
```shell
sudo update-initramfs -c -k ${KVER}
```

Connect your GPU through the riser (make sure to use the right M-Key port), mine was an AMD RX 550, connect the HDMI cable to the GPU and shutdown. Power on the GPU and then power on the system.

The system should start normally. Confirm the GPU is used with `glxgears -info` showing your GPU name.