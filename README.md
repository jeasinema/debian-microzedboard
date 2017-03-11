# Debian/Ubuntu on the Zynq based board
Minimal debian filesystem for armhf and armel (arm hard floating point and arm soft floating point)

Guide to create a minimal minimal Debian filesystem

Jeasine Ma 2016

References:
-----------
Addapted to Zynq based MicroZedBoard from: 
http://www.acmesystems.it/multistrap

More information about what is armhf and armel:
https://wiki.debian.org/ArmHardFloatPort

Default features:
----------------
<ul>
<li>ssh</li>
<li>gcc</li>
<li>wget</li>
<li>rsync</li>
<li>python</li>
</ul>

Requirements:
-------------

    ~$ sudo apt-get install multistrap
    ~$ sudo apt-get install qemu
    ~$ sudo apt-get install qemu-user-static
    ~$ sudo apt-get install binfmt-support
    ~$ sudo apt-get install dpkg-cross

Create a working directory directory:

    ~$ mkdir multistrap
    ~$ cd multistrap
    ~/multistrap$

Create the root filesystem
-------------------------

When your multistrap.conf file is ready launch type:

if you are using the default linaro:

    ~/multistrap$ sudo multistrap -a armhf -f microzed.confg

At the end of this procedure the directory target-rootfs directory will contents the whole rootfs tree to be moved into the second partition of an Acme board bootable microSD.

if you are using a compiler without floating point support as the default of the xilinx SDK:

    ~/multistrap$ sudo multistrap -a armel -f microzed.conf

Configure now the Debian packages using the armel CPU emulator QEMU and chroot to create a jail where dpkg will see the target-rootfs as its root (/) directory.

    ~/multistrap$ sudo cp /usr/bin/qemu-arm-static target-rootfs/usr/bin
    ~/multistrap$ sudo mount -o bind /dev/ target-rootfs/dev/
    ~/multistrap$ sudo LC_ALL=C LANGUAGE=C LANG=C chroot target-rootfs dpkg --configure -a

At the Dash configuration promots answer no.

Do some extra configurations with this script:
    ~/multistrap$ chmod +x microzed.sh
    ~/multistrap$ sudo ./microzed.sh

Create now the target root login password:

    ~/multistrap$ sudo chroot target-rootfs passwd
    Enter new UNIX password:
    Retype new UNIX password:
    passwd: password updated successfully

List the packages installed

    ~/multistrap$ sudo chroot target-rootfs dpkg --get-selections | more

Ugrade the system and install new packages

    ~/multistrap$ sudo chroot target-rootfs dpkg --get-selections

At this point you can upgrade Debian to the latest version by typing:

    ~/multistrap$ sudo LC_ALL=C LANGUAGE=C LANG=C chroot target-rootfs apt-get update
    ~/multistrap$ sudo LC_ALL=C LANGUAGE=C LANG=C chroot target-rootfs apt-get upgrade
    ~/multistrap$ sudo LC_ALL=C LANGUAGE=C LANG=C chroot target-rootfs apt-get clean

Copy the rootfs contents on microSD
-----------------------------------

Remove qemu from the target rootfs contents:

    ~/multistrap$ sudo rm target-rootfs/usr/bin/qemu-arm-static

then format a new microSD. You can use fdisk, gparted. 

Check https://github.com/dasGringuen/MicroZedKernel for a kernel and uboot

Mount the microSD on yourLinux PC and copy all the target-rootfs contents in the second microSD partition.
Use the lsblk command to be check where it is mounted

    ~/multistrap$ sudo rsync -axHAX --progress target-rootfs/ <path of your rootfs>

Test
-----

After booting with the sd in the microzedboard

You should be able to connect through ssh with the next command:

    ~/multistrap$ ssh root@192.168.1.103
    
If you are using the image with the avahi deamon you can connect like this:

    ~/multistrap$ ssh root@microzed.local

Check the size of each package:

    ~/root@arm# dpkg-query -W --showformat='${Installed-Size;10}\t${Package}\n' | sort -k1,1n

Check file system size:

    ~/root@arm# df -h
    ~/root@arm# du -h --max-depth=1


