# Cubieboard2 from scratch #

## Introduction ##

What we will be doing is building everything the board needs to get up and running, this includes compiling the bootloader (U-Boot) making an optimized cross compiler for our specific platform, building the Linux kernel, and whatever else.

### Prerequisites ###

You will need the following packages installed on your host system before proceeding:

  * mercurial (hg)
  * git (git-core)
  * CLooG (Chunky Loop Generator) Parma + PPL or ISL
  * make 3.81 or 3.82 recommended (4.0 is known to have issues)
  * gcc 4.8.3 or 4.7.4 recommended
  * g++ 4.8.3 or 4.7.4 recommended
  * binutils 2.22 or later recommended
  * mpfr 3.1.2 recommended
  * gmp (libgmp) 5.1.3 recommended
  * GNU mpc (libmpc) 1.0.2 recommended
  * automake 1.13.4 recommended (1.14.1 is known to have issues)
  * autoconf 2.69 recommended

**Please note: If you have any U-Boot utilities installed on your host system such as mkimage, you should remove it now to prevent issues later on**

**Please note: Commands with a starting $ implies a standard user command.**

**Commands with a # are meant to be run as root or using sudo.**

## Building the cross compiler ##

1a.) Download crosstool-ng and config file for Cubieboard2:

http://crosstool-ng.org/#using_the_latest_development_stuff

For "./ct-ng menuconfig" I have made a crosstool-ng config file that you can load via the Kconfig interface (Load an Alternate Configuration File) that you can download here:

https://neo-technical.googlecode.com/svn/trunk/crosstool/crosstool-ng.config

**Be sure that after you download and save it, to load the file and save to .config**

2a.) Compile crosstool-ng:

After loading the crosstool-ng config file, run the following command:

```
$ ct-ng build
```

Your cross compiler should now be in in your ~/x-tools folder. Because I do a lot of cross compiling, I have modified the PATH variable in my ~/.bashrc so I no longer have to run export PATH="$PATH:/home/whatever/x-tools/arm-unknown-linux-gnueabihf/bin"

3a.) Now that you have your cross compiler in your PATH (however method you decide to do that) it is time to start fetching the required sources for the cubieboard2:

## Fetch required sources ##

```
$ cd ~/ && mkdir -p cubieboard-dev && cd cubieboard-dev
$ git clone https://github.com/linux-sunxi/u-boot-sunxi.git
$ git clone https://github.com/linux-sunxi/linux-sunxi.git
$ git clone git://github.com/linux-sunxi/sunxi-tools.git
$ git clone git://github.com/linux-sunxi/sunxi-boards.git
```

## Begin building for Cubieboard2 ##

4a.) Compile U-Boot:

**PLEASE NOTE: I have not yet messed with the Cubieboard2 enough to get it to boot off of an external SATA drive!**

```
$ cd u-boot-sunxi
$ make CROSS_COMPILE=arm-unknown-linux-gnueabihf- Cubieboard2_config
$ make CROSS_COMPILE=arm-unknown-linux-gnueabihf-
$ cd ..
```

5a.) Export the U-Boot tools directory to your PATH environment, for example:

```
$ export PATH="$PATH:/home/your user/cubieboard-dev/u-boot-sunxi/tools"
```

Your PATH environment variable should now have both your cross compiler and your U-Boot tools directories, this allows for the kernel to find mkimage to generate a U-Bootable kernel image (uImage)

6a.) Compile fex2bin:

```
$ cd sunxi-tools
$ make fex2bin
$ cd ..
```

7a.) Compile script.bin:

```
$ cd sunxi-boards
$ ../sunxi-tools/fex2bin sys_config/a20/cubieboard2.fex script.bin
$ cd ..
```

8a.) Compile the Linux kernel:

For a good example kernel config, optimized for the Cubieboard2, you can download one that I made here:

https://neo-technical.googlecode.com/svn/trunk/cubieboard2/cubieboard2.defconfig

You can load it either by copying it to .config in your kernel tree or loading it via the ncurses make menuconfig interface. Note that this kernel config is just a base config to get the board going, along with some other odds and ends.

```
$ cd linux-sunxi
$ ARCH="arm" CROSS_COMPILE="arm-unknown-linux-gnueabihf-" make menuconfig
$ ARCH="arm" CROSS_COMPILE="arm-unknown-linux-gnueabihf-" make uImage modules
$ mkdir -p output
$ ARCH="arm" CROSS_COMPILE="arm-unknown-linux-gnueabihf-" INSTALL_MOD_PATH="output" make modules_install
$ cd ..
```

## Transferring it to your bootable media ##

Putting it all to use is more or less not very straight forward. Ideally it would be best to boot the Cubieboard2 off of an external SATA hard drive instead of an SD card, USB flash drive or similar, to avoid the flash becoming worn out. You'll also run into slower transfer rates than a real external hard drive.

I've only tested the Cubieboard2 with an external SD card and here is how I did it, but honestly, don't use an SD card with this board. I'm writing it down here so it's easy for other people to BUILD upon, not simply copy and wish for high performance and high speed reads and writes. If you want faster r/w speeds and don't want to ruin your flash devices, **DO NOT DO THIS!**

**YOU HAVE BEEN WARNED!**

