---
layout: post
title: Linux Kernel Debugging and Development with QEMU
subtitle: KVM/QEMU Series - Chapter 2
---

We can debug the kernel with the help of QEMU, to avoid loosing work environment.

Always better to build latest QEMU to get new feature for debugging NVMe driver.
Here it goes....

# Enable System Support #

## Check KVM Support ##

```
$ egrep -c '(vmx|svm)' /proc/cpuinfo
16

$ sudo apt install cpu-checker
$ kvm-ok
INFO: /dev/kvm exists
KVM acceleration can be used

$ lsmod | grep -i kvm
kvm_amd               147456  0
kvm                   987136  1 kvm_amd
```

If "egrep" output is more than zero and KVM module is not installed in the kernel by default, then it is good to enable and build the new kernel.

```
CONFIG_VIRTUALIZATION
CONFIG_HAVE_KVM
```

## Check IOMMU support ##

```
$ dmesg | grep AMD-Vi
[    0.215003] AMD-Vi: ivrs, add hid:AMDI0020, uid:\_SB.FUR0, rdevid:160
[    0.215005] AMD-Vi: ivrs, add hid:AMDI0020, uid:\_SB.FUR1, rdevid:160
[    0.215006] AMD-Vi: ivrs, add hid:AMDI0020, uid:\_SB.FUR2, rdevid:160
[    0.215006] AMD-Vi: ivrs, add hid:AMDI0020, uid:\_SB.FUR3, rdevid:160
[    0.454664] pci 0000:00:00.2: AMD-Vi: IOMMU performance counters supported
[    0.455395] pci 0000:00:00.2: AMD-Vi: Found IOMMU cap 0x40
[    0.455396] AMD-Vi: Extended features (0x206d73ef22254ade): PPR X2APIC NX GT IA GA PC GA_vAPIC
[    0.455401] AMD-Vi: Interrupt remapping enabled
[    0.455402] AMD-Vi: Virtual APIC enabled
[    0.455402] AMD-Vi: X2APIC enabled
[    1.545239] AMD-Vi: AMD IOMMUv2 loaded and initialized
```

If not, enable GRUB_CMDLINE_LINUX="iommu=pt amd_iommu=on" in /etc/default/grub, and then

```
$ sudo update-grub
```

and, then reboot the system.

Now, we are good go with QEMU installation.

# QEMU Installation #

## Install the required packages ##
```
$ sudo apt install make build-essential exuberant-ctags cscope git git-email
$ sudo apt install libglib2.0-dev libfdt-dev libpixman-1-dev zlib1g-dev ninja-build cpu-checker
$ sudo apt install libaio-dev libbz2-dev libcap-ng-dev libcurl4-gnutls-dev libibverbs-dev libncurses5-dev libnuma-dev librbd-dev librdmacm-dev libsasl2-dev libseccomp-dev libsnappy-dev libssh-dev liblzo2-dev virt-install
```

Create /etc/qemu/bridge.conf and add "allow virbr0" without quotes.

## Download ##
```
$ git clone https://gitlab.com/qemu-project/qemu.git
$ cd qemu
$ git checkout -b stable-v7.0.0 v7.0.0
$ git submodule init
$ git submodule update --recursive
```

## Build ##

For x86_64 system:
```
$ ./configure --target-list=x86_64-softmmu --with-git-submodules=validate --with-git='tsocks git' --enable-debug
$ make
$ sudo make install
```

For full build
```
$ ./configure --with-git-submodules=validate --with-git='tsocks git' --enable-debug
$ make
$ sudo make install
```

# Create Virtual Machine #

Create rootfs image and install file system.
```
$ qemu-img create rootfs.img 10G
$ mkfs.ext4 rootfs.img
```

Mount rootfs image to install Debian operating system.
```
$ sudo mount -o loop rootfs.img /mnt/
$ sudo debootstrap bullseye /mnt/
```

Now, create root password.
```
$ sudo chroot /mnt /usr/bin/passwd
```

Now, we are ready to start the Virtual Machine. However, the kernel image is still missing.

# Build custom kernel #

Download and build the kernel.
```
$ git clone https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git
$ cd linux
$ make x86_64_defconfig
$ make kvm_guest.config
$ make -j 16 BUILD_DIR=build
$ make modules BUILD_DIR=build
$ make install BUILD_DIR=build INSTALL_MOD_PATH=modules
```

Copy the kernel image and modules to specified directory to use for booting the virtual machine.
```
$ cp build/x86_64/arch/x86/boot/bzImage ../test_kernel/
```

Copy "modules" to specified directory.
```
$ sudo mkdir /mnt/lib/modules
$ sudo cp -ar build/modules/lib/modules/5.19.0-rc6-qemu+ /mnt/lib/modules/
$ sync
```

Unmount the rootfs image.
```
$ sudo umount /mnt
```

# Start the Virtual Machine #

Now, we can start the Virtual Machine with our own kernel image.
```
$ sudo qemu-system-x86_64 \
		-kernel test_kernel/bzImage --enable-kvm -m 4G -smp 4 -nographic \
		-hda rootfs.img -append "root=/dev/sda rw console=ttyS0" \
		-netdev bridge,id=hostnet0,br=virbr0,helper=/usr/lib/qemu/qemu-bridge-helper \
		-device virtio-net-pci,netdev=hostnet0,mac=52:54:00:6a:40:f8
```

In first time boot, we see that network is not enabled and user is still root.

For connecting to network, in QEMU guest machine:
```
$ vim /etc/network/interfaces
$ ip a
```
Add interface:
```
auto <eth0>
iface <eth0> inet dhcp
```

Update the QEMU guest and add new user.
Install openssh for connecting to the guest using SSH.
```
$ apt-get update
$ apt-get upgrade
$ adduser kernel-tester
$ usermod -aG sudo kernel-tester
$ apt-get install vim openssh-server
```

Reboot the Virtual Machine to apply new configurations.

We can enable different kernel configuration for debugging the kernel.
Enjoy!
