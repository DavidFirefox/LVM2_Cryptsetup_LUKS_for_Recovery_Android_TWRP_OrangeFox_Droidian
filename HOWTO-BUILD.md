BUILDING LVM2 FOR USE ON ANDROID
==============

This is a lvm2-2_03_30 arm64 version prebuilt and optymisez for armv8. It is built for Android (Release : 2025/01/14).\
First thanks in advance for steven676's instruction (Address at https://github.com/steven676/android-lvm-mod/blob/master/HOWTO-BUILD).  
And thanks in advance for sillaboy's instruction (Address at https://github.com/sillaboy/Lvm_for_Android/blob/master/README-HowToBuild).

## General informations :
The LVM source was written for standard glibc-based GNU/Linux systems, and doesn't build agains Android libc.  
We could invest time and effort into porting it to Android, but it's easier to just statically link the LVM binary against glibc (we already have to use a statically linked
binary, since the boot image doesn't ship shared libraries).

To build LVM with glibc, you will need either an actual GNU/Linux ARM system with a development environment (gcc, make, and glibc header files) set up, or a GNU/Linux ARM cross-compiler environment.

For native compilation, any recent Linux distribution and any ARM device should do.  If you don't have something like a BeagleBoard or SheevaPlug, you can install a GNU/Linux chroot on any rooted Android
device.  
You'll need the development tools for your chosen distribution (install metapackage "build-essential" on Debian or Ubuntu, or package group "Development Tools" on Fedora-derived distros).
-> Today with phone on Linux like Drodian we can directly buid it on phone. Droidian have all Debian package, so it is like a Debian on computer.

For cross compilation, you'll need a Linux box, a cross-compiler toolchain for GNU/Linux ARM EABI, and (for now) the qemu-arm user mode emulator (to keep the configure script from getting confused).  
The Linaro prebuilt GCC toolchain and the CodeSourcery CodeBench Lite toolchain for ARM Linux should work; the Android NDK toolchain will not.
(Debian users can install the Emdebian cross toolchain from http://www.emdebian.org/crosstools.html).


## Here is my pc info :

I directly build it on my phone on a ARM64 Droidian 100. The binary are on lvm-bin/ optymisez for armv8. It could be build for arm32, but I think we don't have Droidian phone with arm32.

1. Debian / Droidian info :  
Droidian 100 based on Debian sid, as of 2024-11-04  
So it is equivalent of Debian 13 Trixie, instructions will be similar for Debian 12 Bookworm or Ubuntu 24.04.

2. arm gcc info  
gcc (Debian 14.2.0-8) 14.2.0

3. lvm version  
lvm2-2_03_30 or lvm2-2.03.30

I directly build on a ARM64 Debian, so I not use cross compilation.
For cross copilation howto please read : https://github.com/DavidFirefox/LVM2_Cryptsetup_LUKS_for_Recovery_Android_TWRP_OrangeFox_Droidian/blob/testing2/HOWTO-BUILD_2.02.102.old.md or https://github.com/DavidFirefox/LVM2_Cryptsetup_LUKS_for_Recovery_Android_TWRP_OrangeFox_Droidian/blob/testing2/HOWTO-BUILD_2.02.98.old


## Build LVM2 :

1. Packages need :  
sudo apt install build-essential pkg-config libblkid-dev libdevmapper-dev libz-dev uuid-dev flex bison vdo libdmraid-dev libaio-dev libaio1 libaio1t64  

I try lot of packages before finding the right parameter, so I think : linux-source is not need.  
And g++-arm-linux-gnueabi g++-arm-linux-gnueabihf are not need  
Try to install it if it not works.  
   
2. Download the LVM2 source.  
   I choose lvm2-2.03.30  
   https://github.com/lvmteam/lvm2/releases/tag/v2_03_30  
   or :  
   ftp://sourceware.org/pub/lvm2/LVM2.2.03.30.tgz  
   
   You can see all version : https://github.com/lvmteam/lvm2/tags  

3. Configure the LVM source:  
   (Instruction according to steven676's guide.)  
   I remove -mthumb on --with-optimisation it don't work, I don't know more.  
```
cd lvm-src/
./configure --prefix=/lvm --enable-static_link --disable-readline \
      --disable-selinux --with-pool=none --with-cluster=none \
      --with-confdir=/lvm/etc --with-default-run-dir=/data/lvm/run \
      --with-default-system-dir=/lvm/etc \
      --with-default-locking-dir=/data/lvm/lock \
      --with-optimisation="-Os -march=armv8-a -mtune=generic-armv8-a"
```

(Explanation: --prefix, --with-confdir, --with-default-run-dir, --with-default-system-dir,  
--with-default-locking-dir tell LVM where to find its pieces.  
--enable-static_link enables the building of a statically linked binary.  --disable-readline --disable-selinux  
--with-pool=none --with-cluster=none disable features we don't need.  
--with-optimisation changes the compiler flags to produce a smaller binary while still complying with Android's "arm64" ABI.)  


4. Build the software:   
   You can just make or make to a directory
   - <ins>Just make :</ins>
    ```
    $ make
    ```

    The resulting statically linked unstripped LVM binary will be located in tools/lvm.static in the LVM source tree,
    and an example configuration file is located in doc/example.conf.

   - <ins>Make and copy all the output to a directory :</ins>
   ```
    $ make install DESTDIR=$HOME/lvm-build
   ```
All build binary files and library are in the $HOME/lvm-build folder.
You can found mine in /lvm-bin

The LVM binary for recovery is lvm.static that can be rename as lvm

  

5. Verification:  
  Boot on recovery and copy the lvm binary
```
  /system/bin # lvm.static version                                                                                                                                                                                                      
```
```
  LVM version:     2.03.30(2) (2025-01-14)
  Library version: 1.02.204 (2025-01-14)
  Driver version:  4.37.0
  Configuration:   ./configure --prefix=/lvm --enable-static_link --disable-readline --disable-selinux --with-pool=none --with-cluster=none --with-confdir=/lvm/etc --with-default-run-dir=/data/lvm/run --with-default-system-dir=/lvm/etc --with-default-locking-dir=/data/lvm/lock --with-optimisation=-Os -march=armv8-a -mtune=generic-armv8-a
```