**Please note: This part of the wiki assumes you know what you are doing and plan on either building your own root filesystem from scratch (gentoo is awesome for this) OR want to simply use your own kernel and bootloader that you just built by following the steps above, with the use of another person's prebuilt root filesystem. If you plan on using someone else's prebuilt root filesystem, be sure to replace their kernel and kernel modules with your own (check the root of their filesystem, the /boot directory, and /lib/modules for this.)**

### Generating required stuff to boot ###

9a.) Write your boot.cmd file in your favorite text editor:

Depending on how you have your SD card partitioned (mine is just one big ext4 partition to the maximum size of the SD card) there are a few ways to do this. If you're sticking everything on one partition such as your kernel and your boot directory, you want your boot.cmd file to look like this:

```
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p1 rootwait rw
ext2load mmc 0 0x43000000 boot/script.bin
ext2load mmc 0 0x48000000 uImage boot/uImage
bootm 0x48000000
```

**Please note: The above boot.cmd file assumes you will place your script.bin and kernel binary in the /boot directory of your root filesystem, while leaving boot.scr and boot.cmd in the root directory of your filesystem. At your option, you may place boot.cmd and boot.scr in the /boot directory. In such case, modify your boot.cmd accordingly:**

```
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p1 rootwait rw
ext2load mmc 0 0x43000000 script.bin
ext2load mmc 0 0x48000000 uImage
bootm 0x48000000
```

**For AHCI SATA:**

```
console=ttyS0,115200 root=/dev/sda1 rootwait rw
ext4load scsi 0 0x43000000 script.bin
ext4load scsi 0 0x48000000 uImage
bootm 0x48000000
```

10a.) Generate a boot.scr file:

```
$ mkimage -C none -A arm -T script -d boot.cmd boot.scr
```

### Preparing your SD card ###

11a.) Wipe your SD card with zeros:

```
# dd if=/dev/zero of=/dev/YourSDCard bs=1024 count=100000
```

12a.) Install the bootloader to the SD card:

```
$ cd u-boot-sunxi
# dd if=spl/sunxi-spl.bin of=/dev/YourSDCard bs=1024 seek=8
# dd if=u-boot.img of=/dev/YourSDCard bs=1024 seek=40
```

13a.) Partition your SD card:

**Please note: Do not use cfdisk, gparted, parted, or any other utilities because a lot of them suck, for example cfdisk wants to use terrible sectors that lead to poor performance such as sector 63 instead of 2048, 4096, or at least some sort of a multiple of the physical sector size of the drive being used. For example, my 2TB hard drive has an optimal I/O size of 4096 and a physical sector size of 4096, yet cfdisk believes a sector size of 63 makes more sense.**

```
# fdisk /dev/YourSDCard
```

Create a new Linux partition (ID 83) at the start of the SD card to the maximum allowable size and be sure it starts on a physical sector boundary.

14a.) Format your SD card:

```
# mkfs.ext4 /dev/YourSDCard1
```


15a.) Mount it:

```
# mkdir -p /mnt/sd
# mount /dev/YourSDCard1 /mnt/sd
```

16a.) Move the required files onto your bootable media:

**Please note: Depending on how you wrote your boot.cmd file, the locations of where you should put your boot.scr, boot.cmd and script.bin may differ.**

```
# cp -prv linux-sunxi/arch/arm/boot/uImage /mnt/sd/boot/uImage
# cp -prv linux-sunxi/output/lib/modules/* /mnt/sd/lib/modules/
# cp -prv boot.cmd /mnt/sd/
# cp -prv boot.scr /mnt/sd/
# cp -prv sunxi-boards/script.bin /mnt/sd/
```

17a.) Unmount your SD card:

```
# sleep 5 && sync
# umount /mnt/sd
```

Make sure it's unmounted:

```
# umount /dev/YourSDCard1
```

## Finishing up ##

All you need now is a filesystem! I recommend the following:

http://mirrors.rit.edu/gentoo/releases/arm/autobuilds/current-stage3-armv7a_hardfp/

Enjoy your Cubieboard2!

## Example layout ##

ls /mnt/sd:

```
bin   boot.cmd  dev  home  lost+found  mnt  proc  run   srv  tmp  var
boot  boot.scr  etc  lib   media       opt  root  sbin  sys  usr
```

ls /mnt/sd/boot:

```
script.bin  uImage
```

cat /mnt/sd/boot.cmd:

```
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p1 rootwait rw
ext2load mmc 0 0x43000000 boot/script.bin
ext2load mmc 0 0x48000000 uImage boot/uImage
bootm 0x48000000
```

fdisk -l:

```
Disk /dev/sdc: 14.9 GiB, 16009658368 bytes, 31268864 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x00000000

Device    Boot Start       End   Blocks  Id System
/dev/sdc1       2048  31268863 15633408  83 Linux
```

echo $PATH:

```
/usr/local/bin:/usr/bin:/bin:/opt/bin:/usr/x86_64-pc-linux-gnu/gcc-bin/4.8.3:/home/ntu/x-tools/arm-unknown-linux-gnueabihf/bin:/home/ntu/devel/cubie/u-boot-sunxi/tools
```

cat ~/.bashrc:

```
export PATH="$PATH:/home/ntu/x-tools/arm-unknown-linux-gnueabihf/bin:/home/ntu/devel/cubie/u-boot-sunxi/tools"
```

# TODO #

  * Hard drive support (Documentation and testing)