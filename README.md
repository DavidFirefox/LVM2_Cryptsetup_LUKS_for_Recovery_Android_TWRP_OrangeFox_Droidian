LVM2_Cryptsetup_LUKS_for_Recovery_Android_TWRP_OrangeFox_Droidian
===============
LVM2 and CRYPTSETUP BINARY for use on ANDROID RECOVERY : TWRP and ORANGEFOX (and building guide)

Nowdays phones are high-performance, so they can operate a real computer Linux OS (in replacement of Android). So we can install Droidian (a phone version of Debian), UBTouch (based on Ubuntu), PostmarketOS, Sailfish ...

Droidian (due to the max number of partitions in GPT table, the fact of certain phone manufacturer uses all of possible partitions, and probably Droidian team found other advantages.) use LVM partitioning system.\
LVM is a way to make many logical volumes on only one physical partition, with a lot of advanced functions.

So when we use Droidian on a phone, the TWRP and OrangeFox recovery can read the Droidian partition that contain system and user data.\
So we can't read it in recovery, and we cant backup it. If Droidian crash we lost all data's and system data. (We can't physically extract the memory of the phone, so we need a recovery with LVM !)

For some users we are finding a way to easily add LVM to existing recovery. (Old LVM binary exist in 2013)

Some users use Cryptsetup/LUCK for encrypting the disk (it seems it can be use for Android advanced user to.), Droidian use this format for encryption, so we need to add the Cryptsetup support on recovery.


So the goal is to have LVM2 and Cryptsetup/LUCK on Recovery. 

LVM2
==============

What's in the tree:

## lvm-bin/
Prebuilt LVM2 binary -> Tested in OrangeFox, Works !\
~~(support scripts, and sample lvm.conf file.)~~ -> TO DO

This is a lvm2-2_03_30 arm64 version prebuilt and optymisez for armv8. It is built for Android (Release : 2025/01/14).\
First thanks in advance for steven676's instruction (Address at https://github.com/steven676/android-lvm-mod/blob/master/HOWTO-BUILD).  
And thanks in advance for sillaboy's instruction (Address at https://github.com/sillaboy/Lvm_for_Android/blob/master/README-HowToBuild).

### Use LVM2 on Recovery
You just have to copy it on recovery (/system/bin) and execute it !
```
chmod +x lvm
lvm vgscan --mknodes # Show and permit use of Volume Group using LVM2 (you will see : "droidian" for Droidian installation)
lvm vgchange -aly # Show the number (and permit use) of Logical Volumes (you will see : 3 for Droidian)
```

Now you can mount you LVM Logical Volume, for example for Droidian :
```
mkdir /mnt/droidian
mount /dev/droidian/droidian-rootfs /mnt/droidian/
```

Please umount before leaving Recovery !
```
umount /mnt/droidian/
```

### /!\ Mounting Logical Volume as /data (in test) /!\
/data already have few folders but is not link to the userdata Drodian Logical Volume, so I try in OrangeFox to mount Droidian-LV on data.\
It works and it permits to directly backup and restore Droidian system and data. And to backup all partitions (boot, super, ...) in the internal SD.\
You have to make /data/media folder for using the Droidian Logical Volume as internal SD.
```
mount /dev/droidian/droidian-rootfs /data
```

The first time you have to make the /data/media directory.
```
mkdir /data/media
```

Please umount before leaving Recovery !
```
umount /data
```

NOT MODIFY
==============

## lvm-src/
LVM2 source code. (If this directory is empty, do
git submodule init
git submodule update
to fetch the source.)

## devices/
Sample lvm.conf and init.rc scripts for particular devices.

## HOWTO-MOD
Information on how to modify a ROM and recovery for a device to use
LVM2.

## HOWTO-BUILD
Information on how to build the LVM2 source for use on Android.

LICENSE
Licensing information for the components.


Questions or comments can be addressed to 


TO DO
==============
CRYPTSETUP/LUCK

