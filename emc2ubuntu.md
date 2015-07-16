# This page is out of date and no longer maintained #

## Building EMC 2.4.5 on Ubuntu with RTAI MAGMA from source ##

**WARNING! DO NOT USE THE CLOSED-SOURCE NVIDIA DRIVER BINARY BLOBS!**

In order to start compiling, you must install several packages first.


1a.) Update, sync, and upgrade everything:

```
sudo apt-get update && sudo apt-get dist-upgrade
```

2a.) Install the following packages via pacman:

```
sudo apt-get install cvs build-essential fakeroot debhelper libpth-dev libgtk2.0-dev kernel-wedge tcl8.4-dev tk8.4-dev bwidget python2.5-dev python-tk python-dev libglu1-mesa-dev libgtk2.0-dev libgnomeprintui2.2-dev libncurses5-dev libxaw7-dev gettext libreadline5-dev lyx texlive-extra-utils imagemagick texinfo groff qt3-dev-tools
```

3a.) Fetch kernel source:

```
cd /usr/src && sudo wget ftp://kernel.org/pub/linux/kernel/v2.6/linux-2.6.35.7.tar.bz2 && sudo tar xjf linux-2.6.35.7.tar.bz2 && sudo mv linux-2.6.35.7 linux
```

4a.) Fetch and patch kernel with latest RTAI code via CVS:

```
sudo cvs -d:pserver:anonymous@cvs.gna.org:/cvs/rtai co magma && cd linux && sudo patch -p1 < ../magma/base/arch/x86/patches/hal-linux-2.6.35.7-x86-2.8-00.patch
```

5a.) Fetch the kernel config:

The following is for a 32-bit x86 based machine. For those looking for the AMD64 version, follow step 1b.

```
sudo wget http://neo-technical.googlecode.com/svn/trunk/x86/PKGBUILDS/rtai-kernel/rtai-kernel.config && sudo mv -vi rtai-kernel.config .config
```

Editing the kernel config is STRONGLY NOT RECOMMENDED! This kernel config file is not the best but should support all if not most x86 machines.

1b.)

Specifically for AMD64 (64-bit) machines:

```
sudo wget http://neo-technical.googlecode.com/svn/trunk/x86/PKGBUILDS/rtai-kernel/rtai-kernel64.config && sudo mv -vi rtai-kernel64.config .config
```

6a.) Compile the kernel, and modules:

```
sudo make clean && sudo make && sudo make modules_install
```

7a.) Install the kernel:

```
sudo cp -vi arch/x86/boot/bzImage /boot/vmlinuz-rtai
```

8a.) Modify menu.lst or grub.cfg to use the new kernel. Each set of configs might need to be modified by hand due to different locations of hard drives and installation.

```
sudo nano /boot/grub/menu.lst
```

For grub 0.9X with no seperate boot partition:

```
timeout=5
default=0

title RTAI Kernel
root (hd0,0)
kernel /boot/vmlinuz-rtai root=/dev/sda1 ro
```

For grub 0.9X with a seperate boot partition:

```
timeout=5
default=0

title RTAI Kernel
root (hd0,0)
kernel /vmlinuz-rtai root=/dev/sda1 ro
```

For grub 1.9X (GRUB2) with no seperate boot partiton:

```
set timeout=5
set default=0

menuentry "RTAI kernel" {
set root=(hd0,1)
linux /boot/vmlinuz-rtai root=/dev/sda1 ro
```

For grub 1.9X (GRUB2) with seperate boot partition:

```
set timeout=5
set default=0

menuentry "RTAI kernel" {
set root=(hd0,1)
linux /vmlinuz-rtai root=/dev/sda1 ro
}
```

I will NOT cover LILO. If you are using lilo, please remove it and install GRUB (0.97 or 1.9X / 2)

9a.) Reboot your computer

```
sudo reboot
```

PLEASE NOTE: If you cannot boot the kernel due to a kernel panic, read the error carefully and edit grub.cfg or menu.lst accordingly to one of the avaliable partitions if given.

10a.) Clean up RTAI and configure:

```
cd && sudo rm -rf /usr/src/magma && cvs -d:pserver:anonymous@cvs.gna.org:/cvs/rtai co magma && cd magma && make menuconfig
```

It is mandatory to turn on RTAI Math support.

To do so, go to Base System > Other features > (Y) Mathfuns support in kernel > (Y) C99 standard support

If you have an SMP machine (multi processor PC) then you must configure RTAI for that specific number of processing cores. To find out how many cores your CPU has, please run:

```
cat /proc/cpuinfo | grep processor | wc -l
```

Now go into Machine in the root of make menuconfig, and hit "Enter" where it says "Number of CPUs (SMP-only)" and replace the number with the number of cores which were detected with the above command.

11a.) Compile it:

```
make
```

If you get any errors (NOT WARNINGS) please post the error/errors as a commment.

12a.) If you get no compiling errors, install it:

```
sudo make install && cd ..
```

13a.) Edit /etc/security/limits.conf along with udev rules and add in the following lines to the end of the file:

/etc/security/limits.conf:

```
*        soft    memlock        20480
*        hard    memlock        20480
```

/etc/udev/rules.d/99-rtai.rules:

```
# RTAI:rtai_shm
KERNEL=="rtai_shm", MODE="0666"

# RTAI:rtai_fifos
KERNEL=="rtf[0-9]*", MODE="0666"
```

14a.) You MUST REBOOT your entire system now otherwise EMC will complain about not having memory access to /dev/rtai\_shm

```
sudo reboot
```

15a.) Fetch, unpack, patch, and configure emc:

```
wget http://downloads.sourceforge.net/project/emc/emc2/2.4.x/emc2_2.4.5/emc2_2.4.5.tar.gz && tar zxf emc2_2.4.5.tar.gz && cd emc2-2.4.5/src/hal/drivers && sed -i 's/pci_find_device/pci_get_device/g' *.c && cd ../.. && ./configure --prefix=/usr
```

16a.) If you don't get any configure errors, compile it:

```
make
```

17a.) If you get no compiling errors, do this to install it:

```
sudo make install
```

18a.) Run emc:

```
emc
```

If you rather launch emc from the desktop, you can either make a symlink to it on your desktop or run 'sudo ldconfig' and see if that does the trick.

To use a symlink:

```
sudo ln -s /usr/bin/emc ~/Desktop/emc
```

Double-clicking on the icon should launch emc.