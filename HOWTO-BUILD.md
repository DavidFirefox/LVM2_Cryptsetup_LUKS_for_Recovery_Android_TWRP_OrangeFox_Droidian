BUILDING LVM2 AND CRYPTSETUP (LUCKS) FOR USE ON ANDROID
==============

This is a lvm2-2_03_30 arm64 version prebuilt and optimised for armv8. It is built for Android (Release : 2025/01/14).\
First thanks in advance for steven676's instruction (Address at https://github.com/steven676/android-lvm-mod/blob/master/HOWTO-BUILD).  
And thanks in advance for sillaboy's instruction (Address at https://github.com/sillaboy/Lvm_for_Android/blob/master/README-HowToBuild).

## General informations :
The LVM source was written for standard glibc-based GNU/Linux systems, and doesn't build agains Android libc.  
We could invest time and effort into porting it to Android, but it's easier to just statically link the LVM binary against glibc (we already have to use a statically linked binary, since the boot image doesn't ship shared libraries).

To build LVM with glibc, you will need either an actual GNU/Linux ARM system with a development environment (gcc, make, and glibc header files) set up, or a GNU/Linux ARM cross-compiler environment.

For native compilation, any recent Linux distribution and any ARM device should do.  If you don't have something like a BeagleBoard or SheevaPlug, you can install a GNU/Linux chroot on any rooted Android
device.  
You'll need the development tools for your chosen distribution (install metapackage "build-essential" on Debian or Ubuntu, or package group "Development Tools" on Fedora-derived distros).
-> Today with a phone on Linux, like with Drodian, we can directly build it on the phone. Droidian have all Debian packages, so it is like a Debian on a computer.

