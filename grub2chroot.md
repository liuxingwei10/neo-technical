## Installing GRUB 2 from a chroot environment ##

**DO NOT USE ANY GRAPHICAL UTILITIES FOR THIS PROCESS!**

1a.) Insert in a Linux live CD of any kind and boot into it.

2a.) Determine your linux install partition of which you would like to mount:

```
fdisk -l
```

3a.) Create a directory on the live CD to mount the linux filesystem in:

```
mkdir -p /mnt/harddrive
```

4a.) When you used fdisk to list all partitions and found the one you would like to mount (i.e. /dev/sda3) you must now mount it to the folder and additional filesystems to the directory. For this example we use /dev/sda3 as our Linux hard drive partition.

```
mount /dev/sda3 /mnt/harddrive
mount --bind /dev /mnt/harddrive/dev
mount --bind /proc /mnt/harddrive/proc
mount --bind /sys /mnt/harddrive/sys
```

5a.) Chroot into the environment:

```
chroot /mnt/harddrive /bin/bash
source /etc/profile
ldconfig
```

6a.) Install GRUB 2 to the MBR:

Please note: We use /dev/sda as our hard drive for this example.

```
grub2-install --no-floppy /dev/sda
```

1b.) If you would also like to update the grub.cfg file, you may do so now.

```
grub2-mkconfig -o /boot/grub2/grub.cfg
```

7a.) Exit the chroot environment:

```
exit
```

8a.) Sync all disks and unmount all filesystems:

Please note: We are still using /dev/sda3 as our example.

```
sleep 5 && sync
umount -l /mnt/harddrive/{sys,proc,dev}
umount /dev/sda3
```

9a.) Reboot the system and eject the live CD.

```
shutdown -r now
```