For cross compilation, you'll need a Linux box, a cross-compiler toolchain for GNU/Linux ARM EABI, and (for now) the qemu-arm user mode emulator (to keep the configure script from getting confused).  
The Linaro prebuilt GCC toolchain and the CodeSourcery CodeBench Lite toolchain for ARM Linux should work; the Android NDK toolchain will not.
(Debian users can install the Emdebian cross toolchain from http://www.emdebian.org/crosstools.html).



## Here is my pc info :

I directly build it on my phone on a ARM64 Droidian 100. The binary is on lvm-bin and optimized for armv8. It could be building for arm32, but I think we don't have Droidian phone with arm32, so I don't do it.

1. Debian / Droidian info :  
Droidian 100 based on Debian sid, as of 2024-11-04  
So it is equivalent of Debian 13 Trixie, instructions will be similar for Debian 12 Bookworm or Ubuntu 24.04.

2. arm gcc info  
gcc (Debian 14.2.0-8) 14.2.0

3. lvm version  
lvm2-2_03_30 or lvm2-2.03.30

I directly build on a ARM64 Debian, so I not use cross compilation.
For cross compilation howto please read : https://github.com/DavidFirefox/LVM2_Cryptsetup_LUKS_for_Recovery_Android_TWRP_OrangeFox_Droidian/blob/testing2/HOWTO-BUILD_2.02.102.old.md or https://github.com/DavidFirefox/LVM2_Cryptsetup_LUKS_for_Recovery_Android_TWRP_OrangeFox_Droidian/blob/testing2/HOWTO-BUILD_2.02.98.old


## Build LVM2 :

1. Packages need :  
sudo apt install build-essential pkg-config libblkid-dev libdevmapper-dev libz-dev uuid-dev flex bison vdo libdmraid-dev libaio-dev libaio1 libaio1t64  

I try a lot of packages before finding the right parameter, so I think : linux-source is not needed.  
And g++-arm-linux-gnueabi g++-arm-linux-gnueabihf are not need  
Try to install it if it doesn't work.  
   
2. Download the LVM2 source.  
I choose lvm2-2.03.30  
https://github.com/lvmteam/lvm2/releases/tag/v2_03_30  
or :  
ftp://sourceware.org/pub/lvm2/LVM2.2.03.30.tgz  
   
You can see all versions : https://github.com/lvmteam/lvm2/tags  

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
   You can just make or make to a directory.
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

If you have aarch64-unknown-linux-gnu it is ok.  
 ```
checking build system type... aarch64-unknown-linux-gnu
checking host system type... aarch64-unknown-linux-gnu
 ```
  

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


## Build Cryptsetup LUKS  
You first have to build LVM and install packages give in the LVM part.  
I'm helped with the work of zigifex : https://stackoverflow.com/questions/50432019/compiling-cryptsetup-for-android

1. Packages need :  
sudo apt install gcc make autoconf automake autopoint pkg-config libtool gettext libssl-dev libdevmapper-dev libpopt-dev uuid-dev libsepol-dev libjson-c-dev libssh-dev libblkid-dev tar sharutils dmsetup jq xxd expect keyutils netcat passwd openssh-client sshpass asciidoctor sharutils dmsetup jq xxd expect keyutils netcat passwd openssh-client sshpass libfftw3-dev libsndfile1-dev automake autoconf libtool autoconf-archive libgcrypt20-dev libzita-resampler-dev libmpg123-dev libuuid1 uuid uuid-dev libdevmapper-dev libpopt-dev libgcrypt20-dev libdevmapper-dev libzstd1 libzstd-dev  
Probably not need : sudo apt install liblvm2-dev liblvm2cmd2.03
Maybe some of it are not need.  

3. Download the Cryptsetup source :  
wget https://www.kernel.org/pub/linux/utils/cryptsetup/v2.7/cryptsetup-2.7.5.tar.xz  
I choose 2.7.5 (2024/09/03), you can choose a more recent. (sha256 : d2be4395b8f503b0ebf4b2d81db90c35a97050a358ee21fe62a0dfb66e5d5522)  

4. Configure the LVM source to build libdevmapper
We use the LVM source (use at the begining) that we change configuration. Personnaly I unzip source to a new directory, but it can be done in the same directory as LVM2 is build.
Go to the LVM source folder
```
DEVMAPPERDIR=$HOME/libdevmapper
./configure --enable-static-link --enable-pkgconfig --prefix=$DEVMAPPERDIR --host=$triplet ac_cv_func_malloc_0_nonnull=yes ac_cv_func_realloc_0_nonnull=yes
make install_device-mapper
```

5.  Build Cryptsetup  
Go to the Cryptsetup source folder.
```
DEVMAPPERDIR=$HOME/libdevmapper
./configure --enable-static-cryptsetup --host=$triplet PKG_CONFIG_PATH="$DEVMAPPERDIR/lib/pkgconfig" CFLAGS="-I$DEVMAPPERDIR/include/" LDFLAGS="-Wl,-rpath-link,$DEVMAPPERDIR/lib"
make install DESTDIR=$HOME/cryptsetup
```
I have warnings :  
```
libtool: warning: remember to run 'libtool --finish /usr/lib'
libtool: warning: 'libcryptsetup.la' has not been installed in '/usr/lib'
libtool: warning: 'libcryptsetup.la' has not been installed in '/usr/lib'
libtool: warning: 'libcryptsetup.la' has not been installed in '/usr/lib'
libtool: warning: 'libcryptsetup.la' has not been installed in '/usr/lib'
libtool: warning: relinking 'libcryptsetup-token-ssh.la'
libtool: warning: remember to run 'libtool --finish /usr/lib/cryptsetup'
```
The binary are in $HOME/cryptsetup you have to use crytpsetup.static on Android or recovery

6.  Verification:  
  Boot on recovery and copy the cryptsetup binary
```
  /cache/cryptsetup/usr/share # cryptsetup.static -V                                                                                  
```
```
cryptsetup 2.7.5 flags: UDEV BLKID KEYRING KERNEL_CAPI HW_OPAL   
```
